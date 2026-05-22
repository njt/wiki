---
url: https://github.com/tmux-plugins/tmux-resurrect
title: Tmux Resurrect
author: tmux-plugins (Bruno Sutic)
date_fetched: 2026-05-22
date_published: 2014
---

# Tmux Resurrect — Full Analysis

A tmux plugin that saves and restores the complete tmux environment: sessions, windows, panes, layouts, working directories, and running programs. ~1,200 lines of bash across 14 source files, plus expect-based integration tests.

## Architecture

The project is a **tmux plugin** loaded via `resurrect.tmux` (the plugin entry point). It follows a clean **two-script architecture**:

- `scripts/save.sh` — serializes the current tmux state into a tab-delimited text file
- `scripts/restore.sh` — reads that file and reconstructs the environment

Both are sourced with shared libraries:
- `scripts/variables.sh` — all configuration defaults and tmux option names
- `scripts/helpers.sh` — shared utilities (path management, pane content archiving, hooks)
- `scripts/spinner_helpers.sh` + `scripts/tmux_spinner.sh` — background spinner process for UX
- `scripts/process_restore_helpers.sh` — the process matching and restoration engine

### File tree

```
resurrect.tmux                    # Plugin entry point (40 lines)
scripts/
  save.sh                         # Save logic (278 lines)
  restore.sh                      # Restore logic (387 lines)
  helpers.sh                      # Shared utilities (159 lines)
  variables.sh                    # Configuration defaults (48 lines)
  process_restore_helpers.sh      # Process restoration engine (198 lines)
  check_tmux_version.sh           # Version check (78 lines)
  tmux_spinner.sh                 # Background spinner (29 lines)
  spinner_helpers.sh              # Spinner start/stop wrappers (8 lines)
  restore.exp                     # Expect script for automated restore
save_command_strategies/
  ps.sh                           # Default: ps-based command discovery
  pgrep.sh                        # Alternative: pgrep-based
  linux_procfs.sh                 # Linux-specific: /proc/PID/cmdline
  gdb.sh                          # GDB-based bash history extraction
strategies/
  vim_session.sh                  # Restore vim with Session.vim
  nvim_session.sh                 # Restore nvim with Session.vim
  irb_default_strategy.sh         # Strip env vars from irb command
  mosh-client_default_strategy.sh # Reconstruct mosh command from mosh-client
tests/
  test_resurrect_save.sh          # Save integration test
  test_resurrect_restore.sh       # Restore integration test
  helpers/                        # Expect-based test infrastructure
  fixtures/                       # Expected save/restore file contents
```

### Data model

The core data structure is a **tab-delimited flat file** with four record types:

1. **pane** — `pane\t<session>\t<window>\t<window_active>\t<flags>\t<pane_idx>\t<title>\t:dir\t<pane_active>\t<short_cmd>\t:<full_cmd>`
2. **window** — `window\t<session>\t<window>\t:name\t<active>\t:<flags>\t<layout>\t<automatic_rename>`
3. **state** — `state\t<client_session>\t<client_last_session>`
4. **grouped_session** — `grouped_session\t<name>\t<original>\t:alt_win\t:active_win`

Values prefixed with `:` are stripped by `remove_first_char()` on restore — this handles the tmux format quirk where values sometimes have a leading colon.

## Key Techniques

### 1. tmux format strings as a serialization protocol

Instead of parsing tmux's structured output, the plugin uses tmux's own `-F` format flag as the serialization layer. Each `*_format()` function builds a format string from `#{}` tmux variables separated by the tab delimiter. For example:

```bash
pane_format() {
    format+="pane"
    format+="${delimiter}"
    format+="#{session_name}"
    format+="${delimiter}"
    format+="#{window_index}"
    # ... 11 fields total
}
```

`tmux list-panes -a -F "$(pane_format)"` then produces perfectly formatted tab-delimited records. This is clever because it offloads all the parsing work to tmux itself — the bash code just reads pre-formatted lines.

### 2. Strategy pattern for command discovery

The save process needs the *full* command running in each pane (not just the short name tmux provides). The `pane_full_command()` function delegates to a configurable strategy script:

```bash
_save_command_strategy_file() {
    local save_command_strategy="$(get_tmux_option "$save_command_strategy_option" "$default_save_command_strategy")"
    local strategy_file="$CURRENT_DIR/../save_command_strategies/${save_command_strategy}.sh"
    # falls back to default if specified strategy file doesn't exist
}
```

Four strategies exist:
- `ps.sh` (default) — `ps -ao "ppid,args"` filtered by PID
- `pgrep.sh` — `pgrep -lf -P $PID`
- `linux_procfs.sh` — reads `/proc/PID/cmdline` with proper null-byte handling via `xargs -0 bash -c 'printf "%q "'`
- `gdb.sh` — attaches gdb to the shell process, calls `write_history()`, reads the last line. This is the most creative — it extracts the actual bash command from the shell's in-memory history, not from the process table

