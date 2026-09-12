# Tmux Resurrect

A tmux plugin (~1,200 lines of bash) that persists and restores complete tmux environments — sessions, windows, panes, layouts, working directories, and running programs. Uses tmux's own format-string introspection as a serialization protocol, producing human-readable tab-delimited save files. The process restoration engine includes a mini DSL for command matching and rewriting. Zero-config by default, extensible via strategy scripts and hooks.

## Architecture

Two-script design with shared libraries, loaded as a tmux plugin via `resurrect.tmux`:

- **`scripts/save.sh`** (278 lines) — serializes tmux state into tab-delimited text using `tmux list-panes -F`, `list-windows -F`, `list-sessions -F`. Four record types: `pane` (11 fields), `window` (8 fields), `state` (3 fields), `grouped_session` (5 fields). Pane content optionally tar-gzipped into an archive.
- **`scripts/restore.sh`** (387 lines) — reads the save file and reconstructs the tmux tree (sessions → windows → panes). Tracks existing panes in a global tab-delimited string for idempotent restore. Handles the "restore from scratch" edge case when tmux has only 1 pane.
- **`scripts/process_restore_helpers.sh`** (198 lines) — the process matching and restoration engine: parses the `~`/`->`/`*` inline strategy DSL, dispatches to per-program strategy scripts, sends restored commands via `tmux send-keys`.
- **`scripts/variables.sh`** (48 lines) — all configuration defaults and tmux option names, making the config surface explicit.
- **`scripts/helpers.sh`** (159 lines) — path management, pane content archiving, hook execution, the `remove_first_char()` utility.

The plugin entry point (`resurrect.tmux`) binds `prefix + C-s` to save and `prefix + C-r` to restore, and sets default strategies for `irb` and `mosh-client`.

## Key Techniques

**tmux format strings as serialization**: Instead of parsing tmux output, the plugin builds format strings from `#{}` variables (e.g., `#{session_name}`, `#{pane_pid}`) separated by tabs. When passed to `tmux list-panes -F`, tmux does the formatting — the bash code just reads pre-formatted lines. This is a protocol-by-convention: the introspection tool IS the serializer.

**Strategy pattern for command discovery**: `pane_full_command()` delegates to a user-configurable strategy script found at `save_command_strategies/<name>.sh`. Four strategies exist: `ps.sh` (parses `ps -ao ppid,args`), `pgrep.sh` (`pgrep -lf -P`), `linux_procfs.sh` (reads `/proc/PID/cmdline` with null-byte handling via `xargs -0 bash -c 'printf "%q "'`), and `gdb.sh` (attaches gdb to the shell, calls `write_history()`, reads the last line — extracting the actual typed command from bash's in-memory history rather than the process table).

**Inline strategy DSL for process restoration**: The `@resurrect-processes` option supports `~` (substring match), `->` (command rewrite), and `*` (argument preservation). For example, `"~rails server->rails server *"` matches any process containing "rails server", restores it as `rails server` with original arguments. This resolves the mismatch between the full path stored by `ps` and the short command the user typed.

**Idempotent restore**: `EXISTING_PANES_VAR` is a global tab-delimited string of `session:window:pane` IDs. Before creating any pane, `pane_exists()` checks tmux; existing panes are registered so their processes won't be restored either. The exception is "restore from scratch" — detected when the entire tmux server has exactly 1 pane, that lone pane gets overwritten.

**Pane content restoration via cat+exec**: Saved pane scrollback is restored by making the new pane's creation command `cat <saved_content>; exec $SHELL` — the content streams to the terminal, then the shell takes over, making it appear as if scrollback was preserved.

**Background spinner with signal trap**: `tmux_spinner.sh` runs as a background process with `trap "tmux display-message 'Done!'; exit" SIGINT SIGTERM`. The main script `kill`s it on completion — the trap fires the completion message.

**Deduplication via cmp**: `files_differ()` uses `cmp -s` to compare the new save to the previous one. If identical, the new file is deleted and the "last" symlink is not updated — repeated saves when nothing changed don't accumulate.

## Design Decisions

**Optimized for**: zero configuration, idempotency, human-readable save files, extensibility via naming-convention strategy scripts and hooks, cross-platform portability (pure bash + tmux).

**Sacrificed**: no incremental/delta saves (full snapshot each time), fragile parsing (tabs in content could break the format), no structured config (everything is tmux options), expect-based integration tests (slow, flaky), minimal error handling (silent failures, no partial-restore rollback).

**Compared to tmuxinator**: tmux-resurrect is observational rather than declarative — it captures what's running rather than requiring YAML definitions. The project includes migration docs for tmuxinator users.

**Compared to tmux-continuum** (same author): resurrect handles the save/restore mechanics; continuum layers automatic periodic saving and boot-time restore on top. Clean separation of concerns.

## Related

- [[Designing a Passively Safe API]] — idempotent operations as a safety property
- [[Idempotency Is Easy Until the Second Request Is Different]] — the edge cases resurrection's pane-tracking handles
- [[Make the Easy Change Hard]] — resurrect's strategy pattern makes extension easy by making the simple case (new program support) require a one-file change

Tags: #tool #project #terminal #tmux #bash

*Source: [github.com/tmux-plugins/tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect), fetched 2026-05-22*
