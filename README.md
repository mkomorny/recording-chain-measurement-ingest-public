# Recording Chain & Measurement Ingest

> **Tri-Platform Agent Skill** for Claude Desktop / Code, Grok CLI, and Google Antigravity / Gemini.  
> Microphone and Preamp Response Compensation — Ingests audio files to extract objective acoustic metrics via Praat, Parselmouth, and Librosa, while decoupling recording chain bias, microphone polar patterns, proximity effect, and room impulse response.

---

## Overview

`recording-chain-measurement-ingest` provides standardized capabilities across conversational AI platforms:
- Acoustic metric extraction pipeline (F0, formants F1-F4, jitter, shimmer, HNR, spectral tilt).
- Microphone frequency response equalization and de-emphasis compensation.
- Proximity effect low-frequency boost normalization algorithms.
- Room impulse response (RIR) deconvolution and dry vocal estimation.
- Standardized acoustic_ground_truth.json schema for reference normalization.
- Tri-platform NTFS junctions connecting Claude, Grok, and Antigravity.
- Standard documentation suite: SKILL.md, HOW_TO_USE.txt, IMPLEMENTATION_PLAN.md, and README.md.

---

## Tri-Platform Architecture

Zero duplicate code across platforms. Canonical source resides in `.agents`:
- **Canonical Source**: `C:\Users\<you>\.agents\skills\recording-chain-measurement-ingest`
- **Claude**: `C:\Users\<you>\.claude\skills\recording-chain-measurement-ingest` (NTFS Junction)
- **Grok**: `C:\Users\<you>\.grok\skills\recording-chain-measurement-ingest` (NTFS Junction)
- **Antigravity**: `C:\Users\<you>\.gemini\config\skills\recording-chain-measurement-ingest` (NTFS Junction)

---

## Documentation Suite

- [`HOW_TO_USE.txt`](HOW_TO_USE.txt): Standardized user guide and prompt templates.
- [`IMPLEMENTATION_PLAN.md`](IMPLEMENTATION_PLAN.md): Feature checklist and roadmap.
- [`App Changes Tracker.html`](App%20Changes%20Tracker.html): Interactive visual version log.
- [`SKILL.md`](SKILL.md): Agent system instructions and triggers.
