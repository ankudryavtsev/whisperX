# Tasks: Speaker Diarization

**Input**: [spec.md](file:///home/andrey/Workspace/whisperX/specs/speaker-diarization/spec.md), [plan.md](file:///home/andrey/Workspace/whisperX/specs/speaker-diarization/plan.md)

## Phase 1: Setup & Dependencies
- [x] T001 Define dependencies `pyannote-audio`, `pandas` in `pyproject.toml`
- [x] T002 Implement token verification hooks to handle gated model downloading permissions

## Phase 2: Interval Tree Infrastructure
- [x] T003 Implement `IntervalTree` class with sorted starts and ends in `whisperx/diarize.py`
- [x] T004 Implement `IntervalTree.query()` using binary search (`np.searchsorted`)
- [x] T005 Implement `IntervalTree.find_nearest()` fallback for non-overlapping silence boundaries

## Phase 3: Diarization Engine
- [x] T006 Implement `DiarizationPipeline` class wrapping `pyannote.audio.Pipeline`
- [x] T007 Implement speaker clustering constraints option (`min_speakers`, `max_speakers`)
- [x] T008 Implement `assign_word_speakers` function calculating overlap durations and resolving final speaker labels

## Phase 4: CLI Orchestration
- [x] T009 Expose CLI options (`--diarize`, `--min_speakers`, `--max_speakers`, `--diarize_model`, `--speaker_embeddings`) in `whisperx/__main__.py`
- [x] T010 Integrate diarization assignment execution within transcription orchestration loop in `whisperx/transcribe.py`

---

## Gaps Identified

### Gap 1: No Automated Test Coverage
- **Problem:** No unit or integration tests exist for `diarize.py`. The `IntervalTree` binary search boundaries and nearest-speaker logic are untested, introducing regression risks if the matching algorithm is refactored.
- **Remedy:** Create `tests/test_diarize.py` to unit-test the `IntervalTree` class using mock intervals, verifying:
  1. Accurate binary search candidate filtering.
  2. Overlapping calculation logic.
  3. `find_nearest` midpoint calculations.

### Gap 2: Gated Hugging Face Token Validation
- **Problem:** Missing token or network failures during gated model loading throw generic PyAnnote/Hugging Face exceptions, which might confuse CLI users.
- **Remedy:** Implement friendly error wrappers verifying token existence and raising clean instructions on how to accept model licenses.
