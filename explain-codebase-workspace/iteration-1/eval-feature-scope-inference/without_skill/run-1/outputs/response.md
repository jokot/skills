# How logrus adds hooks to log entries

Hooks in logrus are user-supplied callbacks that fire on each log event for the levels they opt into. The whole feature lives in three small pieces: a `Hook` interface, a per-logger `LevelHooks` map, and a fire step inside `entry.log`.

## 1. The `Hook` interface

Defined in `hooks.go`:

```go
type Hook interface {
    Levels() []Level
    Fire(*Entry) error
}
```

- `Levels()` tells logrus which severity levels this hook cares about.
- `Fire(*Entry)` is invoked for each matching log event. The hook receives the live `*Entry` (so it can read `entry.Data`, `entry.Message`, `entry.Time`, `entry.Context`, etc., and may mutate it).

The doc comment is explicit that `Fire` runs synchronously on the logging goroutine — if you want async behavior, the hook implementation has to provide it.

## 2. Storage: `LevelHooks` map

Also in `hooks.go`:

```go
type LevelHooks map[Level][]Hook

func (hooks LevelHooks) Add(hook Hook) {
    for _, level := range hook.Levels() {
        hooks[level] = append(hooks[level], hook)
    }
}

func (hooks LevelHooks) Fire(level Level, entry *Entry) error {
    for _, hook := range hooks[level] {
        if err := hook.Fire(entry); err != nil {
            return err
        }
    }
    return nil
}
```

So registration is just a fan-out: when you `Add` a hook, logrus asks the hook which levels it wants and appends it to the slice under each of those level keys. Lookup at fire time is an O(1) map read followed by iterating that level's slice.

## 3. Adding hooks to a logger

Each `Logger` has a `Hooks LevelHooks` field, initialized to an empty map in `New()` (`logger.go`). There are three ways to register:

- **Per-logger method** (`logger.go`):

  ```go
  func (logger *Logger) AddHook(hook Hook) {
      logger.mu.Lock()
      defer logger.mu.Unlock()
      logger.Hooks.Add(hook)
  }
  ```

  This is the recommended path — it takes the logger's mutex so concurrent `AddHook`/log calls are safe.

- **Direct map access**: `log.Hooks.Add(myHook)` (used in `example_hook_test.go`). Works because `LevelHooks.Add` is exported, but does not lock — fine during setup, racy after the logger is in use.

- **Package-level shortcut** for the standard logger (`exported.go`):

  ```go
  func AddHook(hook Hook) { std.AddHook(hook) }
  ```

There is also `logger.ReplaceHooks(newHooks LevelHooks) LevelHooks` for swapping the entire set atomically (useful in tests).

## 4. Firing hooks during a log call

The actual invocation happens in `Entry.log` (`entry.go` around line 299):

```go
// Select hooks based on the level for this log call.
hooks := logger.hooksForLevel(level)
newEntry.fireHooks(hooks)
```

`hooksForLevel` (`logger.go`) takes the logger mutex, looks up `logger.Hooks[level]`, and returns a **shallow copy** of the slice:

```go
func (logger *Logger) hooksForLevel(level Level) []Hook {
    logger.mu.Lock()
    hooks := logger.Hooks[level]
    if len(hooks) == 0 {
        logger.mu.Unlock()
        return nil
    }
    out := make([]Hook, len(hooks))
    copy(out, hooks)
    logger.mu.Unlock()
    return out
}
```

That snapshot matters: it lets `fireHooks` run without holding the logger's mutex, so a hook is free to do its own logging or mutate the entry without deadlocking, and any concurrent `AddHook` won't disturb the in-flight call.

`fireHooks` then walks the snapshot and calls each hook in order:

```go
func (entry *Entry) fireHooks(hooks []Hook) {
    for _, hook := range hooks {
        if err := hook.Fire(entry); err != nil {
            _, _ = fmt.Fprintln(os.Stderr, "Failed to fire hook:", err)
            return
        }
    }
}
```

Notes on behavior:

- Hooks fire **before** the entry is formatted and written out (`newEntry.write()` runs after `fireHooks`). That is why a hook can add fields, change the message, or set the time and have those changes show up in the final output.
- If any hook returns an error, logrus prints `Failed to fire hook: …` to stderr and **stops firing the remaining hooks** for that entry. The log line itself is still written.
- Hook selection uses the entry's call-time level, so a hook that only registers for `Error` and `Fatal` will not run on `Info` calls.

## Putting it together (typical usage)

```go
log := logrus.New()
log.AddHook(myHook)            // safe, locks the logger
// or, during one-time setup:
log.Hooks.Add(myHook)

log.WithField("user", "ada").Info("hello")
// -> entry.log(InfoLevel, …)
//    -> hooksForLevel(InfoLevel) snapshots []Hook
//    -> fireHooks() calls myHook.Fire(entry) for each
//    -> entry.write() formats and writes to log.Out
```

So in one sentence: **logrus stores hooks in a `map[Level][]Hook` on each `Logger`, registration fans the hook out across the levels its `Levels()` method returns, and `Entry.log` snapshots the slice for the current level and synchronously calls each hook's `Fire(entry)` before formatting and writing the entry.**

Key files:
- `hooks.go` — `Hook` interface, `LevelHooks` type, `Add`/`Fire`.
- `logger.go` — `Logger.Hooks` field, `AddHook`, `hooksForLevel`, `ReplaceHooks`.
- `entry.go` — `Entry.log` calls `fireHooks` before `write`.
- `exported.go` — package-level `AddHook` shortcut for the standard logger.
- `example_hook_test.go` — runnable example using `log.Hooks.Add(...)`.
