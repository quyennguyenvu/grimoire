---
name: go-runes
description: Use when writing, reviewing, or refactoring Go (Golang) code, or answering a Go design question — error handling, concurrency, generics, testing, tooling, modules, package layout, API design, performance, pointers vs. zero values.
---

# Go Conventions

Idiomatic, race-free, and `gofmt`-clean by default. Feature notes cite the minimum Go version.

- **Error handling**: Return `error` as the last value; wrap with `fmt.Errorf("...: %w", err)` and inspect via `errors.Is`/`errors.As`. Declare sentinels as `var ErrX = errors.New(...)`; aggregate with `errors.Join` (1.20+). Panic only for programmer bugs or unrecoverable init — never across package boundaries.
- **Concurrency**: Goroutines + channels for concurrent work; every goroutine needs a guaranteed exit path to avoid leaks. Pass `context.Context` as the first arg (`ctx`) for cancellation/deadlines; coordinate with `sync` primitives or `errgroup`; prefer unbuffered channels unless back-pressure requires buffering (the sender closes, never the receiver). Validate with `go test -race`.
- **Generics**: Use type parameters (1.18+) for container/algorithm code instead of `interface{}` + reflection; fall back to a plain interface when one behavior suffices — don't over-parameterize.
- **Testing**: Table-driven subtests via `t.Run`, `t.Parallel()` where isolated, golden files for large fixtures, `httptest` for handlers. Add benchmarks (`testing.B`, `-benchmem`) for hot paths and fuzz targets (`testing.F`, 1.18+) for parsers. Prefer the standard library over heavy assertion frameworks.
- **Tooling**: Run `goimports`/`gofmt` on save; enforce `go vet` + `staticcheck` (or `golangci-lint`) in CI; keep dependencies clean with `go mod tidy`.
- **Modules**: Manage deps via `go.mod`/`go.sum` with semantic import versioning and minimal version selection; commit `go.sum`. Use `internal/` for compiler-enforced encapsulation.
- **Code organization**: `cmd/<binary>` for entrypoints, `internal/` for private packages, small packages named for what they provide (not `utils`). `pkg/` is a community convention — not official Go layout — so don't cargo-cult it.
- **API design**: Accept interfaces, return concrete types; define interfaces at the consumer and keep them small (1–3 methods). Make the zero value useful; use the functional-options pattern for optional config.
- **Performance**: Profile with `pprof` + benchmarks before optimizing. Preallocate slices/maps at known capacity, watch escape analysis and allocations, and reuse hot-path objects with `sync.Pool`. Default to value semantics; use pointers for large structs or required mutation.
- **Pointers vs zero values**: Don't use `*int`/`*string`/`*bool` on primitives when the zero value already means "empty" — branch on the zero value instead of a `*T` + nil check to eliminate nil-deref panics. Reserve a pointer (or `sql.Null*` / a separate `ok bool`) for when you must distinguish _unset_ from _zero_ — e.g., JSON `omitempty`, PATCH payloads, nullable DB columns.
- **Idioms**: Follow `gofmt` naming (MixedCaps, short names in small scopes); use `defer` for cleanup (beware defer-in-loop accumulation); handle every error — never blank-assign (`_ = err`) without a stated reason.
