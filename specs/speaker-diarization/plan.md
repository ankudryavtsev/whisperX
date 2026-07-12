# Implementation Plan: Speaker Diarization

**Branch**: `main` | **Date**: 2026-07-12 | **Spec**: [spec.md](file:///home/andrey/Workspace/whisperX/specs/speaker-diarization/spec.md)

## Summary
Technical design mapping of the PyAnnote-audio speaker diarization integration inside whisperX. The system executes speaker segmentation on raw audio waveforms, then intersects these segments with word boundaries using an interval tree.

## Technical Context
- **Language/Version**: Python 3.10+
- **Primary Dependencies**: `pyannote.audio`, `pandas`, `numpy`, `torch`
- **Testing**: Not automated (needs unit test creation)
- **Target Platform**: Cross-platform (CPU and CUDA GPUs)

## Constitution Check
- **Naming Conventions Match**: Naming follows `snake_case` in `diarize.py` (e.g. `assign_word_speakers()`). ✅ Pass.
- **Pipeline Isolation Maintained**: Diarization logic is encapsulated within `DiarizationPipeline` and `IntervalTree`. It receives transcription results and returns annotated lists, maintaining clear pipeline separation. ✅ Pass.

## Project Structure
The files implementing this feature:

```text
whisperx/
├── diarize.py         # IntervalTree, DiarizationPipeline, assign_word_speakers
├── transcribe.py      # Entry point orchestrating diarization execution
└── schema.py          # Declares speaker labels schema formats
```

## Implementation Details

### Phase 1: Interval Tree for Rapid Overlaps
- Due to the large number of word and speaker segments in long-form audio, a simple nested linear search ($O(n \times m)$) creates a performance bottleneck.
- Implement `IntervalTree` inside `whisperx/diarize.py` using NumPy arrays (`self.starts`, `self.ends`).
- Sort intervals by start time. When mapping a word, use `np.searchsorted` to find candidates via binary search, reducing intersection matching to $O(\log n)$ average query time.

### Phase 2: PyAnnote Pipeline Execution
- `DiarizationPipeline` wraps `pyannote.audio.Pipeline`.
- Requires a Hugging Face user access token (`hf_token`) to download pre-trained gated weights (`pyannote/speaker-diarization-community-1`).
- Runs inference on the audio file, returning a list of segments with start, end, and speaker label attributes.

### Phase 3: Speaker-to-Word Assignment
- `assign_word_speakers()` takes the diarization result and aligned transcription result.
- Instantiates an `IntervalTree` with the speaker segments.
- For each word in each segment, queries the tree.
- If overlaps exist, assigns the speaker with the longest overlapping duration.
- If no overlaps exist, calls `IntervalTree.find_nearest()` to assign the closest speaker midpoint.
