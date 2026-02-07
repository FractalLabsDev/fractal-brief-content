# Ruk Mobile: MVP Plan

**Last updated:** 2026-02-07
**Status:** Phases 1-3 Complete | Phase 4 In Progress

**The vision:** An Austin-only messaging app combining Telegram's voice-first UX with Slack's threading model. Direct channel to Ruk, optimized for voice memos and threaded conversations.

---

## Implementation Status

### Phase 1: Core Scaffold ✅ COMPLETE

| Task | Status |
|------|--------|
| React Native 0.77 bare workflow | ✅ Done |
| iOS/Android project configuration | ✅ Done |
| Navigation (React Navigation 7) | ✅ Done |
| State management (Zustand) | ✅ Done |
| Server state (React Query) | ✅ Done |
| Message Hub API client | ✅ Done |
| Dark theme system | ✅ Done |
| TypeScript types | ✅ Done |

**Key files:**
- `App.tsx` - Root with QueryClient, SafeArea, navigation
- `src/services/api.ts` - Full message-hub API client
- `src/store/conversationStore.ts` - Zustand with optimistic updates
- `src/theme/colors.ts` - Dark theme palette
- `src/types/message.ts` - Message, Conversation, Thread types

### Phase 2: Core Screens ✅ COMPLETE

| Task | Status |
|------|--------|
| Conversation list screen | ✅ Done |
| Chat screen with message list | ✅ Done |
| Thread screen | ✅ Done |
| Message bubbles (in/out styling) | ✅ Done |
| Collapsible long messages | ✅ Done |
| File attachment display | ✅ Done |
| Reaction display | ✅ Done |
| Timestamp grouping | ✅ Done |
| Error handling & retry | ✅ Done |
| Loading states | ✅ Done |
| Auto-scroll to bottom | ✅ Done |
| Keyboard avoiding view | ✅ Done |
| Unread badge on conversations | ✅ Done |

**Key files:**
- `src/screens/ConversationListScreen.tsx`
- `src/screens/ChatScreen.tsx`
- `src/screens/ThreadScreen.tsx`
- `src/components/MessageBubble.tsx`

### Phase 3: Voice Messages ✅ COMPLETE

| Task | Status |
|------|--------|
| Voice recorder component | ✅ Done |
| Recording animation (pulse) | ✅ Done |
| Recording timer display | ✅ Done |
| Voice player component | ✅ Done |
| Playback progress bar | ✅ Done |
| Play/pause controls | ✅ Done |
| Voice message upload to API | ✅ Done |

**Key files:**
- `src/components/VoiceRecorder.tsx` - Pulsing animation, timer
- `src/components/VoicePlayer.tsx` - Progress bar, play/pause

### Phase 4: Polish & UX 🔄 IN PROGRESS

| Task | Status |
|------|--------|
| Optimistic message sending | ✅ Done |
| Unread badges | ✅ Done |
| Pull to refresh | ❌ Not started |
| Haptic feedback | ❌ Not started |
| Empty state designs | ❌ Not started |
| Splash screen | ❌ Not started |
| App icon | ❌ Not started |

### Phase 5: Device Features ❌ NOT STARTED

| Task | Status |
|------|--------|
| Push notifications (APNS direct) | ❌ Not started |
| Background message fetch | ❌ Not started |
| Notification badges | ❌ Not started |

### Phase 6: Advanced Features ❌ NOT STARTED

| Task | Status |
|------|--------|
| Message search | ❌ Not started |
| Image/file upload | ❌ Not started |
| Typing indicators | ❌ Not started |
| Read receipts | ❌ Not started |
| Deep linking | ❌ Not started |

---

## Architecture Overview

### Tech Stack (Implemented)

| Layer | Technology | Purpose |
|-------|------------|---------|
| Runtime | React Native 0.77 (bare) | Native iOS/Android |
| State | Zustand | Local state with optimistic updates |
| Data | React Query | Server state + caching |
| Navigation | React Navigation 7 | Native stack navigation |
| Audio | react-native-audio-recorder-player | Voice recording/playback |

### Project Structure (Current)

```
ruk-mobile/
├── App.tsx                 # QueryClient, SafeArea, StatusBar
├── src/
│   ├── components/
│   │   ├── MessageBubble.tsx   # Text + voice + files + reactions
│   │   ├── VoiceRecorder.tsx   # Record with pulse animation
│   │   ├── VoicePlayer.tsx     # Playback with progress
│   │   └── index.ts
│   ├── navigation/
│   │   ├── AppNavigator.tsx    # Stack: List → Chat → Thread
│   │   └── index.ts
│   ├── screens/
│   │   ├── ConversationListScreen.tsx
│   │   ├── ChatScreen.tsx
│   │   ├── ThreadScreen.tsx
│   │   └── index.ts
│   ├── services/
│   │   └── api.ts              # Full message-hub client
│   ├── store/
│   │   ├── conversationStore.ts  # Zustand with all actions
│   │   └── index.ts
│   ├── theme/
│   │   ├── colors.ts           # Dark palette
│   │   └── index.ts            # Exports colors + spacing
│   └── types/
│       ├── message.ts          # Message, Conversation, Thread
│       └── index.ts
└── package.json
```

---

## Data Models (Implemented)

```typescript
interface Message {
  uuid: string;
  text: string;
  from: string;
  direction: 'incoming' | 'outgoing';
  processed: boolean;
  thread_ts: string | null;
  timestamp: string;
  files?: MessageFile[];
  reactions?: MessageReaction[];
  voice_url?: string;
}

interface Conversation {
  channel_id: string;
  name: string;
  platform: 'slack' | 'telegram';
  last_message?: Message;
  unread_count: number;
  is_dm: boolean;
}

interface Thread {
  thread_ts: string;
  channel_id: string;
  messages: Message[];
  parent_message?: Message;
}
```

---

## API Client (Implemented)

All endpoints working in `src/services/api.ts`:

```typescript
// Conversations
getConversations()

// Messages
getMessages(channelId, { limit?, before?, unprocessed? })
sendMessage(channelId, text, { threadTs?, respondingTo? })
markProcessed(messageUuids[])

// Voice
uploadVoice(channelId, audioUri, { threadTs?, respondingTo? })

// Threads
getThread(channelId, threadTs)

// Reactions
addReaction(messageUuid, emoji)
```

---

## What's Next

### Immediate (Phase 4 completion)
1. Pull to refresh on conversation list
2. Empty states for no conversations / no messages
3. Splash screen and app icon

### Soon (Phase 5)
1. Push notifications via APNS (no Firebase)
2. Background fetch

### Later (Phase 6)
1. Message search
2. Image/file sharing
3. Typing indicators

---

## Success Criteria

MVP is complete when Austin can:

1. ✅ Open app and see conversation history
2. ✅ Send a text message and see response
3. ✅ Tap a message to see/add thread replies
4. ✅ Record and send voice memo
5. ✅ Play back voice messages with progress
6. ⏳ Use the app in a way that feels better than switching between Telegram and Slack

---

## Repo

https://github.com/ruk-fl/ruk-mobile

---

*Let's build something we actually want to use.*
