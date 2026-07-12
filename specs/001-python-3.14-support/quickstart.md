# Quickstart: Python 3.14 Verification Guide

This guide details how to verify that whisperX installs and executes correctly on a Python 3.14 environment.

## Prerequisites
- A system with Python 3.14 installed (check using `python3.14 --version`).
- `uv` or `pip` installed.

## Setup Instructions

### 1. Initialize Virtual Environment with Python 3.14
Create and activate a new virtual environment using Python 3.14:
```bash
uv venv --python 3.14
source .venv/bin/activate
```

### 2. Install whisperX with Dev Dependencies
Install whisperX along with the `dev` extra group:
```bash
uv pip install --extra dev -e .
```

## Validation Scenarios

### Scenario 1: Run Unit Tests
Run the test suite under Python 3.14 to verify core transcription and forced alignment logic:
```bash
uv run --extra dev pytest
```
**Expected Outcome:**
All 12 test assertions in `tests/test_word_timestamp_interpolation.py` collect and pass successfully.

### Scenario 2: CLI Smoke Test
Run the CLI version query to verify packaging information loads correctly:
```bash
whisperx --version
```
**Expected Outcome:**
Displays whisperx package version without throwing import or metadata exceptions.
