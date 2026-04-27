# What changed in the last 10 commits of `logrus`

I walked the linear history starting at `HEAD` (`5649d7c`) and read each commit's diff. The 10 most recent non-merge commits cluster into four clear themes: a real bug fix (tinygo), interface tidy-ups, GoDoc/README polish, and one ancient JSON-formatter fix that finally landed.

Most-recent first:

| # | SHA | Subject | Files | +/- |
|---|-----|---------|-------|-----|
| 1 | `5649d7c` | README: tweak section about maintenance mode | README.md | +4 / -7 |
| 2 | `a328451` | README: remove details about lowercase rename | README.md | +5 / -15 |
| 3 | `d258d28` | README: fix example for caller benchmarks | README.md | +1 / -1 |
| 4 | `bdaca74` | touch-up godoc and add godoc-links | entry.go, exported.go, formatter.go, logger.go, logrus.go | +72 / -74 |
| 5 | `fa4818c` | fix: build against tinygo | terminal_check_{bsd,notappengine,solaris,tinygo,unix}.go | +11 / -4 |
| 6 | `1ed0ade` | add argument names to StdLogger interface | logrus.go | +9 / -9 |
| 7 | `611c346` | cleanup FieldLogger interface | logrus.go | +14 / -26 |
| 8 | `2ab3c8f` | README: correct signature of Fields-type | README.md | +1 / -1 |
| 9 | `a41edc6` | TextFormatter: touch-up GoDoc for quoting | text_formatter.go | +20 / -5 |
| 10 | `faf2645` | JSONFormatter: omit empty DataKey wrapper when no fields are present | json_formatter.go | +1 / -1 |

(Commits 1, 4, 5, 6 landed in the last ~3 days; commit 10 is a 5-year-old PR that just got merged.)

## The only real code/behavior changes

### `fa4818c` — fix: build against tinygo
The one true bug fix in the range. Logrus's terminal-detection used the `unix` syscall package, which TinyGo doesn't implement, so `tinygo build ./...` failed.

- New file `terminal_check_tinygo.go` with `//go:build tinygo`, providing a no-op `checkIfTerminal(_ any) bool { return false }`.
- All other terminal_check_*.go files had their build tags amended with `&& !tinygo` so the platform-specific implementations don't fight the new stub. E.g. `terminal_check_unix.go`:

  ```diff
  -//go:build linux || aix || zos
  +//go:build (linux || aix || zos) && !tinygo
  ```

Net effect: TinyGo users always get "not a terminal," which is the safe answer since TinyGo can't query the tty anyway. No behavior change on supported platforms.

### `faf2645` — JSONFormatter empty DataKey
Tiny correctness fix in `json_formatter.go`:

```diff
-if f.DataKey != "" {
+if f.DataKey != "" && len(entry.Data) > 0 {
```

Previously, if you set `DataKey` but logged a message with no fields, the JSON output still got an empty wrapper object under that key. Now the wrapper is only emitted when there's data to wrap.

## Interface cleanup (API surface, but compatible)

### `611c346` — cleanup FieldLogger interface
Refactored `FieldLogger` in `logrus.go`. It used to redeclare every `Printf/Print/Fatal*` method that `StdLogger` already had; now it just embeds `StdLogger` and only adds the logrus-specific levels (`Debug*`, `Info*`, `Warn*`, `Warning*`, `Error*`). Methods are also regrouped per-level rather than per-suffix so the interface reads top-to-bottom by severity. Also drops a block of commented-out `IsXxxEnabled() bool` placeholders. Net `+14 / -26` — pure simplification, same exported surface.

### `1ed0ade` — add argument names to StdLogger interface
Cosmetic but useful: the `StdLogger` interface methods previously declared parameter types only (`Printf(string, ...any)`); now they have names (`Printf(format string, args ...any)`), matching the convention used by the other interfaces in the same file. Better godoc, no behavior change.

## Documentation: GoDoc and README

### `bdaca74` — touch-up godoc and add godoc-links
Largest commit in the range by line count (`+72 / -74`) but it's all comments. Touches `entry.go`, `exported.go`, `formatter.go`, `logger.go`, `logrus.go`. The pattern is converting bare references like `Logger` or `Entry` into godoc-aware `[Logger]` / `[Entry]` cross-reference syntax, plus light wording cleanups. No code under the hood changed.

### `a41edc6` — TextFormatter: touch-up GoDoc for quoting
Expands the doc comment on `TextFormatter` (`text_formatter.go`) to spell out that it produces logfmt-style `key=value` output, that **only values** are quoted/escaped (keys are written verbatim), and to recommend `JSONFormatter` if you need fully escaped output including safe keys. Also adds a comment on `appendKeyValue` and reformats the `FieldMap` example. Code unchanged.

### README commits (`5649d7c`, `a328451`, `d258d28`, `2ab3c8f`)
A four-commit README cleanup pass:

- **`5649d7c`** rewrites the maintenance-mode blurb. New version is shorter and explicitly mentions interop with Go's `log/slog` as the one kind of change still on the table.
- **`a328451`** removes a long stale section about the historical `Sirupsen` → `sirupsen` lowercase rename plus Glide instructions. The "Case-sensitivity" section further down is shortened to a single paragraph linking out to the original GitHub comment.
- **`d258d28`** fixes a copy-paste in a `go test -bench` example: `-bench=.*CallerTracing` → `-bench=ReportCaller` (the actual benchmark name in the codebase).
- **`2ab3c8f`** updates the documented signature of `Fields` from `map[string]interface{}` to `map[string]any`, matching the modern Go alias used throughout the source.

## Summary

If you only care about runtime behavior, two commits matter:
1. `fa4818c` — TinyGo can now build logrus.
2. `faf2645` — `JSONFormatter` no longer emits an empty `DataKey` wrapper when there are no fields.

Everything else is interface tidying (`611c346`, `1ed0ade`) and doc/README polish (`bdaca74`, `a41edc6`, `5649d7c`, `a328451`, `d258d28`, `2ab3c8f`). Consistent with the project's stated maintenance-mode posture: small, conservative, no new features.
