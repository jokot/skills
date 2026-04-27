---
question: what changed in the last 10 commits
scope: commit:HEAD~10..HEAD
generated_at: 2026-04-25T00:00:00Z
visited_files: 21
depth_reached: 2
budget_state: complete
---

# What Changed In The Last 10 Commits

## Overview

The HEAD~10..HEAD window on this `logrus` checkout is overwhelmingly a maintenance pass — documentation, godoc-link conversion, interface tidy-ups, CI hardening, and one tinygo build-fix. Only two commits have any user-visible runtime effect: `faf2645` (JSONFormatter no longer emits an empty `DataKey` wrapper when there are no fields, see `json_formatter.go:78`) and `fa4818c` (the package now builds under tinygo via a new stub `terminal_check_tinygo.go:1-7` plus `&& !tinygo` build-tag guards on the four unix-family terminal-check files).

The largest churn is in `logrus.go:131-173`, where commits `611c346` and `1ed0ade` reshape the `StdLogger` and `FieldLogger` interfaces. `FieldLogger` now embeds `StdLogger` and groups methods per log-level instead of listing all `*f` then all base then all `*ln` variants; `StdLogger`'s parameter list grew named arguments. This is a source-compatible reshuffle for any in-repo consumer (`logrus_test.go:648` is the only one), but external implementers of `FieldLogger` should re-check method-set coverage because `Fatal*`/`Panic*` are now inherited via `StdLogger` rather than declared inline.

The remaining commits are pure prose: `bdaca74` rewrites godoc across `entry.go`, `exported.go`, `formatter.go`, `logger.go` and `logrus.go` to use `[bracket]` godoc-links (`formatter.go:29-41`, `logger.go:199-205`); `a41edc6` clarifies in `text_formatter.go:18-26` that only values are quoted/escaped, keys are written verbatim; `2ab3c8f`/`d258d28`/`a328451`/`5649d7c` are README-only; `760822c` backfills CHANGELOG entries for 1.9.0 through 1.9.4 (`CHANGELOG.md:1-58`); `c68f1d5` updates `.github/workflows/ci.yaml` with explicit permissions, a concurrency group, and step-order cleanups.

## Entry point

The natural entry point for this scope is the diff list itself. The commit walk hangs on `logrus.go`, which is the only file modified by two of the ten commits and which defines the public interfaces consumers actually depend on:

```go
// logrus.go:131-143  (post 1ed0ade + 611c346)
type StdLogger interface {
	Print(args ...any)
	Printf(format string, args ...any)
	Println(args ...any)
	Fatal(args ...any)
	Fatalf(format string, args ...any)
	Fatalln(args ...any)
	Panic(args ...any)
	Panicf(format string, args ...any)
	Panicln(args ...any)
}
```

The `var _ StdLogger = &log.Logger{}` / `&Entry{}` / `&Logger{}` triple at `logrus.go:122-126` is the compile-time check that pins the interface to its three implementers; that check survives both edits unchanged, which is why the cleanup is safe.

## Sequence

Chronological commit walk (oldest first within the HEAD~10..HEAD window):

