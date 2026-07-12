# Feature Specification: Speaker Diarization

**Status**: migrated

**Created**: 2026-07-12

**Feature Branch**: `main`

## User Scenarios & Testing

### User Story 1 - Multi-Speaker Transcript Labeling (Priority: P1)
As a user transcribing audio with multiple speakers, I want the output transcript segments and individual words to be labeled with speaker identifiers (e.g. `SPEAKER_01`, `SPEAKER_02`), so that I can easily follow the conversation.

**Why this priority**: Crucial for meeting transcriptions, interviews, and conversational speech datasets.

**Independent Test**: Currently not automated. Test via CLI by passing a multi-speaker audio file with `--diarize` and a Hugging Face token, then verifying that the output JSON or text structures include speaker labels.

**Acceptance Scenarios**:
1. **Given** a valid conversational audio recording with two speakers,
   **When** running transcription with `--diarize` and a valid `--hf_token`,
   **Then** segments in the output transcript are labeled with distinct speaker tags.

---

### User Story 2 - Diarization Speaker Bounds Constraint (Priority: P2)
As a developer, I want to guide the diarization model by setting the minimum and maximum number of speakers expected in the audio, so that speaker separation accuracy is optimized.

**Why this priority**: Improves PyAnnote clustering accuracy when the number of speakers is known in advance.

**Independent Test**: Currently not automated. Test by executing diarization with `--min_speakers 2` and `--max_speakers 2` and checking that exactly two speaker classes are output.

**Acceptance Scenarios**:
1. **Given** an audio track containing exactly two speakers,
   **When** initiating diarization with `--min_speakers 2` and `--max_speakers 2`,
   **Then** the PyAnnote pipeline enforces a 2-speaker partition constraint on the output.

---

### Edge Cases
- **Missing Hugging Face Token:** PyAnnote gated models require authentication. If no token is provided, or the token is invalid, the engine must catch the exception and display a clear user message explaining how to acquire and pass the token.
- **No Overlap Matches:** If a word segment has zero timestamp overlaps with any diarization segments (e.g., in background noise or very short segments), the system must fallback to assigning the speaker of the nearest diarization segment to prevent empty labels.

## Requirements

### Functional Requirements
- **FR-001 (PyAnnote Integration):** The engine MUST load and run PyAnnote-audio speaker diarization pipelines (`pyannote/speaker-diarization-community-1`) using PyTorch on CPU or GPU.
- **FR-002 (Interval Tree Overlap Queries):** The system MUST implement an `IntervalTree` utility based on binary search to quickly query overlapping speaker boundaries for aligned words.
- **FR-003 (Word Speaker Assignment):** The assignment function MUST calculate overlap durations between word timestamps and diarization intervals, assigning the speaker with the maximum overlap.
- **FR-004 (CLI/API parameters):** The engine MUST support parameters: `--diarize`, `--min_speakers`, `--max_speakers`, `--diarize_model`, `--speaker_embeddings`, and `--hf_token`.

### Key Entities
- **DiarizationPipeline:** Class wrapping the PyAnnote model loading and prediction steps.
- **IntervalTree:** Structure sorting diarization intervals for fast binary query overlaps.
- **SingleWordSegment:** Modified schema adding a `speaker` string field.

## Success Criteria

### Measurable Outcomes
- **SC-001:** Binary-search-based `IntervalTree` queries must execute speaker mapping in $O(\log n)$ time, avoiding $O(n^2)$ complexity for large transcripts.
- **SC-002:** At least 95% of word segments with valid alignments must receive a speaker label (either by maximum overlap or nearest neighbor distance).

## Assumptions
- A valid Hugging Face account and signed model access agreement exist for pyannote models.
- The audio segment files can be parsed to extract multi-channel speaker features.
