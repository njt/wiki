---
url: https://github.com/bobek-balinek/claude-lamp
date_fetched: 2026-07-05
backfilled: true
---

Control your Moonside LED lamp via BLE based on Claude Code's state. Your lamp becomes a physical status indicator — animated themes while Claude works, green when idle, purple when it needs your input.


WARNINGAuthor takes no responsibility for the hardware issues that may arise from using this script. You run these scripts at your own risk.

**NOTE** The initial connection handshake with the lamp might take few seconds, please tail the daemon logs to check it all works fine.

| State | Lamp | Trigger | 
|---|---|---|
| Working | BEAT2 theme (white/navy) | Prompt submit, tool use | 
| Idle | Solid sunset mango | Claude finishes responding, session start | 
| Needs input | Solid purple | Permission request, plan approval, question, notification | 
| Off | LED off | Session end | 

- macOS (BLE via CoreBluetooth)
- Python 3.10+
- bleak (`pip install bleak`)
- Claude Code
- A Moonside lamp (tested with Halo — should work with One, Aurora, Lighthouse etc.)

`pip install bleak````
mkdir -p ~/.claude/moonside_hooks
cp claude_hooks/moonside_hook.sh claude_hooks/moonside_daemon.py ~/.claude/moonside_hooks/
chmod +x ~/.claude/moonside_hooks/moonside_hook.sh
```
Putting them in `~/.claude/moonside_hooks/` means they work across all projects — no need to have `claude_hooks/` in every repo.

Merge `claude_hooks/settings.json` into `~/.claude/settings.json`. The paths in the config use `$CLAUDE_PROJECT_DIR` — update them to point to `~/.claude/moonside_hooks/` instead:

```
sed 's|\$CLAUDE_PROJECT_DIR/claude_hooks|~/.claude/moonside_hooks|g' \
  claude_hooks/settings.json
```
Or just copy the JSON and replace the paths manually. The config hooks into `SessionStart`, `UserPromptSubmit`, `Stop`, `PreToolUse` (all tools), `PostToolUse`, `PermissionRequest`, `Notification`, and `SessionEnd`.

The daemon includes a debounce to prevent phantom working transitions (e.g. internal prompt suggestions firing PreToolUse shortly after Stop).

Open a new Claude Code session. The daemon auto-discovers your lamp by name — no address configuration needed.


Multiple lamps?Set`MOONSIDE_MAC`to pin a specific device. Run`python3 moonside_ble.py scan`to list devices. On macOS, addresses are UUIDs (not MAC addresses).

```
Claude Code hook event
  → moonside_hook.sh (writes state to /tmp/moonside_state, launches daemon if needed)
    → moonside_daemon.py (persistent BLE connection, reads state file every 200ms)
      → Moonside lamp via BLE (Nordic UART Service)
```
The daemon keeps a persistent BLE connection to avoid 2-5s reconnect latency on every hook event. It runs in the background and auto-exits after 30 minutes of idle or on `SessionEnd`.

| File | Purpose | 
|---|---|
| `claude_hooks/moonside_hook.sh` | Shell hook called by Claude Code. Writes state, starts daemon if needed. Always exits 0. | 
| `claude_hooks/moonside_daemon.py` | Background daemon with persistent BLE connection, state machine, and idle→working debounce. | 
| `claude_hooks/settings.json` | Ready-to-use Claude Code hooks config. Copy/merge into `~/.claude/settings.json`. | 
| `moonside_ble.py` | Standalone BLE controller for Moonside lamps. Usable directly from the command line. | 

- **PID file:**- `/tmp/moonside_daemon.pid`
- **State file:**- `/tmp/moonside_state`
- **Log file:**- `/tmp/moonside_daemon.log`

`moonside_ble.py` can be used independently of Claude Code:

```
python3 moonside_ble.py scan                     # find devices
python3 moonside_ble.py on                        # turn on
python3 moonside_ble.py off                       # turn off
python3 moonside_ble.py color 255 0 128           # set color
python3 moonside_ble.py color 255 0 128 --brightness 80
python3 moonside_ble.py theme rainbow3            # activate theme
python3 moonside_ble.py theme fire2 --colors 255,50,0
python3 moonside_ble.py raw "THEME.GRADIENT1.255,0,0,0,0,255"
python3 moonside_ble.py interactive               # REPL mode
```
**Lamp not responding:**

```
# Check daemon log
cat /tmp/moonside_daemon.log
# Verify BLE connection works
python3 moonside_ble.py on
```
**Daemon stuck:**

```
kill "$(cat /tmp/moonside_daemon.pid)"
rm -f /tmp/moonside_daemon.pid /tmp/moonside_state
```
**bleak not found:**
The hook auto-detects python from `python3`, `/opt/homebrew/bin/python3`, and `$CONDA_PREFIX/bin/python3`. Make sure one of them has bleak installed.

| State | Visual | 
|---|---|
| Working | BEAT2 theme (white + navy) | 
| Idle | Solid sunset mango (255, 180, 50) | 
| Input | Solid purple (200, 0, 255) | 

Colors are configured in `moonside_daemon.py`.

Moonside lamps use the Nordic UART Service (NUS) over BLE. Commands are ASCII text:

| Command | Format | Example | 
|---|---|---|
| LED on/off | `LEDON`/`LEDOFF` | `LEDOFF` | 
| Color (0-255) | `COLORRRRGGGBBB` | `COLOR000255000` | 
| Brightness (0-120) | `BRIGHBBB` | `BRIGH060` | 
| Theme | `THEME.NAME.R,G,B,...` | `THEME.FIRE2.255,50,0` | 

MIT
