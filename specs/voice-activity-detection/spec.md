# Feature Specification: Voice Activity Detection (VAD)

**Status**: migrated

**Created**: 2026-07-12

**Feature Branch**: `main`

## User Scenarios & Testing

### User Story 1 - Speech-Activity Filtering & Segment Merging (Priority: P1)
As a user transcribing long audio files with periods of silence, I want the system to detect and filter out non-speech segments before ASR processing, so that Whisper is not forced to transcribe silence, minimizing hallucinations and speed overhead.

**Why this priority**: Crucial preprocessing phase to prevent Whisper from looping/hallucinating on noise and silence.

**Independent Test**: Currently not automated. Test via CLI by passing a silent audio track and checking that no transcription segments are generated.

**Acceptance Scenarios**:
1. **Given** an audio track containing 10 seconds of speech followed by 50 seconds of silence,
   **When** running transcription with VAD activated,
   **Then** only the active 10-second speech segment is sent to the Whisper decoder.

---

### User Story 2 - Multiple VAD Engines Option (Priority: P2)
As a developer, I want to choose between different Voice Activity Detection engines (Silero VAD or PyAnnote VAD), so that I can optimize for speed (using Silero's ONNX runner) or diarization accuracy (using PyAnnote).

**Why this priority**: Allows flexibility depending on execution requirements, environment, and models.

**Independent Test**: Currently not automated. Run CLI with `--vad_method silero` and `--vad_method pyannote` and verify execution succeeds in both modes.

**Acceptance Scenarios**:
1. **Given** a WAV audio file,
   **When** calling the transcription pipeline with `--vad_method silero`,
   **Then** the Silero ONNX model is initialized and yields speech segments.

---

### Edge Cases
- **Continuous Speech Exceeding Chunk Size:** If a speaker talks continuously for more than 30 seconds without pausing, VAD chunking must clip the segment at 30 seconds to prevent Whisper context window overflow.
- **Onset/Offset Tuning:** For faint or quiet speech, default VAD thresholds might miss words. Users must be able to specify custom `--vad_onset` and `--vad_offset` values.

## Requirements

### Functional Requirements
- **FR-001 (Silero Integration):** The engine MUST load and run Silero VAD weights using ONNX runtime, processing audio inputs into probability states.
- **FR-002 (PyAnnote VAD Integration):** The engine MUST support PyAnnote binarized segmentation pipelines using PyTorch.
- **FR-003 (Chunk Merging):** The VAD base class MUST implement `merge_chunks()` to combine short audio segment timings into batches up to `chunk_size` (default 30 seconds) for batched Whisper inference.
- **FR-004 (CLI/API configurations):** Parameters MUST include `--vad_method`, `--vad_onset`, `--vad_offset`, and `--chunk_size`.

### Key Entities
- **Vad:** Abstract base class defining initialization, audio preprocessing, and chunk merging.
- **Silero / Pyannote:** Specialized VAD implementations.
- **MergedSegment:** Dictionary containing `start`, `end`, and lists of original child segments.

## Success Criteria

### Measurable Outcomes
- **SC-001:** Transcription hallucination rates on silent audio must drop to near-zero when VAD is activated.
- **SC-002:** Chunk sizes returned by `merge_chunks()` must never exceed the user-defined `chunk_size` limit.

## Assumptions
- The host system has appropriate ONNX runtime libraries available to load the `.onnx` Silero weights.
- Audio sample rates are standard 16kHz for proper model compatibility.
