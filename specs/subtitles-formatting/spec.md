# Feature Specification: Subtitles & Export Formatting

**Status**: migrated

**Created**: 2026-07-12

**Feature Branch**: `main`

## User Scenarios & Testing

### User Story 1 - Multiple Export Formats (Priority: P1)
As a user transcribing audio, I want to save the output transcript in various standardized formats (SRT, VTT, TSV, TXT, Audacity, and JSON), so that I can import them directly into video editors, spreadsheets, or database systems.

**Why this priority**: Crucial for pipeline integration and cross-application subtitle rendering.

**Independent Test**: Currently not automated. Test via CLI by passing `--output_format all` and verifying all 6 output files are written to the target directory.

**Acceptance Scenarios**:
1. **Given** a finished transcription result,
   **When** running execution with `--output_format all`,
   **Then** the output directory contains `.txt`, `.srt`, `.vtt`, `.tsv`, `.json`, and `.aud` files.

---

### User Story 2 - Advanced Subtitle Splitting (Priority: P2)
As a caption editor, I want long transcription segments to be automatically split into shorter lines matching standard caption limits (e.g. 45 characters max line length), splitting on natural boundaries like commas or conjunctions, so that the captions are highly readable.

**Why this priority**: Ensures subtitle readability and aesthetic layouts on screen.

**Independent Test**: Currently not automated. Run splitting on a long transcript segment and verify lines do not exceed the max character threshold.

**Acceptance Scenarios**:
1. **Given** a long segment containing conjunctions and commas,
   **When** running the advanced splitting algorithm,
   **Then** the segment is divided into smaller subtitle intervals split at commas or conjunctions, with each line remaining under 45 characters.

---

### Edge Cases
- **Missing Timestamps Estimation:** If words are not aligned (returning no timestamps), the processor must estimate start/end bounds dynamically based on adjacent word timings and average character count durations to prevent formatting errors.
- **Word Highlighting:** When `--highlight_words` is enabled, the SRT/VTT outputs must inject HTML style tags (`<u>word</u>` or formatting markers) dynamically tracking the highlight timeline of each spoken word.

## Requirements

### Functional Requirements
- **FR-001 (Subtitle Exporters):** The system MUST support writing output files in 6 formats: TXT, SRT, VTT, TSV, JSON, and Audacity tags.
- **FR-002 (Word Estimation):** The system MUST implement `estimate_timestamp_for_word()` to calculate start and end times for unaligned words using neighboring word boundaries.
- **FR-003 (Line Width and Count Restrictions):** The subtitle processor MUST enforce configurable maximum line widths and split points based on language script complexity (e.g. Chinese/Japanese scripts trigger shorter line breaks).
- **FR-004 (Word Highlighting):** The system MUST support VTT/SRT styling tags to highlight words in synchronization with audio playback.

### Key Entities
- **SubtitlesProcessor:** Class wrapping splitting points detection and timeline adjustments.
- **ResultWriter / SubtitlesWriter:** Abstract and specialized classes handling file output formatting.
- **SubtitlesLine:** Schema representing a subtitle line with text, start, and end parameters.

## Success Criteria

### Measurable Outcomes
- **SC-001:** Timestamps formatted into SRT and VTT files must match exact standard structures (`HH:MM:SS,mmm` for SRT, `HH:MM:SS.mmm` for VTT).
- **SC-002:** Split subtitles must strictly conform to the user-specified max character length boundaries.

## Assumptions
- Output folders have write access permissions.
- Punctuation rules and conjunction lists are mapped correctly for the transcription language.
