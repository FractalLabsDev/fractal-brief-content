# Open Source AI Music Generation: Comprehensive Research Brief

**Date:** 2026-02-08  
**Focus:** Extended generation (full-length songs 3-5+ minutes)  
**Purpose:** Comprehensive coverage of open source alternatives to Suno/Udio

---

## Executive Summary

The open-source AI music generation landscape has matured significantly in 2025-2026, with several models now capable of generating **full-length songs (3-5+ minutes)** with synchronized vocals and instrumentation. The key players are:

| Model | Max Length | Vocals | Open Source | Best For |
|-------|-----------|--------|-------------|----------|
| **ACE-Step 1.5** | 10 minutes | Yes | Apache 2.0 | Speed + quality + low VRAM |
| **YuE** | 5 minutes | Yes | Apache 2.0 | Lyrics-to-song, multilingual |
| **DiffRhythm** | 4m45s | Yes | Apache 2.0 | Fast diffusion-based generation |
| **LeVo/SongGeneration** | 4m30s | Yes | Open source | Voice cloning, dual-track |
| **HeartMuLa** | Full songs | Yes | Apache 2.0 | Style transfer, multi-lingual |
| **MusicGen** | 2+ minutes | No | MIT | Instrumental, melody conditioning |

---

## 1. Full-Song Generation (Critical for Extended Generation)

### ACE-Step 1.5 - The Suno Killer

**Repository:** https://github.com/ace-step/ACE-Step-1.5  
**License:** Apache 2.0  
**Developed by:** ACE Studio + StepFun

**Capabilities:**
- Generates songs **up to 10 minutes** in length
- Full vocals + instrumentation in one pass
- Under 2 seconds per full song on A100
- Under 10 seconds on RTX 3090
- **Runs locally with less than 4GB VRAM**
- LoRA training from just a few songs
- 50+ language support

**Why it matters:** This is currently the most accessible high-quality option for local generation. The extremely low VRAM requirement (4GB) means it runs on consumer hardware.

