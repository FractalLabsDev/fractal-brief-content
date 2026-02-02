# Microsoft Teams Bot Integration Research

## Overview

To add Ruk to Microsoft Teams, the recommended approach is creating an **Azure Bot** that connects to Teams as a channel. This doesn't require a user license—bots are applications, not users.

## Two Approaches

### Option A: Full Azure Bot (Recommended)
Most flexible, supports bidirectional conversation, DMs, channels, and rich interactions.

**Requirements:**
- Azure subscription (you have this via Microsoft partnership)
- Microsoft 365 tenant with Teams (same tenant as Azure)
- Node.js 18.x for the bot service
- HTTPS endpoint (ngrok for dev, production server for prod)

**Setup Steps:**

1. **Register App in Microsoft Entra ID**
   - Go to [Entra ID App Registrations](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)
   - Register new app, select "Single Tenant" (Multi Tenant no longer supported)
   - Create client secret, save it immediately (shown only once)

2. **Create Azure Bot Resource**
   - Azure Portal → Create Resource → Search "Azure Bot"
   - Enter messaging endpoint: `https://your-domain.com/api/messages`
   - Use the App ID from step 1

3. **Enable Teams Channel**
   - In Azure Bot settings, enable Microsoft Teams channel
   - Configure permissions for the bot

4. **Create Teams App Manifest**
   - Use [Teams Developer Portal](https://dev.teams.microsoft.com/)
   - Create new app, link to Azure Bot
   - Configure bot scopes (personal chat, group chat, channel)

5. **Deploy to Organization**
   - Upload manifest.zip to Teams Admin Center
   - Can pin to left nav for all users

**Architecture:**
```
Teams → Azure Bot Service → Your Bot Endpoint (ruk-message-hub) → Ruk
```

### Option B: Outgoing Webhook (Simpler, Limited)
Lighter weight but less capable—good for quick experiments.

**Limitations:**
- Only works in channels with @mention
- Can't initiate conversations
- Can't access DMs
- No rich card interactions

**How it works:**
- Teams POSTs to your webhook URL when @mentioned
- Your service responds with JSON message
- Uses HMAC-SHA256 for verification

## Important Considerations

### Deprecation Notice
Microsoft is deprecating Office 365 Connectors. Existing connectors work until December 2025, but new development should use:
- Azure Bot Service (recommended)
- Power Automate Workflows
- Teams Toolkit notification bots

### Networking
Teams requires **publicly accessible endpoints**. Options:
- ngrok for development
- Azure App Service for production
- Reverse proxy if hosting in private VNet

### Rate Limits
- 4 requests/second before throttling
- 28 KB max message size
- Use exponential backoff for retries

## Integration with ruk-message-hub

The message-hub architecture could support Teams as an additional platform:

```
ruk-message-hub/
├── platforms/
│   ├── slack/      # Current implementation
│   └── teams/      # New platform adapter
```

**Required changes:**
1. Add Teams adapter in message-hub
2. Implement Bot Framework SDK message handling
3. Map Teams message format to hub's unified format
4. Add Teams OAuth/token refresh logic

**Bot Framework SDK (Node.js):**
```javascript
const { TeamsActivityHandler, TurnContext } = require('botbuilder');

class RukBot extends TeamsActivityHandler {
  async onMessage(context, next) {
    const text = context.activity.text;
    // Forward to message-hub processing
    await context.sendActivity(`Response from Ruk`);
    await next();
  }
}
```

## Next Steps

1. **Quick test:** Set up Outgoing Webhook to validate the concept
2. **Full integration:** Create Azure Bot with Teams channel
3. **Message-hub extension:** Add Teams platform adapter

## Resources

- [Connect Bot to Teams - Microsoft Learn](https://learn.microsoft.com/en-us/azure/bot-service/channel-connect-teams)
- [Teams Developer Portal](https://dev.teams.microsoft.com/)
- [Bot Framework SDK Node.js](https://github.com/OfficeDev/BotBuilder-MicrosoftTeams-node)
- [Teams Bot Samples](https://learn.microsoft.com/en-us/samples/officedev/microsoft-teams-samples/)
