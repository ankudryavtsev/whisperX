# Tasks: [FEATURE NAME]

**Input**: Design documents from `/specs/[###-feature-name]/`

**Prerequisites**: plan.md (required), spec.md (required)

**Tests**: Test tasks must use `pytest` and target tests under `tests/`.

## Format: `[ID] [P?] [Story] Description`
- **[P]**: Can run in parallel
- **[Story]**: Story reference (e.g., US1, US2)

## Phase 1: Setup (Shared Infrastructure)
- [ ] T001 Verify project environment (`uv` env is active)
- [ ] T002 Configure new optional dependencies in `pyproject.toml` and run `uv lock` (if needed)

## Phase 2: Foundational (Blocking Prerequisites)
- [ ] T003 [P] Add helper structures or schemas to `whisperx/schema.py` or `whisperx/utils.py`
- [ ] T004 Implement model wrappers or base pipelines in `whisperx/asr.py` or `whisperx/vads/`

## Phase 3: User Story 1 - [Title] (Priority: P1)
**Goal**: [Goal]
**Independent Test**: `pytest tests/test_[name].py`

### Tests
- [ ] T005 [P] [US1] Write test cases in `tests/test_[name].py` (ensure they fail first)

### Implementation
- [ ] T006 [US1] Implement module changes in `whisperx/[file].py`
- [ ] T007 [US1] Expose parameters in `whisperx/__main__.py`
- [ ] T008 [US1] Run tests using `pytest tests/test_[name].py` and verify they pass

## Phase N: Polish & Verification
- [ ] T009 Run full test suite: `pytest`
- [ ] T010 Validate output formats (SRT, VTT, JSON, TXT)
