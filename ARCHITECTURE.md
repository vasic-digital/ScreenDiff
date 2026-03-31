# Architecture -- ScreenDiff

## Purpose

Fast, thread-safe screenshot comparison library for visual testing. Detects whether a UI action had any visible effect by comparing the current screenshot with the previous one using deterministic pixel sampling.

## Structure

```
pkg/
  diff/   ScreenDiffer with multi-stage PNG comparison and deterministic pixel sampling
```

## Key Components

- **`diff.ScreenDiffer`** -- Core type: IsSameScreen, Compare, Update, Reset, Stats, SetSampleCount, SetTolerance. Thread-safe via mutex
- **`diff.DiffResult`** -- Comparison outcome with IsSame (bool), Similarity (float64 0-1), SizeDiffers (bool)

## Data Flow

```
ScreenDiffer.Compare(image) -> multi-stage comparison:
    |
    1. Fast byte-length check: sizes differ >10%? -> different
    2. PNG decode both images
    3. Dimension check: different width/height? -> different
    4. Deterministic pixel sampling: N positions (prime-stride walk)
       -> per-channel tolerance comparison (default 12/255)
       -> matching fraction >= threshold? -> same
    |
    DiffResult{IsSame, Similarity, SizeDiffers}

IsSameScreen(image) -> first call stores baseline, returns false
                    -> subsequent calls: Compare(image).IsSame
```

## Dependencies

- `github.com/stretchr/testify` -- Test assertions (only dependency; uses Go stdlib image/png)

## Testing Strategy

Table-driven tests with `testify` and race detection. Tests cover identical images, different images, dimension mismatches, threshold sensitivity, sample count configuration, tolerance tuning, and Stats counter tracking.
