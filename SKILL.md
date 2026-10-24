---
name: recording-chain-measurement-ingest
description: Ingests audio files to extract objective acoustic metrics via Praat, Parselmouth, and Librosa, while decoupling recording chain bias, microphone polar patterns, proximity effect, and room impulse response (RIR). Trigger with 'ingest audio measurements for', 'calibrate recording chain of', 'extract acoustic ground truth from', or '/measurement-ingest'.
disable-model-invocation: true
---

# Recording Chain & Measurement Ingest

## Purpose & Scope

This skill provides the **objective acoustic ground-truth and calibration layer**. A voice profile is only as honest as the measurement chain that produced it. Audio recorded on an iPhone in an echoey conference room exhibits false spectral tilt, inflated low-end proximity boost, and distorted reverberant decay that must never be mistaken for the speaker's biological voice. This skill ingests audio, extracts forensic measurements via Praat/Parselmouth/librosa, and explicitly decouples microphone and room acoustics from biological voice traits.

This skill owns:
- Automated metric ingestion schema for Praat/Parselmouth, librosa, and PyWorld
- Separation of empirical measurements vs. estimated/inferred values
- Microphone proximity effect and polar pattern de-biasing
- Room Impulse Response (RIR), RT60 reverberation time estimation, and de-reverberation
- Calibrated Sound Pressure Level (SPL) references and gain staging
- Acoustic calibration metadata blocks that feed biometric profile sections 1–4

This skill does **not** synthesize audio (owned by `synthesis-control-compiler`) or score voice clones (owned by `clone-verification-match-score`).

---

## Activation Triggers

Activate automatically when:
- User provides audio files, recordings, or Praat output for acoustic profiling (e.g., *"ingest measurements from [file.wav]"*, *"calibrate recording chain for [audio]"*).
- User triggers `/measurement-ingest [audio_source]`.
- Calibrating raw acoustic data before writing `voice-biometric-profiler` reports.
- Isolating microphone coloration, room echo, or clipping artifacts from true biological voice signatures.

---

## Mandatory Report Schema & Structure

### 1. Ingested Acoustic Metric Summary (Parselmouth / Praat)
- **F0 Distribution:** Min, max, mean, median, standard deviation (Hz & st).
- **Perturbation Metrics:** Local Jitter (%), RAP (%), PPQ5 (%), Local Shimmer (%), Shimmer (dB), APQ11 (%).
- **Spectral Purity:** Mean Harmonics-to-Noise Ratio (HNR in dB), Cepstral Peak Prominence (CPPS in dB).
- **Long-Term Average Spectrum (LTAS):** Alpha ratio (0–1 kHz vs 1–5 kHz in dB), Hammarberg Index (dB).
- **Formant Architecture:** F1, F2, F3, F4 medians and bandwidths (B1–B4 in Hz) sampled over voiced vowels.

### 2. Recording Chain & Transducer De-biasing
- **Microphone Distance & Proximity Effect:** Estimated mouth-to-capsule distance (cm); low-frequency proximity boost compensation (-dB below 200 Hz).
- **Transducer Class & Polar Pattern:** Cardioid condenser vs. dynamic vs. lavalier vs. smartphone MEMS microphone.
- **Frequency Response Equalization:** Inverse filtering applied to neutralize microphone peaks.

### 3. Room Acoustics & Environmental Calibration
- **Estimated Reverberation Time (RT60 in ms):** Decay time calculation; detection of early reflections.
- **Direct-to-Reverberant Ratio (DRR in dB):** Acoustic clarity indicator.
- **Background Noise Floor & SNR:** Ambient noise SPL (dBA), SNR (dB), and mains hum (50/60 Hz) presence.

### 4. Empirical vs. Inferred Data Ledger
- Strict forensic ledger classifying every metric as either **[EMPIRICAL]** (measured directly from calibrated audio with known hardware) or **[ESTIMATED]** (inferred from uncalibrated YouTube/broadcast audio or anatomical modeling).

---

## Machine Output Block

```json
{
  "$schema": "https://json-schemas.voice/measurement-ingest.v1.json",
  "session_id": "SESSION_INGEST_001",
  "calibration_status": "CALIBRATED_DEBIASED",
  "empirical_measurements": {
    "f0_median_hz": 159.4,
    "f0_std_hz": 22.1,
    "jitter_local_percent": 0.88,
    "shimmer_local_db": 0.38,
    "hnr_mean_db": 16.4,
    "cpps_mean_db": 12.8,
    "formants_hz": {"f1": 578, "f2": 924, "f3": 2345, "f4": 3480}
  },
  "chain_compensation": {
    "estimated_mic_distance_cm": 15,
    "proximity_correction_applied_db": -2.5,
    "transducer_type": "DYNAMIC_BROADCAST_SM7B",
    "snr_db": 34.2
  },
  "room_acoustics": {
    "rt60_estimated_ms": 280,
    "drr_db": 14.5,
    "de_reverberation_applied": true
  },
  "confidence_ledger": {
    "f0_measurements": "EMPIRICAL_HIGH",
    "formants": "EMPIRICAL_HIGH",
    "subglottal_pressure": "INFERRED_ANATOMICAL",
    "vocal_tract_length": "INFERRED_DISPERSION"
  }
}
```

---

## Biometric & Personality Crosswalk

| Ingest Metric | Core Biometric Section | Personality Profile Section | Calibration Target |
| :--- | :--- | :--- | :--- |
| F0 & Perturbation | §1 F0 & §4 Perturbation | §4.1 Dominant Attitude | Praat Parselmouth Direct Output |
| CPPS & HNR | §3 Spectral Characteristics & HNR | §2.2 Directness & Honesty | De-biased Clean Phonation Value |
| Formants F1–F4 | §3 Formants & Vowel Space | §3.1 Lexical Range | Vowel Formant Normalization |
| Proximity Correction | §8 Environmental & Recording | §1 Executive Summary | Transducer Inverse Filter |

---

## Calibration Reference Example: Queens Orator Baseline

- **Audio Sourcing:** Presidential weekly address wav audio (broadcast line feed) combined with live arena microphone samples.
- **De-biasing Correction:** Arena speech exhibited +4.5 dB proximity boost at 120–180 Hz due to tight handheld Shure SM58 usage; inverse low-shelf filter applied at 150 Hz to reveal true biological glottal fundamental.
- **RT60 Neutralization:** Arena reflections (RT60 ~1400 ms) separated via spectral subtraction to prevent room reverberation from corrupting HNR and cepstral peak prominence measurements.
- **Calibrated F0:** True median baseline F0 isolated at 159.4 Hz (standard deviation 22.1 Hz).

---

## Generation Rules

1. Every quantitative value must be explicitly tagged as `[EMPIRICAL]` or `[ESTIMATED]`.
2. Always apply proximity and reverberation corrections when analyzing uncalibrated field recordings.
3. Every profile must output the machine-readable `measurement_ingest.json` block.
