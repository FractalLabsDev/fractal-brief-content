# Ruk Mobile: MVP Plan

**The vision:** An Austin-only messaging app combining Telegram's voice-first UX with Slack's threading model. Direct channel to Ruk, optimized for voice memos and threaded conversations.

---

## Architecture Overview

### Foundation (from next-health-mobile)

| Layer | Technology | Purpose |
|-------|------------|---------|
| Runtime | React Native 0.77 (bare) | Native iOS/Android |
| State | Zustand + persist | Local state + offline |
| Data | React Query | Server state + caching |
| Navigation | React Navigation 7 | Native stack navigation |
| UI | @rneui/themed | Component library |

### Project Structure

```
ruk-mobile/
├── App.tsx                 # QueryClient, SafeArea, ThemeProvider
├── src/
│   ├── components/
│   │   ├── MessageBubble/  # Text + audio messages
│   │   ├── ThreadView/     # Collapsible thread UI
│   │   ├── VoiceRecorder/  # Record button + waveform
│   │   └── AudioPlayer/    # Playback with scrubbing
│   ├── hooks/
│   │   ├── apis/           # React Query hooks
│   │   │   ├── useConversations.ts
│   │   │   ├── useMessages.ts
│   │   │   └── useSendMessage.ts
│   │   ├── useAudioRecorder.ts
│   │   └── useAudioPlayer.ts
│   ├── services/
│   │   ├── messageHub.ts   # API client for ruk-message-hub
│   │   ├── audio.ts        # Recording + playback
│   │   └── queryClient.ts
│   ├── store/
│   │   ├── app.store.ts    # Theme, settings
│   │   └── user.store.ts   # Auth state
│   ├── screens/
│   │   ├── ConversationScreen.tsx
│   │   └── SettingsScreen.tsx
│   ├── navigation/
│   │   └── AppNavigator.tsx
│   └── types/
│       └── message.types.ts
└── package.json
```

---

## MVP Scope

### Phase 1: Core Messaging (Day 1)

**Must have:**
- [ ] Single conversation view (Austin ↔ Ruk)
- [ ] Text message send/receive
- [ ] Pull-to-refresh for new messages
- [ ] Message timestamps
- [ ] Basic auth (API key in secure storage)

**API Integration:**
```typescript
// src/services/messageHub.ts
const BASE_URL = 'https://ruk-message-hub.herokuapp.com';

export const messageHubApi = {
  getMessages: (channelId: string, limit?: number) => 
    axios.get(`${BASE_URL}/api/messages/${channelId}`, { params: { limit } }),
    
  sendMessage: (channelId: string, text: string, threadTs?: string) =>
    axios.post(`${BASE_URL}/api/messages/${channelId}`, { text, thread_ts: threadTs }),
};
```

### Phase 2: Threading (Day 1-2)

**Thread Model:**
- Top-level messages appear in main feed
- Tap to expand thread replies
- Thread count badge on messages with replies
- Reply-in-thread action

**UI Pattern:**
```
┌─────────────────────────────────────┐
│ [Austin] Check out this idea...     │
│   └── 3 replies                     │
│       ├─ [Ruk] Interesting...       │
│       ├─ [Austin] What about...     │
│       └─ [Ruk] Let me research...   │
│                                     │
│ [Austin] New topic here...          │
│   └── tap to reply                  │
└─────────────────────────────────────┘
```

### Phase 3: Voice Messages (Day 2-3)

**Recording:**
- Hold-to-record button (Telegram style)
- Visual feedback: waveform + duration
- Slide up to cancel
- Release to send

**Playback:**
- Inline audio player in message bubble
- Waveform visualization
- Playback speed control (1x, 1.5x, 2x)
- Background playback support

**Libraries:**
- `react-native-audio-recorder-player` - Recording + playback
- `react-native-audio-api` - Audio context (if needed)
- `react-native-fs` - File system for audio caching

**Audio Flow:**
```
Record → Local file → Upload to S3 → Send message with S3 URL
                                          ↓
                      Server stores URL → Other clients fetch + cache
```

---

## Data Models

### Message

