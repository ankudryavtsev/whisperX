# Feature Specification: Python 3.14 Support

**Feature Branch**: `001-python-3.14-support`

**Created**: 2026-07-12

**Status**: Draft

**Input**: User description: "Let's add Python 3.14 support"

## User Scenarios & Testing

### User Story 1 - Running whisperX under Python 3.14 (Priority: P1)
As a developer or user running the latest Python runtime, I want to be able to install and run whisperX CLI and Python API on Python 3.14, so that I can keep my environment updated without compatibility blocks.

**Why this priority**: Essential to prevent deprecation issues and allow execution on modern runtime environments.

**Independent Test**: Install whisperX in a Python 3.14 virtual environment and run the full test suite (`pytest`) to verify execution.

**Acceptance Scenarios**:
1. **Given** a Python 3.14 interpreter environment,
   **When** running `pip install .` or installing via `uv`,
   **Then** whisperX installs without dependency resolution failures.
2. **Given** a Python 3.14 virtual environment,
   **When** running `uv run --extra dev pytest`,
   **Then** all unit and integration tests pass without syntax or import errors.

---

### User Story 2 - CI Compatibility Verification (Priority: P2)
As a maintainer of whisperX, I want our GitHub Action workflows to verify Python 3.14 compatibility automatically on pull requests and commits, so that we don't accidentally introduce regressions.

**Why this priority**: Prevents future regressions and verifies compatibility across different operating systems in CI.

**Independent Test**: Verify that the GitHub Actions run matrix includes `3.14` and passes successfully.

**Acceptance Scenarios**:
1. **Given** a pull request to the main branch,
   **When** GitHub Action workflows trigger,
   **Then** the test suite executes successfully on Python 3.14.

---

### Edge Cases
- **Acoustic and Deep Learning dependencies (PyTorch, PyAnnote):** Python 3.14 is very new, so some pre-compiled binary wheels may not be available on PyPI. We might need to compile from source or allow pre-release versions.
- **Removed C APIs in Python 3.14:** Third-party libraries (e.g. `ctranslate2` or `numpy`) might use deprecated C APIs removed in 3.14. If so, we must identify minimum compatible versions of these dependencies.

## CLI & Python API Options
No new CLI options are added, but we modify the Python version constraint in `pyproject.toml`:
- Modify `requires-python` from `">=3.10, <3.14"` to `">=3.10, <3.15"`.

## Model & Pipeline Impact
There is no direct impact on transcription or alignment math. However, the runtime performance under the new interpreter and GC behavior should be monitored for latency regressions.

- [ ] **ASR / Transcription**: (`whisperx/transcribe.py`, `whisperx/asr.py`)
- [ ] **Forced Alignment**: (`whisperx/alignment.py`)
- [ ] **Speaker Diarization**: (`whisperx/diarize.py`)
- [ ] **VAD (Voice Activity Detection)**: (`whisperx/vads/`)
- [ ] **Output Formatting / Subtitles**: (`whisperx/SubtitlesProcessor.py`, `whisperx/utils.py`)

*Note: All pipeline steps are affected as they run under the Python 3.14 interpreter.*

## Success Criteria

### Measurable Outcomes
- **SC-001:** WhisperX installs successfully on Python 3.14 using `pyproject.toml` and standard package managers.
- **SC-002:** 100% of the unit test suite (`pytest`) passes under a Python 3.14 execution environment.

## Assumptions
- Downstream binary wheels (like `numpy`, `pandas`, `scipy`, `onnxruntime`, `torch`) have compatible releases or source distributions for Python 3.14.
- `ctranslate2` compiles successfully or has compatible packages for Python 3.14.
