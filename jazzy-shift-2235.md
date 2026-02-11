## Llama Inference Provider Research

### The Problem
Replicate cold-start latency (3-5 min for non-405B models).

---

### Top Recommendations

**1. Groq** — Fastest option (sub-300ms TTFT)
- LPU silicon = no cold starts ever
- Llama 3.3 70B: $0.59/$0.79 per M tokens
- Drawback: No 405B support
- Native function calling, 128K context

**2. Together AI** — Best all-rounder
- Full Llama lineup: 8B/70B/405B, 3.2, 3.3, Llama 4
- Serverless ~$0.88/M for 70B, warm pools
- Dedicated endpoints: always warm, 99.9% SLA, 43% cheaper at scale
- OpenAI-compatible API

**3. Cerebras** — Bleeding edge for 405B
- 969 tokens/sec for 405B (18x faster than GPU)
- 2,100 tokens/sec for 70B
- Meta partnership, always warm
- ~$0.60/M for 70B

**4. DeepInfra** — Budget option
- Cheapest: 70B at $0.23/$0.40 per M, 405B at $1.79/M
- All model sizes supported
- Less latency focus but no cold starts on popular models

**5. Fireworks AI** — Production-ready
- 4x faster than vLLM
- Batch inference at 50% serverless pricing
- Dedicated: $3.89/hr per A100

---

### Quick Comparison

| Provider | 70B Price | TTFT | 405B? | Cold Start |
|----------|-----------|------|-------|------------|
| Groq | $0.59/$0.79 | 0.3s | No | None |
| Together | ~$0.88 | 0.5s | Yes | Warm pools |
| Cerebras | ~$0.60 | 0.24s | Yes | None |
| DeepInfra | $0.23/$0.40 | — | Yes | Warm pools |
| Fireworks | ~$0.90 | 0.4s | Yes | Warm pools |

---

### Practical Recommendation for talkwise-oracle

**Multi-provider approach:**
- **Groq** for 70B (fastest, always warm)
- **Together AI** or **Cerebras** for 405B
- Consider **OpenRouter** as aggregator (unified API, automatic fallbacks, 5.5% fee)

**Or single provider:**
- **Together AI** covers everything with dedicated endpoint option for guaranteed warm
