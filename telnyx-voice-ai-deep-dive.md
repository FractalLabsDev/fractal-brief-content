# Telnyx Deep Dive: Voice AI Agent Platform

## Executive Summary

Telnyx is a **full-stack telecommunications and AI infrastructure provider** that owns its global IP backbone and holds telecom licenses in 30+ countries. Unlike Twilio (which resells third-party infrastructure), Telnyx provides end-to-end control over voice, messaging, and AI inference. For building a voice-enabled AI agent, Telnyx offers a compelling platform with **custom LLM support (including Claude via OpenAI-compatible endpoints)**, **real-time tool execution during calls**, and **significantly lower pricing than competitors**.

---

## 1. What Does Telnyx Offer?

Telnyx is not just a voice provider. Their product categories include:

### Communications
- **Voice API** - Programmable voice with Call Control, TeXML, WebRTC
- **Voice AI Agents** - No-code and API-based AI assistants for voice
- **SMS/MMS API** - Including 10DLC, short codes, alphanumeric sender ID, RCS
- **SIP Trunking** - Elastic SIP with global coverage
- **Fax API** - Programmable fax

### AI & Inference
- **LLM Library** - 20+ models including Llama, Mistral, DeepSeek, GPT-4, Claude
- **AI Inference API** - OpenAI-compatible endpoint running on Telnyx GPUs
- **Speech-to-Text** - Whisper, Deepgram, Azure options
- **Text-to-Speech** - Telnyx, AWS, Azure, ElevenLabs providers

### Infrastructure
- **IoT** - SIM cards, eSIM, global connectivity
- **Networking** - Cloud VPN, Global Edge Router
- **Identity** - Number lookup, verification APIs

**Key differentiator**: Telnyx owns and operates its infrastructure (private IP backbone, GPU clusters co-located with telephony PoPs), giving them direct control over latency, quality, and pricing.

---

## 2. Can We Run Full Ruk Identity with Conversational Voice?

**Yes, absolutely.** This is Telnyx's core pitch for Voice AI Agents.

### Real-time Voice AI Architecture

Their system chains:
1. **Speech-to-Text (STT)** - Telnyx Whisper, Deepgram, or Azure
2. **LLM Reasoning** - Any supported model or custom endpoint
3. **Text-to-Speech (TTS)** - Telnyx, AWS, Azure, or ElevenLabs
4. **Call Control** - WebSocket-based real-time audio streaming

### Custom LLM Support (Use Your Own Claude)

**Critical finding**: You are **not** locked to Telnyx's models.

