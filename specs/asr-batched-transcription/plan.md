# Implementation Plan: ASR & Batched Transcription

**Branch**: `main` | **Date**: 2026-07-12 | **Spec**: [spec.md](file:///home/andrey/Workspace/whisperX/specs/asr-batched-transcription/spec.md)

## Summary
Technical design mapping of the batched Whisper ASR transcription engine. By utilizing `faster-whisper` and subclassing its generation functions, whisperX enables batch execution on GPU by chunking continuous audio using VAD timelines.

## Technical Context
- **Language/Version**: Python 3.10+
- **Primary Dependencies**: `ctranslate2`, `faster-whisper`, `torch`, `transformers`
- **Testing**: Not automated (needs unit test creation)
- **Target Platform**: Cross-platform (CPU and CUDA GPUs)

## Constitution Check
- **Naming Conventions Match**: Naming follows `snake_case` in `asr.py` and `transcribe.py`. ✅ Pass.
- **Pipeline Isolation Maintained**: `asr.py` focuses purely on loading model checkpoints and performing speech-to-text batch evaluations. Orchestration is separate. ✅ Pass.

## Project Structure
The files implementing this feature:

```text
whisperx/
├── asr.py             # WhisperModel subclass, FasterWhisperPipeline wrapper
├── transcribe.py      # Entry point orchestration for CLI execution
└── audio.py           # Audio loader, mel filter bank extractor
```

## Implementation Details

### Phase 1: Model Subclassing & Batch Generation
- `whisperx.asr.WhisperModel` inherits from `faster_whisper.WhisperModel`.
- Overrides `generate_segment_batched()` which collects audio features and maps them to a single batch inference call using `self.model.generate()` from `ctranslate2`.
- Handles prompt tokens, hotwords, and decoding parameter configurations.

### Phase 2: Pipeline Orchestration
- `FasterWhisperPipeline` implements `transcribe(audio, batch_size, chunk_size, print_progress)`.
- It processes audio waveforms, computes mel spectrograms, extracts active VAD frames, pads them to 30-second sequences, and triggers batched forward passes.
- Aggregates decoded segments back into a sequential transcript.

### Phase 3: CLI Command Coordination
- `transcribe_task` in `whisperx/transcribe.py` extracts all command arguments (`--model`, `--batch_size`, `--compute_type`, etc.).
- Instantiates the pipeline, executes transcription, calls alignment/diarization modules if requested, clears PyTorch CUDA cache, and flushes outputs via the configured result writer.
