---
url: https://agent-safehouse.dev/
date_fetched: 2026-11-26
---

# Go full `--yolo`. We've got you.

macOS-native sandboxing for local agents. Move fast, break nothing.

`--yolo`. We've got you.macOS-native sandboxing for local agents. Move fast, break nothing.

 LLMs are probabilistic - 1% chance of disaster makes it a matter of *when*, not *if*. 

`rm -rf ~`All agents work perfectly in their sandboxes, but can't impact anything outside it.

Agents inherit your full user permissions. Safehouse flips this — nothing is accessible unless explicitly granted.

Install with Homebrew or download the single shell script, then run your agent inside it. No build step, no dependencies beyond Bash and macOS.

```
# 1. Install with Homebrew
brew install eugene1g/safehouse/agent-safehouse
# Or download the single self-contained script
mkdir -p ~/.local/bin
curl -fsSL https://github.com/eugene1g/agent-safehouse/releases/latest/download/safehouse.sh \
  -o ~/.local/bin/safehouse
chmod +x ~/.local/bin/safehouse
# 2. Run any agent inside Safehouse
cd ~/projects/my-app
safehouse claude --dangerously-skip-permissions
```
Safehouse automatically grants read/write access to the selected workdir (git root by default) and read access to your installed toolchains. Most of your home directory — SSH keys, other repos, personal files — is denied by the kernel.

Try reading something sensitive inside safehouse. The kernel blocks it before the process ever sees the data.

```
# Try to read your SSH private key — denied by the kernel
safehouse cat ~/.ssh/id_ed25519
# cat: /Users/you/.ssh/id_ed25519: Operation not permitted
# Try to list another repo — invisible
safehouse ls ~/other-project
# ls: /Users/you/other-project: Operation not permitted
# But your current project works fine
safehouse ls .
# README.md  src/  package.json  ...
```
Add these to your shell config and every agent runs inside Safehouse automatically — you don't have to remember. To run without the sandbox, use `command claude` to bypass the function.

**POSIX shells ( zsh / bash)**

```
# ~/.zshrc or ~/.bashrc
safe() { safehouse --add-dirs-ro=~/mywork "$@"; }
# Sandboxed — the default. Just type the command name.
claude()   { safe claude --dangerously-skip-permissions "$@"; }
codex()    { safe codex --dangerously-bypass-approvals-and-sandbox "$@"; }
amp()      { safe amp --dangerously-allow-all "$@"; }
gemini()   { NO_BROWSER=true safe gemini --yolo "$@"; }
# Unsandboxed — bypass the function with `command`
# command claude               — plain interactive session
```
`fish`

```
# ~/.config/fish/config.fish
function safe
    safehouse --add-dirs-ro="$HOME/mywork" $argv
end
# Sandboxed helpers without overriding the original binary names.
function sandbox-claude
    safe claude --dangerously-skip-permissions $argv
end
function sandbox-codex
    safe codex --dangerously-bypass-approvals-and-sandbox $argv
end
function sandbox-amp
    safe amp --dangerously-allow-all $argv
end
function sandbox-gemini
    set -lx NO_BROWSER true
    safe gemini --yolo $argv
end
```
Use a ready-made prompt that tells Claude, Codex, Gemini, or another model to inspect the real Safehouse profile templates, ask about your home directory and toolchain, and generate a least-privilege `sandbox-exec` profile for your setup.

The guide also tells the LLM to ask about global dotfiles, suggest a durable profile path like `~/.config/sandbox-exec.profile`, offer a wrapper that grants the current working directory, and add shell shortcuts for your preferred agents.
