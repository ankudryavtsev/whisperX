# Implementation Plan: Voice Activity Detection (VAD)

**Branch**: `main` | **Date**: 2026-07-12 | **Spec**: [spec.md](file:///home/andrey/Workspace/whisperX/specs/voice-activity-detection/spec.md)

## Summary
Technical design mapping of the Voice Activity Detection wrapper subpackage inside whisperX. The VAD layer parses audio timelines into voice-containing ranges and groups them into batches up to 30 seconds for optimal batch transcription.

## Technical Context
- **Language/Version**: Python 3.10+
- **Primary Dependencies**: `onnxruntime`, `pyannote.audio`, `torch`, `pandas`
- **Testing**: Not automated (needs unit test creation)
- **Target Platform**: Cross-platform (CPU and CUDA GPUs)

## Constitution Check
- **Naming Conventions Match**: Modules are located inside `whisperx/vads/` subpackage. Classes are `PascalCase` (`Silero`, `Pyannote`, `Vad`), and utility functions are `snake_case`. ✅ Pass.
- **Pipeline Isolation Maintained**: VAD modules focus exclusively on computing activation bounds from raw audio. They do not depend on ASR models or diarization logic. ✅ Pass.

## Project Structure
The files implementing this feature:

```text
whisperx/
├── vads/
│   ├── __init__.py    # VAD model factory (load_vad_model)
│   ├── vad.py         # Base Vad class with merge_chunks method
│   ├── silero.py      # Silero VAD ONNX wrapper implementation
│   └── pyannote.py    # PyAnnote segmentation wrapper implementation
└── transcribe.py      # Integrates VAD segments slicing into CLI workflow
```

## Implementation Details

### Phase 1: Chunk Merging Algorithm
- VAD models return a list of short speech segments. Passing them individually to Whisper is highly inefficient.
- `Vad.merge_chunks()` processes these segments.
- Iterates over segments and groups them together into a single merged segment.
- Triggers a boundary split when the cumulative duration from the starting timestamp exceeds the user-configured `chunk_size` (usually 30s).
- Returns a list of dictionaries with `start`, `end`, and constituent segment indexes.

### Phase 2: Silero VAD Wrapper
- `Silero` inherits from `Vad` base class.
- Resolves the path to the stored `silero_vad.onnx` weight file.
- Uses `onnxruntime.InferenceSession` to execute predictions.
- Parses probability outputs above `vad_onset` into speech start/end segments.

### Phase 3: PyAnnote VAD Wrapper
- `Pyannote` inherits from `Vad` base class.
- Loads pyannote binarization and voice activity segmentation models.
- Applies standard onset/offset thresholds to extract speech segment annotations.
