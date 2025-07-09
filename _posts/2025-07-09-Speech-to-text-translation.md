---
layout: post
title: LLM Based Speech-to-Text Translation Part-1
tags: ["speech recognition","machine learning","bayes theorem"]
mathjax: true
---

# Making LLMs Better Many-to-Many Speech-to-Text Translators with Curriculum Learning: A Technical Overview

**Authors:** Yexing Du, Youcheng Pan, Ziyang Ma, Bo Yang, Yifan Yang, Keqi Deng, Xie Chen, Yang Xiang, Ming Liu, Bing Qin  
**arXiv:** [2409.19510v2](https://arxiv.org/abs/2409.19510)

---

## Introduction

Multimodal Large Language Models (MLLMs) have recently demonstrated strong performance in Speech-to-Text Translation (S2TT) tasks, especially in English-centric scenarios. However, scaling these models to many-to-many translation directions is hampered by the scarcity of parallel speech-text data for most language pairs. This paper introduces a **three-stage curriculum learning strategy** that leverages the machine translation (MT) capabilities of LLMs and adapts them for S2TT, enabling robust performance even in low-resource settings.

---

## Problem Statement

Traditional S2TT systems typically use a **cascaded approach**:

1. **ASR (Automatic Speech Recognition):** Transcribe speech to text.
2. **MT (Machine Translation):** Translate the transcribed text to the target language.

While effective, this pipeline suffers from error propagation and increased latency. End-to-end MLLMs can mitigate these issues but require large-scale parallel S2TT data, which is often unavailable for many language pairs.

---

## Proposed Solution: Curriculum Learning for S2TT

The authors propose reframing S2TT as a **Speech Recognition and Translation (SRT)** task, where the model is trained to output both the transcription and the translation from speech input. The core innovation is a **three-stage curriculum learning strategy**:

### 1. ASR Stage

- **Objective:** Train the MLLM for multimodal alignment and transcription.
- **Input:** Speech + instruction.
- **Output:** Transcription.

### 2. Speech-Aided Machine Translation (SMT) Stage

- **Objective:** Enhance cross-lingual capabilities by providing both speech and its transcription as input, prompting the model to generate translations.
- **Input:** Speech + transcription + instruction.
- **Output:** Translation.

### 3. SRT Stage

- **Objective:** Finalize the model for S2TT by training it to generate both transcription and translation from speech alone.
- **Input:** Speech + instruction.
- **Output:** Transcription + translation.

Each stage resumes from the checkpoint of the previous one, ensuring knowledge transfer and stable optimization.

---

## Model Architecture: LLM-SRT

The **LLM-SRT** model consists of:

- **Speech Encoder:** A frozen Whisper encoder extracts high-dimensional features from audio.
- **Speech Adapter:** A Q-Former compresses and projects speech features to match the LLM’s hidden dimension, enabling efficient multimodal fusion.
- **LLM Backbone:** Qwen2.5 (3B, 7B, 32B) serves as the language model, processing concatenated speech and text embeddings.

**Instruction Design:** Minimalist, language-tagged instructions (e.g., `<|eng|><|zho|>`) are used to distinguish tasks and segment outputs.

---

## Experimental Setup

- **Datasets:**
  - **FLEURS:** 102 languages, ~10 hours of speech per language (low-resource).
  - **CoVoST-2:** Large-scale multilingual S2TT corpus (high-resource).
- **Baselines:**
  - Cascaded ASR+MT systems (Whisper + Qwen2.5).
  - End-to-end models: SeamlessM4T-V2, Qwen2-Audio.
- **Metrics:**
  - **WER** for ASR.
  - **BLEU** for S2TT.

---

## Results

### Low-Resource Setting (FLEURS)

- **LLM-SRT-3B** achieves a BLEU score of 20.6 (vs. 8.6 for baseline-3B and 20.2 for SeamlessM4T-V2).
- **LLM-SRT-32B** achieves 24.6 BLEU, outperforming all baselines.
- The model supports **15 × 14** translation directions with less than 10 hours of speech per language.

### High-Resource Setting (CoVoST-2)

- LLM-SRT models scale well with more data, achieving competitive or superior BLEU scores compared to state-of-the-art models.

### Ablation Studies

- Removing any stage from the curriculum (ASR, SMT, or SRT) leads to significant performance drops, confirming the necessity of the three-stage approach.
- Training only the speech adapter (freezing the LLM) yields strong results; unfreezing the LLM (e.g., via LoRA) further improves performance.

### Inference Speed

- The optimized speech adapter enables up to **3x faster inference** compared to Qwen2-Audio, with significant reductions in input token length.

---

## Key Takeaways

- **Curriculum learning** is highly effective for adapting LLMs to many-to-many S2TT, especially in low-resource scenarios.
- The **SRT formulation** bridges the gap between MT and S2TT, leveraging the LLM’s translation capabilities.
- **Scaling laws** hold: larger LLMs yield better S2TT performance.
- The approach is robust across both low- and high-resource settings, and supports a wide range of languages.

---

## Limitations

- The method’s performance is bounded by the underlying LLM’s MT capabilities and language coverage.
- Languages not well-supported by the LLM or with poor MT performance remain challenging.

---

## Conclusion

This work demonstrates that **curriculum learning** can unlock the many-to-many S2TT potential of MLLMs, even with limited parallel data. By systematically transferring MT capabilities to S2TT via ASR, SMT, and SRT stages, the proposed LLM-SRT model achieves state-of-the-art results across a wide range of languages and data regimes. The code and models are available at [github.com/yxduir/LLM-SRT](https://github.com/yxduir/LLM-SRT).

---

**References:**  
For a full list of references and technical details, see the [original paper](https://arxiv.org/abs/2409.19510).

---

*If you found this summary useful, follow for more technical deep-dives into cutting-edge AI research!*