1. `760822c` — `CHANGELOG.md:1-58` adds version sections 1.9.0 through 1.9.4: GNU/Hurd + WASI wasip1 support, removal of deprecated `ioutil`, the Writer DoS fix history (#1376 applied/reverted/re-applied across 1.9.1–1.9.3), and min Go bumped to 1.17. No code change.
2. `c68f1d5` — `.github/workflows/ci.yaml:1-90` adds top-level `permissions: contents: read, pull-requests: read`, adds a `concurrency` group keyed on `${{ github.workflow }}-${{ github.ref }}` with `cancel-in-progress: true`, collapses the verbose `on:` block to `on: [push, pull_request]`, drops the single-element platform matrix on the `lint` job, sets bash as the default shell on the `test` job, and reorders Checkout before Install Go.
3. `faf2645` — `json_formatter.go:78` changes `if f.DataKey != ""` to `if f.DataKey != "" && len(entry.Data) > 0`. **Behavioral**: when `DataKey` is set but the entry has no extra fields, the formatter no longer wraps an empty map under that key — output stops including a noise-only `"datakey":{}`.
4. `a41edc6` — `text_formatter.go:18-26,82-95,303-304` expands `TextFormatter` godoc to spell out that the output is logfmt-like, that only values are subject to quoting/escaping, and that keys (including `FieldMap` overrides) are written verbatim; recommends `JSONFormatter` for fully-escaped keys. The `FieldMap` example block is reformatted to gofmt style.
5. `2ab3c8f` — `README.md` corrects the `Fields` type signature shown in the docs to `map[string]any` (the actual definition at `logrus.go:10`).
6. `611c346` — `logrus.go:145-173` cleans up `FieldLogger`: embeds `StdLogger` instead of repeating `Print*`/`Fatal*`/`Panic*` declarations, regroups the level methods so `Debug`, `Debugf`, `Debugln` sit together (then Info, Warn, Warning, Error), drops the commented-out `// IsDebugEnabled() bool` stubs that had lived there for years. Net `-26 +14` lines, no method-set change for the embedded portion.
7. `1ed0ade` — `logrus.go:131-143` adds parameter names (`args`, `format`) to the `StdLogger` interface methods so godoc renders them like the rest of the file; no semantic change.
8. `fa4818c` — adds `terminal_check_tinygo.go:1-7` (new file, `//go:build tinygo`, returns `false`) and changes the build tags on `terminal_check_bsd.go:1`, `terminal_check_unix.go:1`, `terminal_check_solaris.go:1`, `terminal_check_notappengine.go:1` to AND `!tinygo`. Result: under tinygo, no file imports the `unix` syscalls package, so `go build ./...` succeeds.
9. `bdaca74` — godoc-link pass across `entry.go` (`NewEntry`, `Dup`, `Entry.Log`), `exported.go` (newly documents `StandardLogger`, converts `WithError`/`WithField`/etc. to `[ErrorKey]`-style links), `formatter.go:29-41` (rewrites the `Formatter` doc and references `entry.Message` / `Entry.Data` / `Logger.Out` rather than the old `entry.Data["msg"]` strings), `logger.go:199-205` (`Logger.Log` warning is rephrased to reference `[Logger.Panic]` / `[Logger.Fatal]`), and one stray reference fix in `logrus.go`. Doc-only.
10. `d258d28` — `README.md` fixes the example shown for caller-related benchmarks (one-line correction).
11. `a328451` — `README.md` removes 15 lines of stale narrative about the `Sirupsen → sirupsen` lowercase rename ("This should be widely known by now").
12. `5649d7c` — `README.md` rewrites the maintenance-mode section (4 lines added, 7 removed).

(Steps 1–12 reflect the 10 non-merge commits plus their two associated merges that fall in the `HEAD~10..HEAD` window. The merges themselves carry no diff and are omitted.)

## Files touched

- `logrus.go` — public interface definitions (`StdLogger`, `FieldLogger`, `Ext1FieldLogger`, `Level`, `Fields`); modified by `611c346`, `1ed0ade`, `bdaca74`.
- `exported.go` — package-level convenience wrappers around the `std` logger; godoc-link sweep (`bdaca74`).
- `entry.go` — `Entry` struct + per-entry logging methods; godoc-link sweep on `NewEntry`/`Dup`/`Entry.Log` (`bdaca74`).
- `logger.go` — `Logger` struct + per-logger logging methods; godoc clarification on `Logger.Log` (`bdaca74`).
- `formatter.go` — `Formatter` interface + shared field-key constants + `prefixFieldClashes`; godoc rewrite (`bdaca74`).
- `json_formatter.go` — `JSONFormatter.Format`; behavioral fix for empty `DataKey` wrapper (`faf2645`).
- `text_formatter.go` — `TextFormatter` struct, `Format`, `printColored`, `appendKeyValue`; godoc expansion + example reflow (`a41edc6`).
- `terminal_check_tinygo.go` — **new** stub `checkIfTerminal` returning `false` under `//go:build tinygo` (`fa4818c`).
- `terminal_check_bsd.go`, `terminal_check_unix.go`, `terminal_check_solaris.go`, `terminal_check_notappengine.go` — build tags AND'd with `!tinygo` (`fa4818c`).
- `README.md` — four small doc commits (`2ab3c8f`, `d258d28`, `a328451`, `5649d7c`).
- `CHANGELOG.md` — backfilled 1.9.0–1.9.4 (`760822c`).
- `.github/workflows/ci.yaml` — permissions, concurrency, step reorder (`c68f1d5`).

BFS-visited but unchanged in-window (read for ripple checking):

- `logrus_test.go:646-650` — only in-repo consumer of `FieldLogger`/`Ext1FieldLogger`; still compiles after the interface reshuffle.
- `terminal_check_no_terminal.go`, `terminal_check_appengine.go`, `terminal_check_windows.go` — sibling `checkIfTerminal` implementations the new tinygo file mirrors.
- `json_formatter_test.go:170` — exercises `DataKey: "args"`; explains existing coverage of the `faf2645` path.
- `text_formatter_test.go:579` — calls `checkIfTerminal(os.Stderr)`, confirming the tinygo stub matches the expected signature.

## Gotchas

- **`FieldLogger` reshuffle is source-compatible only because `StdLogger` is embedded** (`logrus.go:152`). External code that defined a struct method-by-method against the *old* `FieldLogger` will still satisfy the new one since the method set is unchanged, but anyone who copy-pasted the interface verbatim into another package now has a stale list missing `Fatal*`/`Panic*` (those moved into `StdLogger`). Re-paste from `logrus.go` if you maintain such a mirror.
- **`json_formatter.go:78` change is observable in JSON output.** Tests/dashboards that grepped for the empty `"<DataKey>":{}` object will stop seeing it once `DataKey` is set but no fields are added. This is the only behavioral diff in the window — flag it on upgrade notes.
- **The tinygo stub uses `_ any` not `_ io.Writer`** (`terminal_check_tinygo.go:5`), matching `terminal_check_no_terminal.go:5` and `terminal_check_appengine.go:5`. The unix/bsd/solaris/notappengine variants take `io.Writer`. Swapping a unix tag onto the tinygo file would not compile cleanly without also adding the `io` import; do not "simplify" by deleting the stub.
- **CI permissions are now restrictive by default** (`.github/workflows/ci.yaml:7-15`). Any future job that needs to write (e.g. push tags, comment on PRs) must explicitly grant the missing permission — the implicit "everything" default is gone.
- **`Logger.Log(PanicLevel, ...)` does not panic** (`logger.go:199-205`). The godoc clarification in `bdaca74` exists because this is a real, easy-to-make mistake; use `Logger.Panic` for panic semantics.
- **The 10-commit range as a `git log` produces 12 commits including 2 merges**; the diff itself (`git diff HEAD~10..HEAD`) is the canonical "what changed" because merge commits add no new content.

## Open questions

- Did `611c346` or `1ed0ade` change anything in `entry.go` or `logger.go`'s method set that would break a struct *not* in this repo from satisfying `FieldLogger`? I read both `entry.go` and `logger.go` and did not see a method removal, but the answer ultimately depends on third-party implementations I cannot inspect from this checkout.
- The `c68f1d5` CI changes include a `cross` job and a `test` job whose full post-edit content I only sampled (~90 of 130 lines via `git show`). If you care about the exact go-version matrix or the cross-compilation target list, run `git show c68f1d5 -- .github/workflows/ci.yaml` for the rest.
- `760822c`'s changelog claims a Writer DoS fix and revert/re-apply cycle (#1376). The actual `writer.go` is not in this 10-commit window, so the fix predates HEAD~10; if you need the fix details, `git log --oneline -- writer.go` from the right ancestor will surface it.
