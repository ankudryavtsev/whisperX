# Tasks: Python 3.14 Support

**Input**: [spec.md](file:///home/andrey/Workspace/whisperX/specs/001-python-3.14-support/spec.md), [plan.md](file:///home/andrey/Workspace/whisperX/specs/001-python-3.14-support/plan.md)

## Phase 1: Setup
- [x] T001 Update Python compatibility version target in pyproject.toml

## Phase 2: Foundational
- [x] T002 [P] Update CI testing matrix to include Python 3.14 in .github/workflows/tests.yml
- [x] T003 [P] Update CI compatibility matrix to include Python 3.14 in .github/workflows/python-compatibility.yml

## Phase 3: User Story 1 - Running whisperX under Python 3.14 (Priority: P1)
**Goal**: Install and execute whisperX CLI and tests successfully on a Python 3.14 local environment.
**Independent Test**: Spin up a Python 3.14 virtualenv, install, and run pytest.

- [x] T004 [US1] Create local Python 3.14 virtualenv to verify dependency mapping
- [x] T005 [US1] Install package in dev mode using uv pip install --extra dev -e .
- [x] T006 [US1] Execute test suite on Python 3.14 using uv run --extra dev pytest

## Phase 4: User Story 2 - CI Compatibility Verification (Priority: P2)
**Goal**: Verify that automated CI workflows run and pass test suites on Python 3.14.
**Independent Test**: Check that remote GitHub Actions run matrices successfully pass the 3.14 job.

- [x] T007 [US2] Push updates to remote branch and trigger GitHub Action workflows
- [ ] T008 [US2] Verify that tests.yml runs successfully and passes on Python 3.14 runner
- [ ] T009 [US2] Verify that python-compatibility.yml passes on Python 3.14 runner

## Phase 5: Polish & Cross-Cutting Concerns
- [x] T010 Verify packaging builds successfully on Python 3.14 using uv build

---

## Dependencies
US1 ➡️ US2 (CI verification requires local package definition and config changes first)

## Parallel Execution Examples
- Foundational configuration updates: T002 and T003 can be executed in parallel.
- CI verification checks: T008 and T009 can be executed in parallel once pushed.

## Implementation Strategy
MVP scope includes Phase 1, Phase 2, and Phase 3 (enabling local execution and tests verification on Python 3.14). Phase 4 will finalize integration in CI.
