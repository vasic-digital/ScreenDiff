# CLAUDE.md - ScreenDiff Module

## Overview

`digital.vasic.screendiff` is a generic, reusable Go module for fast screenshot comparison using deterministic pixel sampling. It detects whether two PNG screenshots represent the same screen state.

**Module**: `digital.vasic.screendiff` (Go 1.24+)

## Build & Test

```bash
go build ./...
go test ./... -count=1 -race
go vet ./...
```

## Code Style

- Standard Go conventions, `gofmt` formatting
- Imports grouped: stdlib, third-party, internal (blank line separated)
- Line length target 80 chars (100 max)
- Naming: `camelCase` private, `PascalCase` exported
- Errors: always check, wrap with `fmt.Errorf("...: %w", err)`
- Tests: table-driven where appropriate, `testify`, naming `Test<Struct>_<Method>_<Scenario>`
- SPDX headers on every .go file

## Package Structure

| Package | Purpose |
|---------|---------|
| `pkg/diff` | ScreenDiffer with multi-stage PNG comparison and pixel sampling |

## Key Types

- `diff.ScreenDiffer` -- Thread-safe screenshot comparator with configurable threshold
- `diff.DiffResult` -- Comparison outcome with similarity score and size-differs flag

## Design Patterns

- **Multi-Stage Comparison**: byte-length, decode, dimension, pixel sampling
- **Deterministic Sampling**: prime-stride walk through pixel space for reproducibility
- **Thread Safety**: all state protected by mutex

## Constraints

- **No CI/CD pipelines** -- no GitHub Actions, no GitLab CI
- **Generic library** -- no application-specific logic
- **PNG only** -- input must be PNG-encoded byte slices

## Commit Style

Conventional Commits: `feat(diff): add JPEG support`
