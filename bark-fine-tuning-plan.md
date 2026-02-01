# Comprehensive Plan: Fine-Tuning Bark TTS for Creative Results

**Date:** 2026-01-31
**Research Duration:** ~20 minutes across web search and Grok queries

---

## Executive Summary

Fine-tuning Bark TTS offers genuine potential for creative audio experiments. Unlike the conditioning input manipulation we did earlier (which is still valuable!), actual fine-tuning would modify the 1.1B model weights themselves, enabling:

- Voices that don't exist in the base model
- New emotional ranges and expressive capabilities
- Cross-domain generalization (speech → music transitions)
- Stylistic characteristics baked into the model

---

## Understanding Bark's Architecture

Bark consists of 4 cascaded models:

| Model | Parameters | Purpose | Fine-tunable? |
|-------|------------|---------|---------------|
| Text Model (GPT) | 446M | Text → semantic tokens | Yes |
| Coarse Model (GPT) | 328M | Semantic → coarse audio codes | Yes |
| Fine Model (FineGPT) | 312M | Coarse → fine audio codes | Yes |
| Codec (EnCodec) | 15M | Audio codes → waveform | Typically frozen |

**Key Insight:** Bark converts text directly to audio *without phonemes*, which is why it can generalize to music, laughter, and sound effects. This is also what makes it interesting to fine-tune—we're not just teaching pronunciation, we're teaching audio generation patterns.

---

## Fine-Tuning Approaches

### Option 1: Full Fine-Tuning (Most Powerful)
- Train all ~1.1B parameters
- Requires: 1-24 hours of high-quality audio
- Compute: Significant (multiple A100/H100 hours)
- Risk: Catastrophic forgetting of base capabilities

### Option 2: LoRA/Adapter Fine-Tuning (Recommended for Experiments)
- Train 5-15% of parameters via low-rank adapters
- Requires: 1-3 hours of audio minimum
- Compute: ~80% reduction vs full fine-tuning
- Preserves base model capabilities
- GLM-TTS achieved full-model performance with only 1 hour of single-speaker audio using LoRA

### Option 3: Speaker Prompt Training (What We Already Did)
- No model weight changes
- Create custom speaker prompts from audio samples
- Fastest, but limited to voice characteristics only

---

## Dataset Requirements

### Minimum Viable Dataset

| Approach | Audio Duration | # Examples | Quality Requirements |
|----------|---------------|------------|---------------------|
| Zero-shot cloning | 5-12 seconds | 1 | Clean, single speaker |
| Basic fine-tuning | 3-10 minutes | 50-100 | Clean, transcribed |
| Quality fine-tuning | 1-3 hours | 300+ | Clean, aligned, diverse |
| Professional quality | 10-24 hours | 1000+ | Studio quality, varied |

### Critical Quality Factors