### 3. Inline strategy DSL for process restoration

The `@resurrect-processes` option supports a mini language parsed in `process_restore_helpers.sh`:

- `~` prefix — substring match instead of exact prefix match
- `->` — inline command rewrite (saved command → displayed command)
- `*` token — preserve original command arguments

Example: `"~rails server->rails server *"` means:
1. Match any process containing "rails server" anywhere in the string
2. On restore, send `rails server` plus the original arguments

The parsing separates on `->`, then checks for `*` in the right side:

```bash
_get_proc_match_element() { echo "$1" | sed "s/${inline_strategy_token}.*//"; }
_get_proc_restore_element() { echo "$1" | sed "s/.*${inline_strategy_token}//"; }
```

### 4. Idempotent restore via existing-pane tracking

On restore, panes that already exist are registered in a global tab-delimited string:

```bash
register_existing_pane() {
    EXISTING_PANES_VAR="${EXISTING_PANES_VAR}${delimiter}${pane_custom_id}"
}
is_pane_registered_as_existing() {
    [[ "$EXISTING_PANES_VAR" =~ "$pane_custom_id" ]]
}
```

This prevents restoring processes into panes that were already open before the restore — making repeated restore safe. The "restore from scratch" mode (when tmux has only 1 pane total) is the exception: it overwrites that lone pane.

### 5. Pane content capture and archive

Pane text content is captured via `tmux capture-pane`, saved to individual files keyed by pane ID, then tar-gzipped:

```bash
pane_contents_create_archive() {
    tar cf - -C "$(resurrect_dir)/save/" ./pane_contents/ | gzip > "$(pane_contents_archive_file)"
}
```

On restore, the content is piped into a `cat` command executed when the pane is created:

```bash
pane_creation_command() {
    echo "cat '$(pane_contents_file "restore" "${1}:${2}.${3}")'; exec $(tmux_default_command)"
}
```

This is a clever trick: the pane starts by dumping the saved content to the terminal via `cat`, then `exec`s the default shell — making it appear as if the old scrollback was restored.

### 6. Background spinner via signal trap

The spinner runs as a background process with a `trap` on SIGINT/SIGTERM:

```bash
trap "tmux display-message '$END_MESSAGE'; exit" SIGINT SIGTERM
```

The save/restore scripts start it with `start_spinner "Saving..." "Tmux environment saved!" &`, do their work, then `kill $SPINNER_PID` — the trap fires and displays the completion message.

### 7. Deduplication via file comparison

Before writing a save file, the plugin compares it to the last save using `cmp -s`:

```bash
files_differ() { ! cmp -s "$1" "$2"; }
# ...
if files_differ "$resurrect_file_path" "$last_resurrect_file"; then
    ln -fs "$(basename "$resurrect_file_path")" "$last_resurrect_file"
else
    rm "$resurrect_file_path"  # nothing changed, discard
fi
```

This means repeated saves when nothing changed don't accumulate files or update the "last" symlink timestamp.

## Design Decisions

### What they optimized for
1. **Zero configuration** — works out of the box with sensible defaults
2. **Idempotency** — restore is safe to run multiple times
3. **Human-readable save files** — tab-delimited text you can grep and inspect
4. **Extensibility** — strategies are separate scripts found by naming convention, hooks allow arbitrary commands
5. **Portability** — pure bash + tmux, works on Linux, macOS, Cygwin

### What they sacrificed
1. **No incremental saves** — each save is a full snapshot, no delta encoding
2. **Fragile parsing** — tab-delimited format means tabs in window names or commands could break things (mitigated by the `printf '%s\n'` trick for pane content)
3. **No structured config format** — all config is tmux options (`set -g @resurrect-...`), which is idiomatic for tmux but limits complex configuration
4. **Expect-based testing** — the tests use expect scripts to spawn tmux, which is slow and flaky compared to unit tests
5. **Limited error handling** — most failures are silent; there's no cleanup on partial restore failure

### Non-obvious architectural insight
The `restore.exp` expect script reveals a hard problem: automated restore. The script spawns a tmux, sends the restore command, then `sleep 100` — a brute-force wait because there's no way to signal that restore is complete when you're not attached. The companion plugin `tmux-continuum` solves this by using tmux hooks.

## Comparison Notes

Unlike **tmux-resurrect**, which is a manual save/restore tool, **tmux-continuum** (by the same author) layers automatic periodic saving and boot-time restore on top. This is a clean separation: resurrect does the mechanics, continuum does the automation.

Unlike **tmuxinator** (a Ruby-based tmux layout manager that defines sessions in YAML), resurrect is *observational* rather than *declarative* — it captures what's actually running rather than requiring you to describe what should run. The project includes migration docs from tmuxinator.

The approach of using tmux's own formatting as a serialization layer is reminiscent of how `jq` uses its own query language as an output formatter — the tool's introspection becomes the data protocol, avoiding the need for a separate parser or schema.
