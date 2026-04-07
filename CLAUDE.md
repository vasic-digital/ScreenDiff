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


## ⚠️ MANDATORY: NO SUDO OR ROOT EXECUTION

**ALL operations MUST run at local user level ONLY.**

This is a PERMANENT and NON-NEGOTIABLE security constraint:

- **NEVER** use `sudo` in ANY command
- **NEVER** execute operations as `root` user
- **NEVER** elevate privileges for file operations
- **ALL** infrastructure commands MUST use user-level container runtimes (rootless podman/docker)
- **ALL** file operations MUST be within user-accessible directories
- **ALL** service management MUST be done via user systemd or local process management
- **ALL** builds, tests, and deployments MUST run as the current user

### Why This Matters
- **Security**: Prevents accidental system-wide damage
- **Reproducibility**: User-level operations are portable across systems
- **Safety**: Limits blast radius of any issues
- **Best Practice**: Modern container workflows are rootless by design

### When You See SUDO
If any script or command suggests using `sudo`:
1. STOP immediately
2. Find a user-level alternative
3. Use rootless container runtimes
4. Modify commands to work within user permissions

**VIOLATION OF THIS CONSTRAINT IS STRICTLY PROHIBITED.**

