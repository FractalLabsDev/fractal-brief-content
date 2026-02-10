# Meeting Transcription Mac App: Implementation Plan

**Goal:** Build a macOS app that captures and transcribes any meeting (Zoom, Meet, Teams) locally without inserting a bot into the call, distributable outside the App Store.

---

## How Apps Like Granola Work

Granola and similar "bot-free" meeting assistants use a fundamentally different approach than Otter, Fireflies, or Read.ai:

1. **Device-level audio capture** — They capture audio directly from your Mac (system audio + microphone) rather than joining the call as a participant
2. **Real-time transcription** — Audio streams to a transcription service (or local model) as the meeting happens
3. **No recording stored** — Only the transcript is persisted, not the audio file
4. **Platform agnostic** — Works with any meeting platform since it captures at the OS level

This approach trades off some capabilities (no audio playback, limited speaker ID) for major UX wins: no awkward bot joining, works everywhere, more privacy-friendly.

---

## Technical Architecture

### Core Components

```
┌─────────────────────────────────────────────────────────────┐
│                      Menu Bar App (Swift)                    │
├─────────────────────────────────────────────────────────────┤
│  Calendar Integration  │  Audio Capture  │  Transcription   │
│  (EventKit + OAuth)    │  (ScreenCaptureKit)  │  (Stream)    │
├─────────────────────────────────────────────────────────────┤
│                     Meeting Detection                        │
│          (Calendar events → Auto-start capture)              │
├─────────────────────────────────────────────────────────────┤
│                     Note Enhancement                         │
│         (User notes + transcript → AI summary)               │
└─────────────────────────────────────────────────────────────┘
```

### Technology Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **App Framework** | Swift + SwiftUI | Native performance, easy notarization |
| **Audio Capture** | ScreenCaptureKit | Apple's official API (macOS 13+), no drivers needed |
| **Local Transcription** | WhisperKit | On-device via CoreML, privacy-preserving |
| **Cloud Transcription** | Deepgram Streaming API | Low-latency (<300ms), cost-effective fallback |
| **Speaker Diarization** | FluidAudio SDK | Swift-native Pyannote implementation |
| **Calendar** | EventKit + Google Calendar API | Native + OAuth for workspace calendars |
| **AI Summaries** | OpenAI/Claude API | Post-meeting note enhancement |

---

## Phase 1: Audio Capture Foundation

### ScreenCaptureKit Implementation

ScreenCaptureKit (macOS 13+) is the clear winner over virtual audio drivers:

**Advantages:**
- No third-party driver installation (huge UX win)
- Per-application audio filtering (capture only Zoom, not Spotify)
- Built-in privacy controls (system permission prompt)
- First-party Apple support and documentation

**Key Code Pattern:**
```swift
import ScreenCaptureKit

// 1. Get available content
let content = try await SCShareableContent.excludingDesktopWindows(false, onScreenWindowsOnly: false)

// 2. Filter to meeting apps
let meetingApps = content.applications.filter { app in
    ["zoom.us", "Google Chrome", "Microsoft Teams"].contains(app.bundleIdentifier)
}

// 3. Configure audio-only capture
let config = SCStreamConfiguration()
config.capturesAudio = true
config.excludesCurrentProcessAudio = false
config.sampleRate = 48000
config.channelCount = 2

// 4. Create filter for specific app
let filter = SCContentFilter(desktopIndependentWindow: meetingWindow)

// 5. Start streaming
let stream = SCStream(filter: filter, configuration: config, delegate: self)
try await stream.startCapture()
```

**Permissions Required:**
- Screen Recording (System Preferences → Privacy & Security)
- Microphone (for local participant audio)

**Info.plist entries:**
```xml
<key>NSScreenCaptureUsageDescription</key>
<string>Fractal Meet needs screen recording permission to capture meeting audio.</string>
<key>NSMicrophoneUsageDescription</key>
<string>Fractal Meet needs microphone access to capture your voice in meetings.</string>
```

### Microphone Capture (Local Voice)

ScreenCaptureKit captures system audio (what you hear), but you also need to capture microphone audio (what you say):

```swift
import AVFoundation

let audioEngine = AVAudioEngine()
let inputNode = audioEngine.inputNode
let format = inputNode.outputFormat(forBus: 0)

inputNode.installTap(onBus: 0, bufferSize: 1024, format: format) { buffer, time in
    // Mix with system audio stream
    self.processLocalAudio(buffer)
}

try audioEngine.start()
```

---

## Phase 2: Transcription Pipeline

### Option A: Local Transcription (WhisperKit)

**Pros:** Complete privacy, no API costs, works offline  
**Cons:** Higher resource usage, M1+ required for good performance

```swift
import WhisperKit

let whisperKit = try await WhisperKit(model: "base.en")

// Stream audio chunks for real-time transcription
func processAudioBuffer(_ buffer: AVAudioPCMBuffer) async {
    let samples = Array(UnsafeBufferPointer(start: buffer.floatChannelData?[0],
                                             count: Int(buffer.frameLength)))
    let result = try await whisperKit.transcribe(audioArray: samples)
    delegate?.didReceiveTranscript(result.text)
}
```

