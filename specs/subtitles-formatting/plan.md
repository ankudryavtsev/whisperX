# Implementation Plan: Subtitles & Export Formatting

**Branch**: `main` | **Date**: 2026-07-12 | **Spec**: [spec.md](file:///home/andrey/Workspace/whisperX/specs/subtitles-formatting/spec.md)

## Summary
Technical design mapping of the subtitle formatting and output writing classes in whisperX. The processor handles segment line-breaking and word-level timing estimation, while the writer classes format text to standardized specifications.

## Technical Context
- **Language/Version**: Python 3.10+
- **Primary Dependencies**: `pandas`, `numpy`, `torch`
- **Testing**: Not automated (needs unit test creation)
- **Target Platform**: Cross-platform (CPU and CUDA GPUs)

## Constitution Check
- **Naming Conventions Match**: Naming follows `snake_case` for files and variables with exceptions like `SubtitlesProcessor.py` (which is PascalCase, documented exception). Classes are PascalCase (`WriteSRT`, `WriteVTT`, `SubtitlesProcessor`). ✅ Pass.
- **Pipeline Isolation Maintained**: Formatting classes ingest structured data dictionaries and execute write routines, remaining isolated from model execution code. ✅ Pass.

## Project Structure
The files implementing this feature:

```text
whisperx/
├── SubtitlesProcessor.py    # SubtitlesProcessor, timestamp formatter, estimation logic
└── utils.py                 # ResultWriter, WriteSRT, WriteVTT, WriteJSON, WriteTXT, get_writer
```

## Implementation Details

### Phase 1: Timestamp Formatting & Estimation
- Format timestamps using `format_timestamp(seconds, is_vtt)` using mathematical divisions to compute hours, minutes, seconds, and milliseconds.
- Implement `estimate_timestamp_for_word` using adjacent boundaries:
  - If a previous word has an end time, assign the current word start to it.
  - If a next word has a start time, assign the current word end to it.
  - Otherwise, fallback to character-count-based duration estimates.

### Phase 2: Advanced Segment Splitting
- Detect split points in `determine_advanced_split_points()` using:
  - Punctuation (commas, periods, exclamation points).
  - Language-specific conjunctions (loaded from `whisperx/conjunctions.py`).
  - Maximum character lengths (`max_line_length` defaults to 45).
- Re-generate subtitle segments and divide the timeline proportionally.

### Phase 3: Export Exporters (ResultWriter Factory)
- `get_writer(format, output_dir)` acts as a factory returning specialized subclasses:
  - `WriteTXT`: plain text transcription output.
  - `WriteSRT`: SubRip subtitle file with highlight tag formatting support.
  - `WriteVTT`: WebVTT subtitle format with native WebVTT timestamp tags.
  - `WriteTSV`: Tab-separated spreadsheet output.
  - `WriteJSON`: complete output including word segments, characters, and speaker embeddings.
  - `WriteAudacity`: Audacity label track format.