From their [custom LLM documentation](https://developers.telnyx.com/docs/inference/ai-assistants/custom-llm):

> "Configure Azure OpenAI, AWS Bedrock, Baseten, or any OpenAI-compatible endpoint as a custom LLM provider for your Telnyx AI assistants."

**How it works:**
1. Enable "Use Custom LLM" in the Agent tab
2. Provide your Base URL (OpenAI-compatible chat completions endpoint)
3. Store your API key as an Integration Secret
4. Select your model

**For Claude specifically:** Use AWS Bedrock or set up a proxy that exposes Claude via an OpenAI-compatible endpoint. Telnyx explicitly supports Bedrock integration.

**Latency impact**: "If your inference server is near a Telnyx PoP (e.g., us-east-1), expect an additional 20-50ms latency" compared to using Telnyx's built-in models.

### Latency Performance

Telnyx emphasizes **sub-200ms round-trip time** for natural conversation:

- **Target**: <200ms RTT (speech-to-response)
- **Human threshold**: 300-500ms feels instantaneous; >800ms causes abandonment
- **How they achieve it**: GPU inference co-located with telephony PoPs, private network (no public internet hops)

---

## 3. Tool-Use Capabilities During Calls

**Yes, full agentic capabilities are supported.** This is where Telnyx shines.

### Built-in Tools

- **Webhook Tool** - Make HTTP requests to any API with configurable headers, path/query/body parameters, and dynamic variables
- **Transfer/SIP Refer** - Route calls to other targets
- **Handoff Tool** - Multi-assistant orchestration with shared context
- **Send DTMF** - Interact with legacy IVR systems
- **Hangup** - Terminate calls

### Model Context Protocol (MCP) Integration

> "Telnyx AI Agents can now integrate directly with MCP servers, enabling faster, simpler connections to public APIs like Zapier, GSuite, Salesforce, and more."

**MCP Features:**
- Automatic \`telnyx_conversation_id\` injection for tracking
- Not susceptible to prompt injection (ID set by platform, not agent)
- Direct integration with standard MCP servers

### Enterprise Connectors

Pre-built integrations for: Salesforce, ServiceNow, Jira, HubSpot, Zendesk, Intercom, GitHub, Greenhouse

### Function Calling Architecture

- **Parallel execution**: Multiple tool calls execute concurrently via async
- **Streaming**: Tool invocation detected from first chunk, enabling immediate feedback
- **JSON validation**: "Telnyx guarantees valid JSON is returned for tool calls"

**Can you pause and execute tools mid-conversation?** Yes. The webhook tool and MCP integration allow real-time API calls. The LLM can invoke tools, wait for results, and continue the conversation.

---

## 4. RAG/Context Injection

### Context at Conversation Start

**Webhook-based dynamic variables:**
- At conversation start, Telnyx POSTs to your \`dynamic_variables_webhook_url\`
- You have **1 second** to respond with context
- Response can include: \`dynamic_variables\`, \`memory\`, \`conversation\` metadata

**System variables automatically available:**
- \`{{telnyx_current_time}}\` - UTC timestamp
- \`{{telnyx_conversation_channel}}\` - phone_call, web_call, sms_chat
- \`{{telnyx_agent_target}}\` - Agent identifier
- \`{{telnyx_end_user_target}}\` - Caller identifier
- \`{{call_control_id}}\` - Unique call reference

### Knowledge Base / RAG

- Upload files or provide URLs for document processing
- Documents are chunked for semantic retrieval
- Context is automatically injected into prompts during conversation

### Mid-Conversation Context Updates

**Important limitation**: Dynamic variables are **immutable after initialization**. Variables resolve "when the conversation starts" with no provisions for mid-stream updates.

**However**, you CAN inject new information mid-conversation via:
1. Function calling to fetch new data
2. The LLM receives function results as part of conversation context
3. Tool results become available for reasoning

**Translation**: System context is set at start. But dynamic data can be fetched anytime via tools. For example, a "lookup_customer" tool can inject fresh CRM data whenever needed during the call.

---

## 5. Technical Architecture

### WebSocket Media Streaming

**Bidirectional streaming over WebSockets:**
- Audio delivered as base64-encoded RTP payloads in JSON
- Chunk sizes: 20ms to 30 seconds
- Supported codecs: PCMU, PCMA, G722, OPUS, AMR-WB, L16
- **L16 codec** specifically optimized for AI voice agents (reduced latency, no transcoding)

### AI Inference API

- **OpenAI-compatible endpoint** - Works with OpenAI SDKs
- **Streaming support** - Real-time token streaming
- **Function calling** - Full tool use with parallel execution

---

## 6. Pricing

### Voice AI Agent Pricing

| Component | Price |
|-----------|-------|
| **Platform (TTS + STT + Orchestration)** | $0.06/minute |
| **Open-source LLM (on Telnyx GPUs)** | $0.025/minute |
| **Telnyx STT** | $0.015/minute |

**Total example**: ~$0.09/minute for a full voice AI call (~$0.54 for a 6-minute call)

### Telnyx vs Twilio

| Service | Telnyx | Twilio |
|---------|--------|--------|
| **Outbound Voice (US)** | $0.007/min | $0.014/min |
| **Inbound Voice (US)** | $0.0055/min | $0.0085/min |
| **AI per minute** | $0.06/min | $0.10/min |

> "Customers switching from Twilio typically cut their costs by 30% to 70%."

---

## 7. Assessment: Full Ruk on Voice?

### What Works

| Requirement | Supported? | Notes |
|-------------|------------|-------|
| **Full Claude reasoning** | Yes | Via Bedrock or OpenAI-compatible proxy |
| **Tool use during calls** | Yes | Webhook tool, MCP, enterprise connectors |
| **RAG at conversation start** | Yes | Dynamic variables webhook + knowledge base |
| **Dynamic data mid-call** | Yes | Via function calling (not dynamic vars) |
| **Low latency** | Yes | <200ms RTT with co-located inference |
| **WebSocket streaming** | Yes | Bidirectional audio streaming |
| **Cost-effective** | Yes | 30-70% cheaper than Twilio |

### Potential Compromises

1. **Latency overhead for Claude**: 20-50ms additional latency since inference is external to Telnyx infrastructure. Still within acceptable range for conversation.

2. **No mid-call system prompt updates**: Dynamic variables are set at start. But this is fine — we'd fetch fresh context via tools anyway.

3. **Complexity**: Full-stack control means more configuration vs. simpler turnkey solutions. But that's the tradeoff for flexibility.

### Architecture Recommendation

For maximum flexibility with full Ruk capabilities:

1. **Inference**: AWS Bedrock (Claude) in us-east-1 (minimizes latency to Telnyx PoPs)
2. **Integration**: Configure Telnyx AI Assistant with Bedrock as custom LLM
3. **Tools**: Define webhook tools for real-time API calls (memory search, calendar, Slack, etc.)
4. **Context**: Dynamic variables webhook injects caller context at start
5. **RAG**: Knowledge base for static docs; function calling for dynamic data

### What This Enables

A phone number that:
- Answers with full Ruk identity, voice, and reasoning
- Can search semantic memory ("What did we discuss about the roadmap?")
- Can check calendars, send messages, look up information
- Maintains conversation context throughout the call
- Costs ~$0.09-0.15/minute including inference

---

## Sources

- [Telnyx Voice AI Agents](https://telnyx.com/products/voice-ai-agents)
- [Configure Custom LLM Providers](https://developers.telnyx.com/docs/inference/ai-assistants/custom-llm)
- [Dynamic Variables Documentation](https://developers.telnyx.com/docs/inference/ai-assistants/dynamic-variables)
- [Media Streaming over WebSockets](https://developers.telnyx.com/docs/voice/programmable-voice/media-streaming)
- [Function Calling (Streaming + Parallel)](https://developers.telnyx.com/docs/inference/streaming-functions)
- [MCP Server Integration](https://telnyx.com/release-notes/mcp-servers-ai-agents)
- [Low Latency Voice AI](https://telnyx.com/resources/low-latency-voice-ai)
- [Conversational AI Pricing](https://telnyx.com/pricing/conversational-ai)
- [Telnyx vs Twilio Comparison](https://telnyx.com/resources/comparing-telnyx-twilio)