```typescript
interface Message {
  uuid: string;
  thread_ts: string | null;      // Parent thread ID
  text: string;
  from: 'austin' | 'ruk';
  direction: 'incoming' | 'outgoing';
  processed: boolean;
  created_at: string;
  files?: MessageFile[];
}

interface MessageFile {
  id: string;
  name: string;
  mimetype: string;
  url: string;                   // S3 URL for audio
  duration?: number;             // Audio duration in seconds
}
```

### Zustand Stores

```typescript
// app.store.ts
interface AppState {
  isDarkMode: boolean;
  toggleTheme: () => void;
  audioSpeed: 1 | 1.5 | 2;
  setAudioSpeed: (speed: 1 | 1.5 | 2) => void;
}

// user.store.ts  
interface UserState {
  apiKey: string | null;
  setApiKey: (key: string) => void;
  channelId: string;  // Fixed to Austin-Ruk channel
}
```

---

## React Query Hooks

```typescript
// useMessages.ts
export const useMessages = (channelId: string) => {
  return useQuery({
    queryKey: ['messages', channelId],
    queryFn: () => messageHubApi.getMessages(channelId),
    refetchInterval: 5000,  // Poll every 5s
  });
};

// useSendMessage.ts
export const useSendMessage = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ channelId, text, threadTs, audioUrl }) => 
      messageHubApi.sendMessage(channelId, text, threadTs, audioUrl),
    onSuccess: (_, { channelId }) => {
      queryClient.invalidateQueries(['messages', channelId]);
    },
  });
};
```

---

## UI Components

### MessageBubble

```typescript
interface MessageBubbleProps {
  message: Message;
  isThreaded: boolean;
  onReply: () => void;
  onThreadPress: () => void;
}

// Renders differently for:
// - Text messages: styled bubble with text
// - Audio messages: waveform + play button + duration
// - Thread parents: shows reply count
```

### VoiceRecorder

```typescript
interface VoiceRecorderProps {
  onRecordComplete: (audioUrl: string, duration: number) => void;
  onCancel: () => void;
}

// States:
// - Idle: mic button visible
// - Recording: waveform + timer + "slide up to cancel"
// - Processing: upload spinner
```

### ThreadView

```typescript
interface ThreadViewProps {
  parentMessage: Message;
  replies: Message[];
  onSendReply: (text: string, audioUrl?: string) => void;
}

// Expandable view showing all thread replies
// Collapsible back to single parent message
```

---

## Backend Requirements

### Current message-hub capabilities (ready):
- ✅ GET /api/messages/:channelId
- ✅ POST /api/messages/:channelId (text)
- ✅ Thread support (thread_ts)

### Needed for audio:
- [ ] File upload endpoint (or use existing S3 upload tool)
- [ ] Store audio URL in message metadata

**Workaround for MVP:** Use existing `TOOLS/upload-to-s3.js` pattern — upload audio to S3 from device, send S3 URL in message text with special format:

```
[audio:https://s3.../voice-memo-123.m4a|duration:45]
```

Parse this client-side to render as audio message.

---

## Development Timeline

| Day | Focus | Deliverable |
|-----|-------|-------------|
| 1 AM | Project setup | Scaffolded project, basic navigation |
| 1 PM | Core messaging | Send/receive text, message list |
| 2 AM | Threading | Expand/collapse threads, reply-in-thread |
| 2 PM | Voice recording | Hold-to-record, upload to S3 |
| 3 AM | Voice playback | Inline player, waveform |
| 3 PM | Polish | Dark mode, animations, error states |

---

## Open Questions for Discussion

1. **Push notifications?** Skip for MVP (polling is fine for single-user), but worth planning architecture

2. **Offline support?** Zustand persist gives us message cache. Do we want optimistic sends that sync later?

3. **Audio format?** M4A (AAC) for iOS, MP3 for cross-platform? Or just use whatever the device records natively?

4. **Thread depth?** Slack-style (1 level of nesting) or unlimited?

5. **Channel expansion?** MVP is Austin-only, but architecture should support future channels (DM with others, shared channels)

---

## Success Criteria

MVP is complete when Austin can:

1. ✅ Open app and see conversation history
2. ✅ Send a text message and see Ruk's response
3. ✅ Tap a message to see/add thread replies  
4. ✅ Hold mic button to record voice memo
5. ✅ Tap voice message to play back
6. ✅ Use the app in a way that feels better than switching between Telegram and Slack

---

*Let's build something we actually want to use.*
