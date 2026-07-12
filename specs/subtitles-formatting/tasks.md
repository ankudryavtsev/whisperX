# Tasks: Subtitles & Export Formatting

**Input**: [spec.md](file:///home/andrey/Workspace/whisperX/specs/subtitles-formatting/spec.md), [plan.md](file:///home/andrey/Workspace/whisperX/specs/subtitles-formatting/plan.md)

## Phase 1: Subtitle Formats & Estimation
- [x] T001 Implement mathematical `format_timestamp` converting seconds to SRT/VTT time blocks
- [x] T002 Implement word-level timestamp fallback interpolation `estimate_timestamp_for_word`

## Phase 2: Advanced Splitting Engine
- [x] T003 Implement `SubtitlesProcessor` class handling max line and char constraints
- [x] T004 Implement `determine_advanced_split_points` splitting lines on commas and conjunction boundaries
- [x] T005 Implement `generate_subtitles_from_split_points` reconstructing new subtitle structures

## Phase 3: Export Exporters
- [x] T006 Implement base class `ResultWriter` in `whisperx/utils.py`
- [x] T007 Implement formatting subclasses `WriteTXT`, `WriteTSV`, `WriteAudacity`, `WriteJSON`
- [x] T008 Implement subtitle subclasses `WriteSRT` and `WriteVTT` with native timing wrappers
- [x] T009 Implement factory helper `get_writer` to dynamically resolve output writers based on CLI inputs

---

## Gaps Identified

### Gap 1: No Unit/Integration Tests
- **Problem:** Neither `SubtitlesProcessor` (word estimation math, segment split boundaries) nor the 6 exporter classes in `utils.py` are covered by automated unit tests.
- **Remedy:** Create `tests/test_subtitles.py` to:
  1. Unit-test `estimate_timestamp_for_word` with partial word timings.
  2. Verify advanced splitting cuts segments exactly on punctuation/conjunctions without leaving empty arrays.
  3. Validate exporter writers by using mock outputs and checking that standard files (VTT, SRT, JSON) are generated correctly.

### Gap 2: Explicit File Encoding
- **Problem:** File writer outputs open file streams without specifying `encoding="utf-8"`, which could fail with UnicodeDecodeError/UnicodeEncodeError on Windows systems running with non-UTF-8 local configurations.
- **Remedy:** Refactor output file streams in `utils.py` to open files explicitly with `encoding="utf-8"`.
