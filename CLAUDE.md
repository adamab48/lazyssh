# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
make run          # Run from source (go run ./cmd/main.go)
make build        # Build binary to ./bin/lazyssh (runs quality checks first)
make test         # Run tests with race detector and coverage
make test-verbose # Run tests with verbose output
make lint         # Format + golangci-lint
make lint-fix     # Format + golangci-lint with auto-fix
make fmt          # Format with gofumpt + go fmt
make quality      # fmt + vet + lint
make tools        # Install golangci-lint, gofumpt, staticcheck locally to ./bin/
```

Run a single test:
```bash
go test -run TestName ./internal/...
```

## Architecture

Hexagonal (ports & adapters) architecture:

```
cmd/main.go                         # Wires dependencies, starts cobra CLI → tui.Run()
internal/core/domain/server.go      # Server struct (the central domain model)
internal/core/ports/                # Interfaces: ServerRepository, ServerService
internal/core/services/             # ServerService impl: SSH, ping, CRUD, forwarding
internal/adapters/data/ssh_config_file/   # Repository: reads/writes ~/.ssh/config
internal/adapters/ui/               # tview TUI: all UI components
internal/logger/                    # zap-based structured logger
```

**Data flow:** `tui` → `ServerService` (port) → `ServerServiceImpl` → `ServerRepository` (port) → `ssh_config_file` adapter (reads/writes `~/.ssh/config`).

**Metadata** (pins, last-seen, SSH count, tags) is stored separately in `~/.lazyssh/metadata.json` via `MetadataManager`, keeping `~/.ssh/config` clean.

**ssh_config dependency:** The `go.mod` has a `replace` directive pointing `github.com/kevinburke/ssh_config` to a fork (`github.com/adamab48/ssh_config`) — this fork is used for non-destructive config parsing that preserves comments and formatting.

**UI components** in `internal/adapters/ui/`:
- `tui.go` — root app struct, layout construction, global key bindings
- `handlers.go` — event handlers for all key actions
- `server_list.go`, `server_details.go`, `server_form.go` — list, detail panel, add/edit form
- `search_bar.go` — fuzzy search input
- `validation.go` — form field validation
- `sort.go` — sort modes (by alias, by last SSH, ascending/descending)

## Code conventions

- All Go source files must begin with the Apache 2.0 license header (enforced by `goheader` linter).
- `gofumpt` is used for formatting (stricter than `gofmt`).
- Platform-specific syscall attributes use build-tag files: `sysprocattr_unix.go` / `sysprocattr_windows.go`.

## PR conventions

Semantic PR titles are enforced by CI. Format: `type(scope): description`

Allowed types: `feat`, `fix`, `improve`, `refactor`, `docs`, `test`, `ci`, `chore`, `revert`  
Allowed scopes (optional): `ui`, `cli`, `config`, `parser`
