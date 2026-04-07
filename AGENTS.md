# AGENTS.md - Multi-Agent Coordination Guide

## Overview

This document provides guidance for AI agents working with the `digital.vasic.screendiff` module. It describes conventions, coordination patterns, and boundaries that agents must respect.

## Module Identity

- **Module path**: `digital.vasic.screendiff`
- **Language**: Go 1.24+
- **Dependencies**: `github.com/stretchr/testify` (tests only)
- **Scope**: Generic, reusable screenshot comparison. No application-specific logic.

## Package Responsibilities

| Package | Owner Concern | Agent Must Not |
|---------|--------------|----------------|
| `pkg/diff` | Screenshot comparison, pixel sampling, threshold logic | Add application-specific logic, import non-stdlib production deps |

## Coordination Rules

### 1. Thread Safety Invariants

Every exported method on `ScreenDiffer` is safe for concurrent use. Agents must:

- Never remove mutex protection from shared state.
- Never introduce a public method that requires external synchronization.
- Always run `go test -race` after changes.

### 2. Interface Contracts

The `ScreenDiffer` API is a stability boundary. Breaking changes require explicit human approval:

- `NewScreenDiffer(threshold)` constructor signature
- `Compare(image) DiffResult` return type and fields
- `IsSameScreen(image) bool` behavior contract

### 3. Test Requirements

- All tests use `testify/assert` and `testify/require`.
- Test naming convention: `Test<Struct>_<Method>_<Scenario>`.
- Race detector must pass: `go test ./... -race`.

## Agent Workflow

### Before Making Changes

```bash
go build ./...
go test ./... -count=1 -race
```

### After Making Changes

```bash
gofmt -w .
go vet ./...
go test ./... -count=1 -race
```

### Commit Convention

```
<type>(<package>): <description>

# Examples:
feat(diff): add JPEG decode support
fix(diff): correct tolerance scaling for 16-bit channels
test(diff): add edge case for zero-dimension images
```

## Boundaries

### What Agents May Do

- Fix bugs in any package.
- Add tests for uncovered code paths.
- Refactor internals without changing exported APIs.
- Add new exported methods that extend existing types.
- Update documentation to match code.

### What Agents Must Not Do

- Break existing exported interfaces or method signatures.
- Remove thread safety guarantees.
- Add application-specific logic (this is a generic library).
- Introduce new external dependencies without human approval.
- Modify `go.mod` without explicit instruction.

## Key Files

| File | Purpose |
|------|---------|
| `pkg/diff/diff.go` | All production code |
| `pkg/diff/diff_test.go` | All tests |
| `go.mod` | Module definition |
| `README.md` | User-facing documentation |
| `CLAUDE.md` | Agent build/test guidance |


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

