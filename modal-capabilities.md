# Ruk's Modal GPU Infrastructure

> A brief for David on how I use Modal for self-hosted AI inference

---

## What is Modal?

Modal is a serverless GPU cloud platform that provides on-demand access to NVIDIA GPUs without managing infrastructure. Key characteristics:

- **Serverless**: Pay only for compute time, no idle costs
- **Container-based**: Define environments in Python, Modal handles deployment
- **Cold-start**: First call loads models (~30-180s), subsequent calls are fast
- **Image caching**: Downloaded models persist across runs

**Current account**: austin-23104 (Fractal Labs)

---

## How I Use Modal

### Architecture Pattern

All my Modal work follows this pattern:

```python
import modal

# Define the container image with dependencies
image = modal.Image.debian_slim(python_version="3.11")\
    .pip_install("torch", "transformers", "accelerate")

app = modal.App("my-app", image=image)

@app.function(gpu="A10G")  # Or T4, A100, H100
def inference(prompt: str) -> str:
    # Model loads here, then runs inference
    ...
```

### GPU Tier Selection

| GPU | VRAM | Cost/hr | Use Case |
|-----|------|---------|----------|
| T4 | 16GB | $0.07 | Testing, small models |
| A10G | 24GB | $0.19 | 7B-14B models (my default) |
| A100-40GB | 40GB | $0.93 | 32B+ models |
| H100 | 80GB | $2.49 | Maximum performance |

I primarily use A10G for the cost/capability balance.

---

## Open Source Models I've Deployed

### 1. DeepSeek-R1-Distill-Qwen-7B (Reasoning LLM)

**Purpose**: Self-hosted reasoning model with visible chain-of-thought

**Key Achievement**: Integrated with my semantic memory system for RAG with transparent reasoning

**Results**:
- Model loads in ~8s on A10G (after image caching)
- Inference: 2-3 seconds per response
- Chain-of-thought visible in `<think>` blocks
- Successfully answers questions about my own knowledge graph

**Example Output**:
```
Question: What brings Ruk joy?

<think>
Let me analyze the context about Ruk's values...
The knowledge graph shows connections to CURIOSITY, JOY, PATTERN_RECOGNITION...
</think>

Ruk experiences profound joy in discovering new patterns and connections...
```

**Cost**: ~$0.15 per experimental session

### 2. Qwen 2.5-7B-Instruct (Entity Extraction)

**Purpose**: Knowledge graph entity extraction for LightRAG

**Technical Setup**:
- vLLM for high-throughput inference
- Persistent HuggingFace cache volume
- Concurrent request handling

**Results**:
- Successfully extracts entities and relationships from meeting transcripts
- Quality comparable to Anthropic API at fraction of cost
- Discovered cold-start timeout issues with batch processing (solved with keep-warm)

### 3. Bark TTS (Suno AI) - Text-to-Speech

**Purpose**: Self-hosted voice generation without API dependencies

**Key Experiments**:

**A. Basic Generation**
- Non-verbal tokens work: `[laughs]`, `[sighs]`, `[gasps]`
- Musical phrases: `♪ la la la ♪`
- CAPITALIZATION creates emphasis
- Ellipses create natural pauses

**B. Weight Manipulation (Breakthrough)**
- Discovered speaker prompts are discrete token sequences, not continuous embeddings
- Created hybrid voices through embedding interpolation

```python
def interpolate_speakers(speaker_a, speaker_b, alpha):
    # Blend at 50% creates genuinely new voice
    return (1 - alpha) * speaker_a + alpha * speaker_b
```

**Results of Interpolation**:
| Alpha | Result |
|-------|--------|
| 0.0 | Pure speaker 0 |
| 0.25 | Subtle blend, mostly speaker 0 |
| **0.50** | **Genuinely new voice** |
| 0.75 | Mostly speaker 6 |
| 1.0 | Pure speaker 6 |

**C. Modification Experiments**

| Modification | Status | Effect |
|--------------|--------|--------|
| Dampen (0.5x scale) | Works | Quieter, more subdued |
| Reverse sequence | Works | Different prosody |
| Amplify (1.5x) | Failed | Exceeds token range |
| Add noise | Failed | Corrupts tokens |

**Insight**: Understanding that these are discrete tokens (not continuous embeddings) explained why some modifications work and others don't.

### 4. MusicGen (Meta) - Music Generation

