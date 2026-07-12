# Research: Python 3.14 Support

This research documents technical feasibility, dependency support, and CI matrix changes required to add Python 3.14 support to whisperX.

## Technical Decisions

### Decision 1: Upgrade Python version constraint in `pyproject.toml`
* **Decision:** Update the line `requires-python = ">=3.10, <3.14"` to `requires-python = ">=3.10, <3.15"`.
* **Rationale:** Allows pip and uv package managers to install whisperX on Python 3.14 environments.
* **Alternatives Considered:** Specifying no upper bound (e.g. `requires-python = ">=3.10"`). Rejected because future major Python versions (e.g., 3.15+) might introduce breaking C-API changes that crash compiled dependencies (like PyTorch or ctranslate2).

### Decision 2: Update CI test matrix
* **Decision:** Add `3.14` to the Python version list in `.github/workflows/tests.yml` and `.github/workflows/python-compatibility.yml`.
* **Rationale:** Ensures that pull requests are automatically tested against Python 3.14.
* **Alternatives Considered:** Only testing Python 3.14 manually before release. Rejected because manual checks are error-prone and fail to prevent regressions in future commits.

## Dependencies Compatibility Analysis
* **PyTorch & Torchaudio:** PyTorch rolling releases add Python 3.14 compatibility via binary wheels or source compilation.
* **numpy:** NumPy has native compatibility with Python 3.14 in its 2.x releases (whisperX uses `numpy>=2.1.0`).
* **ctranslate2:** Requires compilation from source if binary wheels are missing on PyPI for 3.14. The installation falls back to source build automatically under `uv` or modern `pip` if build-dependencies (like CMake) are set.