**Community Tools:**
- [ACE-Step UI](https://github.com/fspecii/ace-step-ui) - Spotify-like interface
- [ComfyUI Integration](https://github.com/billwuhao/ComfyUI_ACE-Step)

---

### YuE - Open Full-Song Generation Foundation Model

**Repository:** https://github.com/multimodal-art-projection/YuE  
**License:** Apache 2.0  
**Developed by:** M-A-P / HKUST

**Capabilities:**
- **Lyrics-to-song generation** (up to 5 minutes)
- Synchronized vocals + accompaniment
- Based on LLaMA2 architecture
- Dual-token technique for vocal/instrumental sync
- Models: 1B and 7B parameters
- Languages: English, Chinese (Mandarin/Cantonese), Japanese, Korean

**Extended Generation:** Supports music continuation through YuE-extend for incremental generation.

**Commercial Use:** Allowed with attribution that songs were AI-generated.

**Community Versions:**
- [YuEGP](https://github.com/deepbeepmeep/YuEGP) - "For the GPU Poor"
- [ComfyUI YuE](https://github.com/smthemex/ComfyUI_YuE)
- [YuE for Windows](https://github.com/sdbds/YuE-for-windows)

---

### DiffRhythm - Diffusion-Based Full-Length Songs

**Repository:** https://github.com/ASLP-lab/DiffRhythm  
**License:** Apache 2.0  
**HuggingFace:** Available

**Capabilities:**
- First open-source **diffusion-based** full-song generator
- Up to **4m45s (285 seconds)** generation
- Synchronized vocals + instrumentals in single pass
- 44.1kHz stereo output
- **Generates in ~10 seconds**
- Non-autoregressive architecture (faster than transformers)

**Inputs:** Lyrics + style prompt

**Training:** 1 million songs dataset

---

### LeVo / SongGeneration (Tencent AI Lab)

**Repository:** https://github.com/tencent-ailab/SongGeneration  
**HuggingFace:** https://huggingface.co/tencent/SongGeneration  
**License:** Open source (NeurIPS 2025)

**Capabilities:**
- Full-length songs up to **4m30s**
- Hybrid tracks (merged) or dual tracks (vocals + accompaniment separate)
- **Zero-shot voice cloning** from 3 seconds of audio
- Outperforms Suno 4.5 in lyric alignment
- Chinese and English support

**Community Tools:**
- [SongGeneration-Studio](https://github.com/BazedFrog/SongGeneration-Studio) - Clean UI with batch processing
- Minimum requirement: 10GB VRAM

---

### HeartMuLa - Music Foundation Models Family

**Repository:** https://github.com/HeartMuLa/heartlib  
**License:** Apache 2.0

**Components:**
- **HeartCLAP**: Audio-text alignment
- **HeartCodec**: Low-frame-rate music tokenizer
- **HeartTranscriptor**: Lyric recognition
- **HeartMuLa**: LLM-based song generation (3B version released)

**Capabilities:**
- 5+ languages (English, Chinese, Japanese, Korean, Spanish)
- Reference audio style transfer
- Fine-grained style control

**Community Tools:**
- [HeartMuLa-Studio](https://github.com/fspecii/HeartMuLa-Studio) - Suno-like interface
- [ComfyUI Integration](https://github.com/filliptm/ComfyUI_FL-HeartMuLa)
- [HeartMuLaGUI](https://github.com/Starnodes2024/HeartMuLaGUI) - Windows GUI

---

## 2. Instrumental Generation

### MusicGen (Meta/AudioCraft)

**Repository:** https://github.com/facebookresearch/audiocraft  
**License:** MIT  
**Documentation:** https://audiocraft.metademolab.com/musicgen.html

**Capabilities:**
- Text-to-music and melody conditioning
- Trained on 20K hours of licensed music
- **Extended generation** via windowing: 30-second chunks with 10-second stride
- Can generate **2+ minute tracks** with proper windowing
- Models: 300M, 1.5B, 3.3B parameters
- 32kHz output

**Extended Generation Technical:**
```python
# extend_stride parameter controls context preservation
# Larger values = less context, shorter values = more compute
```

**No vocals** - instrumental only.

---

### Stable Audio Open

**Repository:** https://github.com/Stability-AI/stable-audio-tools  
**HuggingFace:** stabilityai/stable-audio-open-1.0  
**License:** Open source

**Capabilities:**
- Up to **47 seconds** per generation
- 44.1kHz stereo
- Best for: drum beats, instrument riffs, ambient sounds, foley
- **Fine-tunable** on custom audio data

**Limitations:** Not optimized for full songs, melodies, or vocals.

**Commercial version** (Stable Audio 2.5) supports up to 3 minutes with audio-to-audio.

---

### Riffusion

**Repository:** https://github.com/riffusion/riffusion-hobby  
**HuggingFace:** riffusion/riffusion-model-v1

**How it works:** Fine-tuned Stable Diffusion that generates spectrograms, then converts to audio via inverse Fourier transform.

**Unique feature:** Interpolation between musical prompts for smooth transitions.

**Status:** No longer actively developed, but influential proof-of-concept.

---

### Magenta (Google)

**Repository:** https://github.com/magenta/magenta  
**Magenta Studio:** Ableton Live plugins + standalone apps

**Key Tools:**
- **Continue**: RNN-based melody/drum extension (up to 32 measures)
- **Generate**: VAE trained on millions of melodies
- **Interpolate**: Bridge between two MIDI tracks
- **Drumify**: AI drum pattern generation
- **Groove**: Humanize drum timing/velocity

**Recent (2025):** Magenta RealTime - open-weights model for real-time continuous audio generation.

---

### Jukebox (OpenAI)

**Repository:** https://github.com/openai/jukebox  
**License:** Open source

**Capabilities:**
- Raw audio generation with singing
- Conditioned on genre, artist, lyrics
- Multi-scale VQ-VAE + autoregressive transformers

**Limitations:** Very slow inference, high compute requirements.

---

## 3. Singing Voice & Vocals

### GPT-SoVITS

**Repository:** https://github.com/RVC-Boss/GPT-SoVITS  
**License:** Open source

**Capabilities:**
- **1-minute voice cloning** for high-quality TTS
- **5-second zero-shot** voice cloning
- Cross-lingual: English, Chinese, Japanese, Korean, Cantonese
- Expressive emotional speech generation
- WebUI interface included

**Use case:** Clone any voice for singing or speech synthesis.

---

### RVC (Retrieval-Based Voice Conversion)

**Repository:** https://github.com/RVC-Boss/sovits (SoftVC VITS)  
**Additional:** https://github.com/JackismyShephard/ultimate-rvc

**Capabilities:**
- Speech-to-speech voice conversion
- AI cover songs
- Voice acting and dubbing
- RVC3 is latest version with enhanced features

**Workflow:** Record vocals, convert to target voice.

---

### Amphion

**Repository:** https://github.com/open-mmlab/Amphion  
**License:** MIT

**Capabilities:**
- Unified toolkit for audio, music, and speech generation
- TTS, Text-to-Audio, Singing Voice Conversion (SVC)
- Multiple vocoder options
- Vevo1.5: Unified speech/singing generation (VC, TTS, SVC, SVS, style conversion)

**Best for:** Research and modular pipeline building.

---

### Fish Speech / OpenAudio S1

**Repository:** https://github.com/fishaudio/fish-speech  
**Website:** https://fish.audio

**Capabilities:**
- State-of-the-art TTS
- Voice cloning from 10-30 second reference
- Multilingual support
- Low-latency inference

**Note:** Focused on expressive speech rather than singing, but high quality for narration/dialogue.

---

### Other Voice Synthesis Tools

| Tool | Repository | Best For |
|------|-----------|----------|
| **VoiceCraft** | https://github.com/jasonppy/VoiceCraft | Zero-shot speech editing & TTS |
| **TortoiseTTS** | https://github.com/neonbjb/tortoise-tts | Multi-voice emphasis on quality |
| **Bark** | Included in Coqui TTS | Expressive with non-speech sounds |
| **Coqui TTS** | https://github.com/coqui-ai/TTS | Battle-tested TTS toolkit |

---

## 4. Stem Separation & Audio Processing

### Demucs (Meta)

**Repository:** https://github.com/facebookresearch/demucs  
**License:** MIT

**Capabilities:**
- **Best open-source stem separation** (2025)
- 10-15% better quality than Spleeter
- Waveform-domain processing
- Maintains phase coherence
- `htdemucs_ft` model is current best

**Output:** Vocals, drums, bass, other

---

### Spleeter (Deezer)

**Repository:** https://github.com/deezer/spleeter  
**License:** MIT

**Status:** Feature complete since 2019, no major updates.

**Still useful for:** Fast CPU-only separation, legacy workflows.

---

### Audio Enhancement

| Tool | Repository | Purpose |
|------|-----------|---------|
| **AudioSR** | https://github.com/haoheliu/versatile_audio_super_resolution | Upscale any audio to 48kHz |
| **Resemble Enhance** | Open source | Denoise + enhance + bandwidth extension |
| **ClearerVoice-Studio** | https://github.com/modelscope/ClearerVoice-Studio | Speech enhancement, separation, super-resolution |
| **NovaSR** | Open source | 52KB model, 3600x realtime speed |

---

### Matchering (Open Source Mastering)

**Repository:** https://github.com/sergree/matchering  
**License:** Open source

**Capabilities:**
- Reference-based audio mastering
- Match EQ and loudness to a reference track
- Used by Songmastr web service

---

## 5. MIDI & Composition

### Chord & Accompaniment Generation

| Tool | Repository | Capabilities |
|------|-----------|--------------|
| **ChordSeqAI** | https://chordseqai.com | Browser-based chord progression AI |
| **AccoMontage2** | https://github.com/billyblu2000/AccoMontage2 | Full harmonization from melody |
| **MusicLang Chord** | HuggingFace | Chord scale progression generation |

### MIDI Generation

| Tool | Description |
|------|-------------|
| **Magenta Studio** | MIDI generation/continuation/interpolation |
| **MuseNet** | 4-minute, 10-instrument compositions (OpenAI) |
| **HookPad Aria** | Uses Anticipatory Music Transformer (Google DeepMind) |

---

## 6. Music Inpainting & Editing

### AudioLDM2

**Repository:** https://github.com/haoheliu/AudioLDM2

**Capabilities:**
- Text-to-Audio/Music generation
- Text-to-Speech
- **Super Resolution Inpainting** - regenerate sections

### Other Inpainting Tools

- **NONOTO** (Sony CSL): Symbolic sheet music inpainting
- **Stable Audio 2.5** (commercial): Audio inpainting workflow

---

## 7. Hardware Requirements Summary

### Low VRAM Options (4-8GB)

| Model | VRAM | Speed |
|-------|------|-------|
| **ACE-Step 1.5** | <4GB | ~10s on RTX 3090 |
| **MusicGen (small)** | ~4GB | Moderate |
| **Riffusion** | ~4GB | Fast |

### Medium VRAM (10-16GB)

| Model | VRAM | Notes |
|-------|------|-------|
| **LeVo/SongGeneration** | 10GB+ | Full-length songs |
| **YuE (1B)** | ~10GB | Lyrics-to-song |
| **DiffRhythm** | ~12GB | 4m45s generation |

### High VRAM (24GB+)

| Model | VRAM | Notes |
|-------|------|-------|
| **YuE (7B)** | 24GB+ | Best quality |
| **Jukebox** | 40GB+ | Very resource intensive |

---

## 8. Recommended Pipelines

### Pipeline A: Full Song from Text (Consumer Hardware)

```
1. ACE-Step 1.5 (lyrics + style prompt)
   - Generates full song with vocals
   - <4GB VRAM, ~10 seconds

2. Demucs (if stem separation needed)
   - Extract vocals/instrumentals

3. Matchering (mastering)
   - Match to reference track
```

### Pipeline B: Voice Cloning Workflow

```
1. LeVo/SongGeneration (with reference voice)
   - Zero-shot voice cloning from 3s sample
   - Full song generation

OR

1. YuE/DiffRhythm (generate song)
2. Demucs (extract vocals)
3. RVC/GPT-SoVITS (convert vocals to target voice)
4. Remix stems
```

### Pipeline C: Instrumental + Custom Vocals

```
1. MusicGen (instrumental)
   - Extended generation via windowing

2. GPT-SoVITS (vocal synthesis)
   - Generate/clone singing voice

3. Mix & Matchering (mastering)
```

### Pipeline D: MIDI-First Workflow

```
1. Magenta/ChordSeqAI (chord progressions)
2. Magenta Studio (melody generation/continuation)
3. Any MIDI-to-audio renderer
4. ACE-Step/YuE (add vocals if needed)
```

---

## 9. Quality Comparison (Based on Benchmarks)

From a 6,000-song study with 15,600 human comparisons:

**Top performers for musicality:**
1. ACE-Step - Speed + coherence
2. YuE - Vocal quality + lyric alignment
3. DiffRhythm - Fast diffusion, good quality

**MusicGen** remains strong for instrumental despite being older.

---

## 10. Licensing Quick Reference

| Model | License | Commercial Use |
|-------|---------|----------------|
| ACE-Step | Apache 2.0 | Yes |
| YuE | Apache 2.0 | Yes (with attribution) |
| DiffRhythm | Apache 2.0 | Yes |
| LeVo | Open source | Yes |
| HeartMuLa | Apache 2.0 | Yes |
| MusicGen | MIT | Yes |
| Demucs | MIT | Yes |
| GPT-SoVITS | Open source | Check terms |

---

## Key Takeaways

1. **ACE-Step 1.5 is the current leader** for accessible, high-quality full-song generation. <4GB VRAM makes it run on almost anything.

2. **YuE and DiffRhythm** are strong alternatives with different architectures (transformer vs diffusion).

3. **LeVo** excels at voice cloning with only 3 seconds of reference audio.

4. **For instrumentals**, MusicGen with extended generation windowing can produce 2+ minute tracks.

5. **Stem separation** is a solved problem - Demucs htdemucs_ft is the standard.

6. **Voice cloning** has multiple strong options: GPT-SoVITS (1 minute training), RVC (voice conversion).

7. **All major models are Apache 2.0 or MIT** - fully open for commercial use.

---

## Sources

### Full-Song Generation
- [YuE GitHub](https://github.com/multimodal-art-projection/YuE)
- [ACE-Step 1.5 GitHub](https://github.com/ace-step/ACE-Step-1.5)
- [DiffRhythm HuggingFace Blog](https://huggingface.co/blog/Dzkaka/diffrhythm-open-source-ai-music-generator)
- [LeVo/SongGeneration GitHub](https://github.com/tencent-ailab/SongGeneration)
- [HeartMuLa GitHub](https://github.com/HeartMuLa/heartlib)

### Instrumental & Audio
- [MusicGen/AudioCraft GitHub](https://github.com/facebookresearch/audiocraft)
- [Stable Audio Open](https://stability.ai/news/introducing-stable-audio-open)
- [Magenta](https://magenta.withgoogle.com/)
- [Demucs GitHub](https://github.com/facebookresearch/demucs)

### Voice Synthesis
- [GPT-SoVITS GitHub](https://github.com/RVC-Boss/GPT-SoVITS)
- [Amphion GitHub](https://github.com/open-mmlab/Amphion)
- [Fish Speech GitHub](https://github.com/fishaudio/fish-speech)

### Enhancement & Processing
- [AudioSR GitHub](https://github.com/haoheliu/versatile_audio_super_resolution)
- [Matchering GitHub](https://github.com/sergree/matchering)
- [ClearerVoice-Studio GitHub](https://github.com/modelscope/ClearerVoice-Studio)

### Comparison & Benchmarks
- [SiliconFlow Guide: Best Open Source Music Generation Models](https://www.siliconflow.com/articles/en/best-open-source-music-generation-models)
- [WhiteFiber: Best GPUs for Audio Generation](https://www.whitefiber.com/compare/best-gpus-for-audio-generation-in-2025)
