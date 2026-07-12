# Tasks: Voice Activity Detection (VAD)

**Input**: [spec.md](file:///home/andrey/Workspace/whisperX/specs/voice-activity-detection/spec.md), [plan.md](file:///home/andrey/Workspace/whisperX/specs/voice-activity-detection/plan.md)

## Phase 1: Setup & Dependencies
- [x] T001 Add dependency definitions `onnxruntime`, `pyannote-audio` in `pyproject.toml`
- [x] T002 Gather Silero VAD `.onnx` weights file inside package resources

## Phase 2: VAD Base Interface
- [x] T003 Implement base `Vad` class in `whisperx/vads/vad.py`
- [x] T004 Implement `merge_chunks` logic grouping VAD outputs within `chunk_size` bounds in `whisperx/vads/vad.py`

## Phase 3: Engine Implementations
- [x] T005 Implement `Silero` wrapper using `onnxruntime` inside `whisperx/vads/silero.py`
- [x] T006 Implement `Pyannote` wrapper using PyTorch models inside `whisperx/vads/pyannote.py`
- [x] T007 Implement factory method `load_vad_model` inside `whisperx/vads/__init__.py`

## Phase 4: CLI Integration
- [x] T008 Add parameters `--vad_method`, `--vad_onset`, `--vad_offset`, `--chunk_size` in `whisperx/__main__.py`
- [x] T009 Coordinate loading VAD model and generating merged chunks within CLI loop in `whisperx/transcribe.py`

---

## Gaps Identified

### Gap 1: No Unit/Integration Tests
- **Problem:** No tests cover the `vads` subpackage. The `merge_chunks` algorithm, Silero ONNX runner, and PyAnnote binarization wrappers are completely untested in CI.
- **Remedy:** Implement `tests/test_vad.py` to:
  1. Unit-test `merge_chunks` using synthetic segment boundaries.
  2. Verify that `merge_chunks` never exceeds the maximum chunk duration (e.g. 30 seconds).
  3. Validate that `load_vad_model` instantiates the correct wrappers.

### Gap 2: ONNX Runtime Compatibility Gaps
- **Problem:** ONNX runtime versions can crash on Apple Silicon (macOS) or ARM64 Linux if wheels are mismatched, which is not verified by tests.
- **Remedy:** Setup automated CI steps running a lightweight Silero VAD forward pass on both CPU and GPU runners.
