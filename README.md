# digital.vasic.screendiff

A fast, thread-safe screenshot comparison library for visual testing. Detects whether a UI action had any visible effect by comparing the current screenshot with the previous one using deterministic pixel sampling.

## Installation

```bash
go get digital.vasic.screendiff
```

## Quick Start

```go
package main

import (
    "fmt"

    "digital.vasic.screendiff/pkg/diff"
)

func main() {
    // Create a differ with 99% similarity threshold.
    sd := diff.NewScreenDiffer(0.99)

    // Compare screenshots (PNG-encoded byte slices).
    screenshotA := captureScreen() // your capture logic
    screenshotB := captureScreen()

    // First call stores the image, returns false.
    sd.IsSameScreen(screenshotA)

    // Subsequent calls compare against the stored image.
    if sd.IsSameScreen(screenshotB) {
        fmt.Println("No visible change detected")
    } else {
        fmt.Println("Screen changed")
    }

    // Get detailed comparison results.
    result := sd.Compare(screenshotB)
    fmt.Printf("Same: %v, Similarity: %.2f%%\n",
        result.IsSame, result.Similarity*100)

    // Check statistics.
    same, different := sd.Stats()
    fmt.Printf("Same: %d, Different: %d\n", same, different)
}
```

## API Reference

### ScreenDiffer

The core type. All methods are safe for concurrent use.

| Method | Description |
|--------|-------------|
| `NewScreenDiffer(threshold float64)` | Create a differ. Threshold in (0, 1], defaults to 0.99 |
| `IsSameScreen(image []byte) bool` | Quick same/different check |
| `Compare(image []byte) DiffResult` | Detailed comparison with similarity score |
| `Update(image []byte)` | Set baseline without comparing |
| `Reset()` | Clear stored image and counters |
| `Stats() (same, diff int)` | Get comparison counters |
| `Threshold() float64` | Get configured threshold |
| `HasPrevious() bool` | Check if a baseline image is stored |
| `SetSampleCount(n int)` | Override pixel sample count (default 500) |
| `SetTolerance(t int)` | Override per-channel tolerance 0-255 (default 12) |

### DiffResult

Returned by `Compare()`.

| Field | Type | Description |
|-------|------|-------------|
| `IsSame` | `bool` | True when screens are nearly identical |
| `Similarity` | `float64` | Fraction of matching samples (0-1) |
| `SizeDiffers` | `bool` | True when raw byte lengths differ by >10% |

### Comparison Algorithm

1. **Fast byte-length check** -- if sizes differ by >10%, screens are different
2. **PNG decoding** -- both images are decoded
3. **Dimension check** -- different dimensions means different screens
4. **Pixel sampling** -- N deterministic pixel positions compared with channel tolerance
5. **Threshold** -- if matching fraction exceeds threshold, screens are the same

## License

Apache-2.0
