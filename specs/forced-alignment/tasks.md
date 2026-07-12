# Tasks: Forced Phoneme Alignment

**Input**: [spec.md](file:///home/andrey/Workspace/whisperX/specs/forced-alignment/spec.md), [plan.md](file:///home/andrey/Workspace/whisperX/specs/forced-alignment/plan.md)

## Phase 1: Setup & Dependencies
- [x] T001 Define dependencies (`torchaudio`, `transformers`, `torch`) in `pyproject.toml`
- [x] T002 Configure `nltk` for punctuation/conjunction preprocessing

## Phase 2: Schema & Layout Definitions
- [x] T003 Implement `SingleWordSegment`, `SingleAlignedSegment` TypedDicts in `whisperx/schema.py`
- [x] T004 Define language conjunctions rules inside `whisperx/conjunctions.py`

## Phase 3: Alignment Logic
- [x] T005 Implement `load_align_model` to load Torchaudio/Hugging Face models in `whisperx/alignment.py`
- [x] T006 Implement dynamic programming trellis calculation and backtracking pathfinding in `whisperx/alignment.py`
- [x] T007 Integrate `interpolate_nans` to resolve timestamps for unaligned/wildcard words in `whisperx/utils.py`
- [x] T008 Add CLI arguments interface support (`--align_model`, `--interpolate_method`, `--no_align`, `--return_char_alignments`) in `whisperx/__main__.py`

## Phase 4: Verification & Testing
- [x] T009 Create mock emission helper utilities in `tests/test_word_timestamp_interpolation.py`
- [x] T010 Implement end-to-end tests for known and unknown character sequences, ensuring timestamps match targets
- [x] T011 Verify nearest/linear/ignore interpolation methods in `tests/test_word_timestamp_interpolation.py`
- [x] T012 Run test validation locally via `pytest tests/test_word_timestamp_interpolation.py`

---

## Gaps Identified

### Gap 1: No Automated Model Loading tests
- **Problem:** `load_align_model` is untested in the current unit test suite because model downloads are slow and require external network access.
- **Remedy:** Consider adding a test suite that runs integration tests with small cached models, or mocks Hugging Face downloads to verify parser mappings.

### Gap 2: GPU/CUDA Pipeline Verification
- **Problem:** Test suite executes only on CPU (`device="cpu"`). Device mapping to `cuda` is untested by automation.
- **Remedy:** Add conditional GPU checks in tests (`torch.cuda.is_available()`) to execute tests on CUDA devices when run on GPU-enabled runners.
