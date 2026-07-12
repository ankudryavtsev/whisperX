# Tasks: ASR & Batched Transcription

**Input**: [spec.md](file:///home/andrey/Workspace/whisperX/specs/asr-batched-transcription/spec.md), [plan.md](file:///home/andrey/Workspace/whisperX/specs/asr-batched-transcription/plan.md)

## Phase 1: Setup & Dependencies
- [x] T001 Define dependencies `ctranslate2`, `faster-whisper`, `torch` in `pyproject.toml`
- [x] T002 Implement ffmpeg audio-loading logic inside `whisperx/audio.py`

## Phase 2: Core Batched Generation
- [x] T003 Implement `WhisperModel` subclass in `whisperx/asr.py`
- [x] T004 Implement `generate_segment_batched` handling token padding, generation parameter overrides, and ctranslate2 generation inputs
- [x] T005 Implement `find_numeral_symbol_tokens` tokenizer utility to prevent numeral-alignment failure edge cases

## Phase 3: Pipeline Integration
- [x] T006 Implement `FasterWhisperPipeline` class to manage batching sequences and loading models
- [x] T007 Add feature/model caching hooks in pipeline setup
- [x] T008 Implement `transcribe_task` CLI orchestration in `whisperx/transcribe.py`
- [x] T009 Integrate garbage collection (`gc.collect()`, `torch.cuda.empty_cache()`) to minimize GPU OOM risk during multi-model pipeline execution

---

## Gaps Identified

### Gap 1: Complete Lack of Unit/Integration Tests
- **Problem:** There are absolutely no unit tests covering `asr.py` or `transcribe.py` in the entire codebase. Features like batched inference, language auto-detection, translation task forwarding, and CLI coordination are completely untested.
- **Remedy:** Create a new test suite (e.g. `tests/test_asr.py` and `tests/test_transcribe.py`) using small dummy audio files or mocked neural weights to verify:
  1. `load_model()` initialization.
  2. Batched generation returns expected token lists.
  3. CLI parameters parse correctly without throwing unexpected errors.

### Gap 2: GPU Memory & Concurrency Safety
- **Problem:** Multiple CUDA settings (e.g., `device_index`, `compute_type`, multi-GPU token emission) are handled dynamically, but no regression tests verify their thread-safety or check if memory leaks happen across large audio batches.
- **Remedy:** Implement stability tests monitoring peak VRAM allocation under different batch sizes and compute settings using mock inputs.
