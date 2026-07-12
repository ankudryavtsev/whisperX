# Feature Specification: Forced Phoneme Alignment

**Status**: migrated

**Created**: 2026-07-12

**Feature Branch**: `main`

## User Scenarios & Testing

### User Story 1 - Word-Level Timestamps for Known Languages (Priority: P1)
As a whisperX user, I want my transcription segments to be aligned with the audio at a word level, so that I can see exactly when each word was spoken in subtitles or JSON logs.

**Why this priority**: Essential to improve Whisper's default coarse-grain sentence/utterance timestamps to precise word-level timings.

**Independent Test**: Verified via CLI or Python API by passing a standard WAV audio and transcript, then checking that each word in `word_segments` contains `"start"`, `"end"`, and `"score"` fields.

**Acceptance Scenarios**:
1. **Given** a valid English audio recording and its transcription text,
   **When** running forced alignment with the default English Wav2Vec2 model,
   **Then** all words in the output are assigned valid start and end timestamps.

---

### User Story 2 - Character-Level Alignment Option (Priority: P2)
As a developer, I want to obtain character-level timestamps so that I can create detailed visual text-to-speech alignment interfaces or precise subtitle effects.

**Why this priority**: Enhances the standard word-level outputs for advanced downstream applications.

**Independent Test**: Run `align()` with `return_char_alignments=True` and verify that the output contains character-level start and end coordinates.

**Acceptance Scenarios**:
1. **Given** a transcription task,
   **When** calling `align` with `return_char_alignments=True`,
   **Then** each word segment contains a `chars` list detailing individual letter timestamps.

---

### User Story 3 - Handling Unalignable and OOV Words (Priority: P2)
As a user, I want the alignment engine to successfully process numbers, symbols, and words that are not in the acoustic model's vocabulary, so that the entire transcription receives timestamps.

**Why this priority**: Prevents alignment failures and omissions of segments containing digits or punctuation.

**Independent Test**: Covered in [test_word_timestamp_interpolation.py](file:///home/andrey/Workspace/whisperX/tests/test_word_timestamp_interpolation.py) (e.g. `test_unknown_word_gets_timestamps`).

**Acceptance Scenarios**:
1. **Given** a transcript containing numbers or out-of-vocabulary words (e.g. "cost 43 dollars"),
   **When** running alignment,
   **Then** timestamps are assigned to unalignable words via wildcard masking or time interpolation.

---

### Edge Cases
- **Silence / No Speech:** When a segment contains only silence, the alignment grid trellis will fail to resolve. The engine must fall back to assigning segment boundary timestamps or raise clear warnings without crashing.
- **Transcribe/Align Language Mismatch:** When the alignment model loaded does not match the actual spoken language, alignment scores will drop significantly. The engine must raise errors if the default model for a language is unavailable.

## Requirements

### Functional Requirements
- **FR-001 (Model Mapping):** The engine MUST support automatic mapping of 37+ languages to pretrained Wav2Vec2/VoxPopuli models on Torchaudio and Hugging Face.
- **FR-002 (CTC Trellis Grid):** Alignment MUST be computed using CTC emission probability trellis grids, aligning characters to audio frames via dynamic programming backtracking.
- **FR-003 (Interpolation Methods):** For non-aligned words, the engine MUST support multiple interpolation modes: `nearest`, `linear`, and `ignore` (merging into neighbors).
- **FR-004 (Device Compatibility):** Alignment models MUST be loadable on both CPU and GPU (`cuda`).
- **FR-005 (Word Tokenization):** The engine MUST clean punctuation, handle language-specific conjunction rules (e.g. utilizing `conjunctions.py`), and split sentences correctly, maintaining mapping to original text.

### Key Entities
- **AlignedTranscriptionResult:** The top-level output dictionary containing aligned word segments.
- **SingleWordSegment:** TypedDict schema containing `word`, `start`, `end`, and `score`.
- **Trellis:** The dynamic programming grid storing paths of frame-to-character emissions.

## Success Criteria

### Measurable Outcomes
- **SC-001:** Timestamps must be accurate to within a single Wav2Vec2 frame duration (~20ms resolution).
- **SC-002:** At least 90% of words in standard audio segments must receive valid start/end timestamps (no NaN values) under the `nearest` or `linear` interpolation methods.
- **SC-003:** Execution time for aligning a 30-minute audio track must remain under 1 minute on a modern CPU/GPU.

## Assumptions
- The host system has Ffmpeg installed to handle decoding raw audio files.
- Hugging Face Hub or local cache is accessible to retrieve model weights.
