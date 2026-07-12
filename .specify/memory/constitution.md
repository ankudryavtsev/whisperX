# whisperX Constitution

## Core Principles

### I. Python-First Library & CLI
whisperX is primarily a Python library exposing high-performance transcription pipelines, which is also accessible as a CLI tool. All code must be designed as reusable Python components first, with CLI arguments in `whisperx/__main__.py` acting as a thin interface layer.

### II. Pipeline Component Isolation
The transcription workflow is structured as a pipeline (VAD -> Whisper ASR Transcription -> Forced Alignment -> Diarization). Every pipeline module (`asr`, `alignment`, `diarize`, `vads`) must maintain clear separation of concerns and avoid cyclic or direct cross-module coupling. The coordination of these steps must be handled exclusively by `whisperx/transcribe.py`.

### III. GPU-Aware Design and Resource Management
Models (Whisper, wav2vec2, PyAnnote) must be loaded dynamically, supporting options for device mapping (`cuda`, `cpu`), device indices, and compute precision (`float16`, `float32`, `int8`). Code must handle device transfer and memory cleanup gracefully to prevent CUDA out-of-memory (OOM) errors.

### IV. Standardized Testing with pytest
Any new feature or modification must include test coverage in the `tests/` directory. All test files must be prefixed with `test_` (e.g., `tests/test_feature.py`) and be executed via `pytest`.

## Code Boundaries
- **Core Library**: All core logic lives inside the `whisperx/` package.
  - `whisperx/vads/`: Subpackage containing Voice Activity Detection wrappers.
  - `whisperx/assets/`: Static and binary assets (e.g. mel filters).
- **CLI Interface**: Entrypoint resides in `whisperx/__main__.py`.
- **Test Suite**: All test code resides in `tests/`. No testing code should be shipped in the distribution package.

## Naming Conventions
- **Files**: All python files must use `snake_case` (e.g., `log_utils.py`), with the exception of the legacy `SubtitlesProcessor.py`.
- **Classes**: `PascalCase` (e.g. `WhisperModel`, `SubtitlesWriter`).
- **Functions & Variables**: `snake_case` (e.g. `load_align_model()`, `device_index`).
- **Git Branches**: Use standard prefix conventions like `feat/*`, `fix/*`, `chore/*`, `ci/*`.
- **Commits**: Follow Conventional Commits format (e.g., `feat: add alignment language`, `fix(cli): correct device mapping`).

## Quality Gates
- **Tests**: All tests in `tests/` must pass before merging.
- **Dependency Management**: Updates to project metadata or dependencies must be made in `pyproject.toml` and locked using `uv lock` (updating `uv.lock`).

**Version**: 1.0.0 | **Ratified**: 2026-07-12 | **Last Amended**: 2026-07-12
