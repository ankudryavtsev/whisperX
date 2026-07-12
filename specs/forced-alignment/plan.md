# Implementation Plan: Forced Phoneme Alignment

**Branch**: `main` | **Date**: 2026-07-12 | **Spec**: [spec.md](file:///home/andrey/Workspace/whisperX/specs/forced-alignment/spec.md)

## Summary
Reverse-engineered implementation details of the forced phoneme alignment pipeline in whisperX. The system maps the transcription output of Whisper onto a CTC-based acoustic model (Wav2Vec2) to resolve word boundaries.

## Technical Context
- **Language/Version**: Python 3.10+
- **Primary Dependencies**: `torch`, `torchaudio`, `transformers` (Wav2Vec2 CTC models), `nltk`
- **Testing**: `pytest`
- **Target Platform**: Cross-platform (CPU and CUDA GPUs)
- **Performance Goals**: Fast alignment runtime (under 1/30th of audio length)

## Constitution Check
- **Naming Conventions Match**: All core modules are `snake_case` (e.g. `alignment.py`, `conjunctions.py`, `schema.py`). ✅ Pass.
- **Pipeline Isolation Maintained**: Alignment modules exist as pure processing functions that consume a transcript and waveform, returning structured results. No dependency on ASR or Diarization classes. ✅ Pass.
- **Testing**: Test suite executes via `pytest`. ✅ Pass.

## Project Structure
The alignment feature components are distributed as follows:

```text
whisperx/
├── alignment.py          # Core alignment engine (trellis computation, backtracking, interpolation)
├── conjunctions.py       # Language-specific word-cleaning parameters
├── schema.py             # Schema TypedDict definitions (SingleWordSegment, SingleAlignedSegment)
└── utils.py              # Contains interpolate_nans utility function

tests/
└── test_word_timestamp_interpolation.py  # End-to-end tests using mock emissions
```

## Implementation Details

### Phase 1: Model Management
- The system checks if a pre-compiled Torchaudio bundle exists for the requested language.
- If not, it attempts to load a corresponding Hugging Face model (`jonatasgrosman/wav2vec2-large-xlsr-...`).
- Exposes `load_align_model()` which returns the model instance and metadata containing the character dictionary and mapping type.

### Phase 2: Frame Emission Trellis
- The audio waveform is forwarded through the Wav2Vec2 network to generate a logit emission matrix.
- A dynamic programming grid (trellis) is initialized of size `(num_frames + 1, num_tokens + 1)`.
- For each step, it calculates transition probabilities: staying on the same character or transitioning to the next token/blank state.

### Phase 3: Path Backtracking
- Once the trellis grid is populated, the path is reconstructed by walking backward from the final frame and character to the starting state.
- Path tokens are merged to form words, utilizing the boundary character (`|`) as a separator.

### Phase 4: Interpolation of Missing Timestamps
- Words containing only unaligned characters (such as digits, punctuation, or out-of-vocabulary terms) will output `NaN` for start and end timings.
- The system calls `interpolate_nans()` to interpolate these timings based on adjacent valid boundaries, using nearest-neighbor or linear algorithms.