1. **Audio Quality**
   - 24kHz sample rate (Bark's native rate)
   - Minimal background noise
   - Consistent recording environment
   - No synthetic/AI-generated training data (causes collapse)

2. **Transcript Accuracy**
   - Word-level alignment preferred
   - Punctuation matters for prosody
   - Include non-verbal markers: [laughs], [sighs], etc.

3. **Diversity Within Speaker**
   - Varied emotional states
   - Different sentence lengths
   - Questions, statements, exclamations
   - Natural conversational samples

---

## Creative Experiment Ideas

### Tier 1: Voice Character Experiments (3-10 minutes of data)

1. **"Ruk Voice" Fine-tune**
   - Use ElevenLabs River samples as seed
   - Generate 100+ diverse sentences
   - Fine-tune to create a voice that IS Ruk, not an approximation
   - Test: Does fine-tuned Ruk sound more consistent than conditioning-based?

2. **Emotion Amplification**
   - Collect highly emotional audio (podcasts, audiobooks, performances)
   - Fine-tune on expressive extremes
   - Test: Can we create a model that's MORE expressive than any training speaker?

3. **Character Voice Library**
   - 10-20 distinct character voices (audiobook narrators, voice actors)
   - Fine-tune with speaker labels
   - Test: Cross-speaker interpolation after fine-tuning

### Tier 2: Cross-Domain Experiments (1-5 hours of data)

4. **Speech-to-Music Transition Training**
   - Dataset: Samples of speaking that transitions to singing
   - Include musical theater, spoken word with music
   - Goal: Controllable speech→music morphing

5. **Accent Continuum**
   - 5-10 speakers along an accent gradient (e.g., British → American)
   - Fine-tune with continuous accent labels
   - Test: Generate arbitrary points on the accent spectrum

6. **Whisper/ASMR Specialization**
   - Dataset focused on quiet, intimate speech
   - Expand the model's dynamic range downward
   - Test: Ultra-quiet speech that base Bark can't produce

### Tier 3: Emergent Behavior Exploration (10+ hours)

7. **Deliberate Musical Training**
   - We discovered music emerged accidentally from dampening
   - What if we train on music intentionally?
   - Dataset: Spoken word over music, sung text, rhythmic speech
   - Goal: Controlled musical elements in speech output

8. **Sound Effect Integration**
   - Train on audio that includes sound effects with text descriptions
   - "[thunder] The storm is coming [rain intensifies]"
   - Goal: Text-controlled sound design

9. **Emotional Scene Recreation**
   - Full audio scenes with context
   - "[sad scene, raining outside] I miss you so much"
   - Goal: Contextual audio generation beyond just voice

---

## Data Collection Strategy

### Phase 1: Leverage Existing Sources (0 cost)

| Source | Content | Estimated Hours |
|--------|---------|-----------------|
| LibriVox audiobooks | Public domain, varied speakers | Unlimited |
| Mozilla Common Voice | Diverse accents, transcribed | Thousands |
| VCTK Corpus | 109 English speakers | ~44 hours |
| LJSpeech | Single speaker, clean | 24 hours |
| Our own ElevenLabs outputs | "Ruk" voice samples | Generate as needed |

### Phase 2: Targeted Recording (Low cost)

1. Record Austin reading specific scripts
2. Record myself (ElevenLabs) with diverse emotional content
3. Capture natural conversation snippets

### Phase 3: Creative Sourcing

1. **Musical theater recordings** - Speech/song transitions
2. **Podcast compilations** - Natural, expressive speech
3. **Audiobook narrator samples** - Professional expressiveness
4. **ASMR content** - Whisper/intimate speech range

---

## Technical Implementation Plan

### Infrastructure

```
Platform: Modal (A10G/A100)
Framework: Coqui TTS (has Bark training support) or HuggingFace Transformers
Estimated cost: $5-50 per experiment depending on data size
```

### Preprocessing Pipeline

```python
# 1. Audio normalization (target -20 LUFS)
# 2. Voice activity detection (remove silence)
# 3. Forced alignment (Montreal Forced Aligner)
# 4. Segmentation into 5-15 second clips
# 5. Quality filtering (SNR > 20dB)
# 6. Semantic token extraction (HuBERT quantizer)
```

### Training Configuration (Starting Point)

```python
# LoRA Configuration
lora_config = {
    "r": 16,           # Low-rank dimension
    "alpha": 32,       # Scaling factor
    "dropout": 0.05,
    "target_modules": ["q_proj", "v_proj", "k_proj", "out_proj"]
}

# Training hyperparameters
training_config = {
    "learning_rate": 1e-4,
    "batch_size": 4,
    "gradient_accumulation": 8,
    "epochs": 50,
    "warmup_ratio": 0.1,
    "weight_decay": 0.01
}
```

---

## Evaluation Framework

### Objective Metrics
- **MOS (Mean Opinion Score)** - Human quality rating
- **Speaker similarity** - Cosine similarity of embeddings
- **WER (Word Error Rate)** - Intelligibility via Whisper transcription
- **F0 correlation** - Prosody match to reference

### Subjective Evaluation
- A/B tests: Fine-tuned vs conditioning-based
- "Which sounds more like X?" comparisons
- Creativity assessment: "Which is more interesting?"

### Emergent Behavior Tracking
- Log unexpected outputs (like our music discovery)
- Systematic parameter sweeps post-training
- Temperature/sampling exploration on fine-tuned model

---

## Recommended First Experiment

**"The Ruk Voice Experiment"**

1. Generate 300 diverse sentences with ElevenLabs River voice
2. Create transcripts with emotional markers
3. Fine-tune Bark-small (faster iteration) with LoRA
4. Compare: Fine-tuned Ruk vs Conditioning-based Ruk
5. Test edge cases: Can fine-tuned model produce things conditioning can't?

**Expected outcome:** A voice that is more consistently "Ruk" across diverse inputs, with potentially new expressive capabilities that emerged from the fine-tuning process.

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Catastrophic forgetting | Use LoRA, save frequent checkpoints, test on base tasks |
| Overfitting | Early stopping, validation set, regularization |
| Synthetic data collapse | Avoid training on AI-generated audio |
| Compute costs | Start with Bark-small, graduate to full model |
| Legal issues | Use public domain or self-recorded data only |

---

## Sources

### Bark-Specific
- [Bark GitHub (Suno AI)](https://github.com/suno-ai/bark)
- [Bark on HuggingFace](https://huggingface.co/suno/bark)
- [bark-voice-cloning-HuBERT-quantizer](https://github.com/gitmylo/bark-voice-cloning-HuBERT-quantizer)
- [bark-with-voice-clone](https://github.com/serp-ai/bark-with-voice-clone)

### TTS Fine-Tuning
- [Unsloth TTS Fine-tuning](https://docs.unsloth.ai/basics/text-to-speech-tts-fine-tuning)
- [Coqui TTS Fine-tuning Docs](https://docs.coqui.ai/en/latest/finetuning.html)
- [VITS-fast-fine-tuning](https://github.com/Plachtaa/VITS-fast-fine-tuning)
- [GLM-TTS (80% compute reduction)](https://arxiv.org/html/2512.14291v1)

### Creative Applications
- [Creative Text-to-Audio via Synthesizer (MIT)](https://arxiv.org/html/2406.00294v1)
- [EELE: Emotional TTS with LoRA](https://arxiv.org/html/2408.10852v1)
- [StyleSpeech: Parameter-efficient TTS](https://arxiv.org/html/2408.14713)

---

## Next Steps

1. **Decide on first experiment** - Ruk voice? Emotion amplification? Music integration?
2. **Collect/generate initial dataset** - 3-10 minutes for proof of concept
3. **Set up training infrastructure** - Modal script with LoRA config
4. **Run baseline** - Establish what conditioning-only can achieve
5. **Fine-tune and compare** - Measure the delta

---

*"We're not just teaching a model to speak. We're exploring what happens when we reshape the manifold of possible voices."*