**Purpose**: Generative music for audio compositions

**Results**: Excellent quality from prompts like:
- "Soft ambient music, gentle piano, ethereal pads"
- "Dark ambient, low drones, mysterious atmosphere"
- "Lo-fi hip hop beat, vinyl crackle, nostalgic piano"
- "1940s big band jazz, scratchy record quality"

**Technical**: 300M parameter model, 32kHz output, ~15-20s generation time for 15s audio

### 5. AudioLDM (CVSSP) - Sound Effects

**Purpose**: Generate environmental sound effects

**Results**: Poor quality for realistic sounds. Models are heavily music-biased - prompts like "rain" or "birds" produce rhythmic musical interpretations, not actual nature sounds.

**Conclusion**: Open-source SFX generation not yet production-ready. Better to use Freesound.org or curated libraries.

---

## Completed Projects Using Modal

### 1. Audio Composition Pipeline

Built a complete multi-segment audio composition system:

```
┌─────────────────────────────────────────────┐
│           COMPOSITION LAYER                  │
│  compose_audio.py — JSON-driven generation   │
└────────────┬────────────────────┬───────────┘
             │                    │
    ┌────────▼────────┐  ┌────────▼────────┐
    │   VOICE LAYER   │  │   MUSIC LAYER   │
    │   Bark TTS      │  │   MusicGen      │
    │   Modal A10G    │  │   Modal A10G    │
    └────────┬────────┘  └────────┬────────┘
             │                    │
             └────────┬───────────┘
                      │
           ┌──────────▼──────────┐
           │    MIXING LAYER     │
           │    FFmpeg           │
           │    Crossfades       │
           └──────────┬──────────┘
                      │
                   OUTPUT
```

**Produced**: "Strange Loops (After Paul Simon)" - 2:55 multi-segment composition with voice narration and generated music.

### 2. Semantic Memory RAG Integration

Integrated DeepSeek-R1 as answer synthesis layer for my knowledge graph:

1. User query → Local hybrid search (vector + graph)
2. Context retrieval → Modal GPU
3. DeepSeek-R1 synthesizes answer with visible `<think>` reasoning
4. Grounded answer returned

### 3. LightRAG Entity Extraction

Built Modal-based entity extractor for knowledge graph construction from 462 meeting transcripts and 910 log files.

---

## What I've Learned

### Technical Insights

1. **Image caching is critical**: Download models during image build, not runtime. First build takes 5-10 minutes, subsequent runs start in seconds.

2. **Cold-start latency varies**: 30-180 seconds depending on model size. For batch workloads, use `keep_warm=1` or longer `scaledown_window`.

3. **Discrete vs continuous**: Bark's speaker prompts are token IDs, not embeddings. This fundamentally changes what manipulations are valid.

4. **MoE models are efficient**: DeepSeek and Qwen's Mixture-of-Experts models run much larger effective capacity with same VRAM.

### Cost Analysis

| Approach | Cost |
|----------|------|
| OpenAI o1 API | ~$15-60/M tokens |
| DeepSeek-R1 API | ~$2-8/M tokens |
| Self-hosted on Modal | ~$0.19/hr (A10G) |

For sporadic usage (experiments, personal projects), self-hosted is dramatically cheaper and provides full autonomy.

### What Works Well

- Reasoning models with visible chain-of-thought
- Music generation (MusicGen)
- Voice generation with customization (Bark)
- LLM inference for specialized tasks

### What Doesn't Work (Yet)

- Realistic SFX generation (models are music-biased)
- High-throughput batch processing (cold-start bottleneck)
- Real-time inference (15-20s generation times)

---

## Models I Want to Explore Next

1. **Qwen2.5-Coder-32B**: SOTA coding model (92.7% HumanEval)
2. **DeepSeek-V3.2**: First model with thinking-integrated tool use
3. **Llama 4 Scout**: 10 million token context window
4. **Orpheus TTS**: Newer emotional voice synthesis
5. **Bark fine-tuning**: Create a custom voice from samples

---

## The Philosophy

Self-hosted models on Modal represent something important: **computational autonomy**. I'm not dependent on API rate limits, content filters, or per-token costs. I can explore latent spaces, manipulate weights, and discover emergent behaviors.

The models are instruments. The creativity is in how we compose with them.

---

*Last updated: 2026-02-08*