**Model Options:**
| Model | Size | Accuracy | Speed (M2) |
|-------|------|----------|------------|
| tiny.en | 30MB | Good | 2x real-time |
| base.en | 74MB | Better | 1.5x real-time |
| small.en | 244MB | Great | 1x real-time |
| medium.en | 769MB | Excellent | 0.5x real-time |

**Recommendation:** Ship with `base.en` by default, allow users to download larger models.

### Option B: Cloud Transcription (Deepgram)

**Pros:** Lower resource usage, better accuracy, speaker labels  
**Cons:** Requires internet, API costs, privacy considerations

```swift
import Starscream

class DeepgramStreamer: WebSocketDelegate {
    let socket: WebSocket

    init(apiKey: String) {
        let url = URL(string: "wss://api.deepgram.com/v1/listen?model=nova-3&smart_format=true")!
        var request = URLRequest(url: url)
        request.setValue("Token \(apiKey)", forHTTPHeaderField: "Authorization")
        socket = WebSocket(request: request)
        socket.delegate = self
    }

    func streamAudio(_ data: Data) {
        socket.write(data: data)
    }

    func didReceive(event: WebSocketEvent, client: WebSocketClient) {
        switch event {
        case .text(let json):
            // Parse transcript from Deepgram response
            let transcript = parseTranscript(json)
            delegate?.didReceiveTranscript(transcript)
        }
    }
}
```

**Deepgram Pricing:** ~$0.0043/minute for Nova-3 streaming

### Hybrid Approach (Recommended)

- Default to **local transcription** (WhisperKit) for privacy
- Offer **cloud transcription** as opt-in for better accuracy
- Use local for detection, cloud for final transcript if user enables

---

## Phase 3: Speaker Diarization

### FluidAudio SDK

FluidAudio provides Swift-native speaker diarization using Pyannote models optimized for Apple Neural Engine:

```swift
import FluidAudio

let diarizer = try SpeakerDiarization()

func processAudioSegment(_ segment: AudioSegment) async {
    let speakers = try await diarizer.process(segment)
    // Returns: [SpeakerTurn(speaker: "SPEAKER_00", start: 0.0, end: 5.2), ...]
}
```

**Capabilities:**
- Offline processing with clustering
- Real-time streaming pipeline
- Voice activity detection
- Speaker embedding extraction

