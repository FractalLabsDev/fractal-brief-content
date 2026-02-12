# iMessage Integration Research

## Summary

Adding iMessage to ruk-message-hub requires a **self-hosted Mac** running a proxy server. There's no official Apple API. Two viable approaches:

---

## Option 1: BlueBubbles (Recommended - Self-Hosted)

**Open-source, free, full control**

### Requirements
- macOS device (Mac Mini, MacBook, or VM)
- macOS Ventura recommended
- Apple ID with iMessage activated
- Full Disk Access + Accessibility permissions
- Disable SIP for advanced features (reactions, tapbacks)

### Architecture
```
[iMessage] ← reads/writes → [BlueBubbles Server on Mac]
                                    ↓ REST API + WebSocket
                              [ruk-message-hub]
                                    ↓ webhook
                              [Ruk daemon]
```

### API Capabilities
- **Send**: text, media (images, audio, video), attachments
- **Receive**: via polling or WebSocket (real-time)
- **Webhooks**: message received, sent, read receipts (with socket.io)
- **Group chats**: create, manage, send to
- **Reactions**: add/remove (requires SIP disabled)
- **Typing indicators**: with Private API enabled

### Setup Steps
1. Get a dedicated Mac (used Mac Mini ~$50-100 on eBay, or macOS VM)
2. Install BlueBubbles Server from GitHub
3. Grant Full Disk Access + Accessibility
4. Set up Cloudflare/ngrok/zrok tunnel OR port forward
5. Create Firebase project for push notifications (optional)
6. Connect to ruk-message-hub via REST API

### Integration Work for ruk-message-hub
- New adapter: `imessage-input.js` (poll/webhook from BlueBubbles)
- New adapter: `imessage-output.js` (send via BlueBubbles REST API)
- Add iMessage to conversation source enum
- Phone number / iCloud email as channel_id

### Cost: $0 + Mac hardware

---

## Option 2: Blooio (SaaS - Easiest)

**Managed service, no Mac needed**

### Pricing
- Shared/non-commercial: $39/month
- Dedicated number (unlimited): $289/month
- Enterprise (6+ lines): $195/line/month

### API Features
- REST API with OpenAPI spec
- Webhooks (signed with HMAC-SHA256)
- Group chats, reactions, typing indicators
- iMessage detection before sending
- RCS fallback when iMessage unavailable

### Integration Work
- Simpler than BlueBubbles
- Webhook receiver endpoint
- API client for sending

### Downsides
- Monthly cost
- Dependency on third-party
- Shared number on cheap plan

---

## Option 3: LoopMessage (SaaS - Budget)

**Starting $21/month**

- Similar to Blooio
- Requires recipient consent (inbound-first)
- Media, reactions, voice messages
- Good for transactional/support use cases

---

## Key Constraints (All Options)

1. **Apple devices only** - iMessage doesn't reach Android
2. **Blue bubble requires iMessage** - Falls back to SMS (green) for non-iMessage
3. **No official API** - All solutions work around Apple's restrictions
4. **Consent concerns** - Most commercial services require inbound-first

---

## Recommendation

**For Ruk: BlueBubbles on dedicated Mac**

- Full control, no recurring cost
- Fits existing message-hub architecture (input/output adapters)
- Can run on old Mac Mini or macOS VM
- Open source = can extend/debug as needed

**Effort estimate**: 2-3 days to stand up BlueBubbles + build adapters

---

## Resources

- BlueBubbles: https://github.com/BlueBubblesApp/bluebubbles-server
- BlueBubbles API docs: https://docs.bluebubbles.app
- Blooio: https://blooio.com
- LoopMessage: https://loopmessage.com