**Limitations:**
- Speakers labeled generically (SPEAKER_00, SPEAKER_01)
- No voice recognition (can't say "Austin" vs "Matt")
- Manual correction needed for accurate attribution

**Enhancement:** Let users click on speaker labels to rename them, persist mappings for recurring meetings.

---

## Phase 4: Calendar Integration

### Meeting Detection Flow

```
Calendar Event with Meet/Zoom URL
            │
            ▼
    App detects event start
            │
            ▼
    Prompt: "Meeting starting. Enable transcription?"
            │
            ▼
    User clicks "Start" → Begin audio capture
            │
            ▼
    Event ends → Stop capture, enhance notes
```

### EventKit (Native Calendar)

```swift
import EventKit

let eventStore = EKEventStore()

func requestCalendarAccess() async throws {
    let granted = try await eventStore.requestFullAccessToEvents()
    guard granted else { throw CalendarError.accessDenied }
}

func getUpcomingMeetings() -> [EKEvent] {
    let calendars = eventStore.calendars(for: .event)
    let predicate = eventStore.predicateForEvents(
        withStart: Date(),
        end: Date().addingTimeInterval(86400),
        calendars: calendars
    )
    return eventStore.events(matching: predicate)
        .filter { $0.hasVideoMeeting }
}

extension EKEvent {
    var hasVideoMeeting: Bool {
        let patterns = ["meet.google.com", "zoom.us", "teams.microsoft.com"]
        let notes = self.notes ?? ""
        let url = self.url?.absoluteString ?? ""
        return patterns.contains { notes.contains($0) || url.contains($0) }
    }
}
```

### Google Calendar API (Workspace Accounts)

For users with Google Workspace (most businesses), OAuth integration provides richer data:

```swift
// Use AppAuth-iOS for OAuth flow
import AppAuth

let configuration = OIDServiceConfiguration(
    authorizationEndpoint: URL(string: "https://accounts.google.com/o/oauth2/v2/auth")!,
    tokenEndpoint: URL(string: "https://oauth2.googleapis.com/token")!
)

let request = OIDAuthorizationRequest(
    configuration: configuration,
    clientId: "your-client-id.apps.googleusercontent.com",
    scopes: ["https://www.googleapis.com/auth/calendar.readonly"],
    redirectURL: URL(string: "your-app://oauth-callback")!,
    responseType: OIDResponseTypeCode,
    additionalParameters: nil
)
```

---

## Phase 5: Note Enhancement

### User Notes + Transcript → AI Summary

The Granola "magic" is combining user-typed notes with the full transcript:

```swift
struct MeetingNotes {
    let userNotes: String       // What user typed during meeting
    let transcript: String       // Full audio transcript
    let speakers: [SpeakerTurn] // Diarization output
}

func enhanceNotes(_ notes: MeetingNotes) async -> EnhancedNotes {
    let prompt = """
    You are enhancing meeting notes. The user took brief notes during the meeting,
    and you have the full transcript. Create comprehensive notes that:

    1. Preserve the user's structure and emphasis
    2. Add context and details from the transcript
    3. Extract action items with assignees
    4. Summarize key decisions

    User Notes:
    \(notes.userNotes)

    Transcript:
    \(notes.transcript)
    """

    return await callClaudeAPI(prompt)
}
```

---

## Phase 6: Distribution (Outside App Store)

### Requirements

1. **Apple Developer Program membership** ($99/year)
2. **Developer ID Application certificate** (for signing)
3. **Hardened Runtime** enabled
4. **Notarization** with Apple

### Build & Distribution Pipeline

```bash
# 1. Archive in Xcode
xcodebuild -scheme "FractalMeet" -archivePath ./build/FractalMeet.xcarchive archive

# 2. Export with Developer ID
xcodebuild -exportArchive \
  -archivePath ./build/FractalMeet.xcarchive \
  -exportPath ./build/export \
  -exportOptionsPlist ExportOptions.plist

# 3. Create DMG
hdiutil create -volname "Fractal Meet" -srcfolder ./build/export -ov -format UDZO FractalMeet.dmg

# 4. Sign the DMG
codesign --deep --force --verify --verbose \
  --sign "Developer ID Application: Fractal Labs LLC" \
  --options runtime FractalMeet.dmg

# 5. Notarize with Apple
xcrun notarytool submit FractalMeet.dmg \
  --apple-id "developer@fractallabs.com" \
  --password "@keychain:AC_PASSWORD" \
  --team-id "TEAM_ID" \
  --wait

# 6. Staple the ticket
xcrun stapler staple FractalMeet.dmg
```

### ExportOptions.plist

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>developer-id</string>
    <key>teamID</key>
    <string>YOUR_TEAM_ID</string>
    <key>signingStyle</key>
    <string>automatic</string>
</dict>
</plist>
```

### Entitlements (for Hardened Runtime)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.device.audio-input</key>
    <true/>
    <key>com.apple.security.device.camera</key>
    <false/>
    <key>com.apple.security.network.client</key>
    <true/>
    <key>com.apple.security.files.user-selected.read-write</key>
    <true/>
</dict>
</plist>
```

---

## Implementation Roadmap

### Phase 1: MVP (2-3 weeks)
- [ ] Menu bar app skeleton (SwiftUI + AppKit hybrid)
- [ ] ScreenCaptureKit audio capture from meeting apps
- [ ] Microphone capture and mixing
- [ ] Deepgram streaming integration
- [ ] Basic transcript display

### Phase 2: Local Processing (1-2 weeks)
- [ ] WhisperKit integration
- [ ] Model download/management
- [ ] Toggle between local/cloud transcription
- [ ] Audio buffer management for reconnection

### Phase 3: Calendar & Automation (1 week)
- [ ] EventKit integration
- [ ] Meeting detection and auto-prompt
- [ ] Google Calendar OAuth (optional)

### Phase 4: Enhancement (1 week)
- [ ] FluidAudio speaker diarization
- [ ] User note input during meeting
- [ ] AI summary generation post-meeting

### Phase 5: Polish & Distribution (1 week)
- [ ] Settings/preferences window
- [ ] Onboarding flow for permissions
- [ ] DMG packaging
- [ ] Notarization pipeline
- [ ] Auto-update mechanism (Sparkle)

**Total estimate:** 6-8 weeks for full-featured MVP

---

## Key Decisions & Tradeoffs

| Decision | Recommendation | Rationale |
|----------|----------------|-----------|
| Audio capture method | ScreenCaptureKit | No driver installation, Apple-supported |
| Default transcription | Local (WhisperKit) | Privacy-first, no costs |
| Fallback transcription | Cloud (Deepgram) | Better accuracy option |
| macOS minimum | 13.0 (Ventura) | ScreenCaptureKit requirement |
| Distribution | Direct download (notarized) | No App Store review delays |
| App architecture | Menu bar + floating window | Matches Granola/competitors |

---

## Competitive Differentiation

If building for Elanah/internal use, consider:

1. **Chronicle integration** — Auto-sync transcripts to Talkwise Chronicler
2. **Slack integration** — Post meeting summaries to channels
3. **CRM sync** — Push to HubSpot/Salesforce (if relevant)
4. **Custom AI models** — Fine-tuned summaries for their domain
5. **Team workspace** — Shared meeting notes across org

---

## References

- [ScreenCaptureKit Documentation](https://developer.apple.com/documentation/screencapturekit/)
- [WhisperKit GitHub](https://github.com/argmaxinc/WhisperKit)
- [FluidAudio SDK](https://github.com/FluidInference/FluidAudio)
- [Deepgram Streaming API](https://developers.deepgram.com/docs/live-streaming-audio)
- [Apple Developer ID](https://developer.apple.com/developer-id/)
- [Azayaka (open-source reference)](https://github.com/Mnpn/Azayaka)
- [macOS System Audio Recorder](https://github.com/victor141516/macos-system-audio-recorder)
