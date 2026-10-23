# PowerShell Is Actually Good

Corbin Davenport apologises to PowerShell: the 2006-era shell he'd ignored turned out to be the best tool he knows for cross-platform automation, once he gave PowerShell 7 a real project. The essay is part tool review, part indictment of Bash scripting, and a small argument for hand-crafting scripts in the age of AI.

---

## The argument

Davenport's case is that Bash scripts are fine for running commands but fall apart as *programs*: almost no built-in functions, reliance on whichever external tools happen to exist (`curl`? `wget`? both? neither?), `getopts` boilerplate for arguments, and a portability lottery when other people run your script under zsh or on a system without `sudo`. PowerShell 7 fixes all of this out of the box:

- Typed `Param` blocks — a mandatory `[uri]` parameter plus free `Get-Help` documentation replaces Bash argument-parsing boilerplate.
- Native types, hash tables, JSON conversion (`ConvertFrom-Json`/`ConvertTo-Json`), and full .NET access.
- Built-in `Invoke-WebRequest`, so no dependency on the target machine having curl or wget.
- A uniform verb-noun cmdlet grammar (`Set-Location`, `Test-Path`) that makes scripts legible to a search engine.

The design idea he's most impressed by is the **virtual drive**: PowerShell exposes the Windows Registry and certificate store as filesystems, so `Test-Path "HKLM:\SOFTWARE\Policies\Google\Chrome"` is the same operation as checking a file in Downloads. One abstraction instead of two — and you can make your own drives, "even call it `FART:\` if you want."

Two real projects back the claim up: the Windows installer for his Just the Browser project (registry checks via `Test-Path`/`Get-ItemProperty`, downloads via `Invoke-WebRequest`, .NET `List<T>` for the menu) and a Tumblr-to-Eleventy migration script mapping API responses into `PSCustomObject` metadata and JSON output. Both are jobs he'd previously have reached for Node.js for — with fewer dependencies and less `await` noise.

He's honest about the contemporary counterfactual: Claude could write these scripts. He keeps doing it himself anyway — "I'm trying my best to prevent my brain from atrophying" — using a local Gemma model only for troubleshooting when web searches fail.

---

## Key quotes

> "For anything slightly more complicated, Bash scripting is just the worst."

Commentary: the essay's real thesis. The Bash-vs-PowerShell comparison isn't shell aesthetics; it's about whether the scripting environment ships a standard library. Bash's dependence on ambient POSIX tools is a portability tax paid on every script that has to survive on someone else's machine.

> "You don't have to learn how to use files *and* how to use registry keys, because *they're the same thing* in PowerShell."

Commentary: the strongest technical point in the piece. A uniform filesystem abstraction over heterogeneous resources is the same unifying move that made Unix pipes great — Microsoft applied it to the Registry, certificate store, and user-defined drives. This is the design insight worth stealing regardless of platform.

> "However, I like learning things, and I'm trying my best to prevent my brain from atrophying."

Commentary: the quiet thesis underneath the tool review. Coming from the author who uses yt-dlp, Eleventy, and Homebrew, this is a deliberate stance against full delegation — the same position the wiki's hand-craft essays take, but stated without moralising. Notably, his AI use is properly scoped: a *local* model, for troubleshooting, when web search fails — tool, not wheel.

> "It can go right next to my 1996 paperback edition of O'Reilly's *HTML: The Definitive Guide*."

Commentary: a closing joke that lands the essay's frame — learning a tool deeply in 2026 is itself a retro choice, and he knows it.

---

## Themes

#tool #concept #pattern

- **Standard library as the differentiator** — the gap between Bash and PowerShell isn't syntax, it's what ships with the shell.
- **Uniform abstractions over messy resources** — virtual drives as the Registry-as-filesystem pattern.
- **Scripting as craft** — hand-writing automation as resistance to cognitive offloading, with AI used narrowly for troubleshooting.

---

## Opinionated take

This is a better argument for PowerShell than anything Microsoft ever wrote, precisely because it comes from a Linux-and-Mac user who didn't want to be convinced. The essay also indirectly indicts the modern dependency culture it avoids: the Node.js ecosystem's answer to "download a file and unzip it" is two npm packages and an audit anxiety, while PowerShell answers with two cmdlets. That's a fair point, though the essay undersells PowerShell's own rough edges — the Windows PowerShell 5.1 / PowerShell 7 split it has to document ("Windows 11 still only includes Windows PowerShell") is exactly the kind of confusing versioning it criticises Bash for, and nobody's first week with the verb-noun grammar is pleasant.

The more interesting tension is with the agentic era: scripts like these are precisely the kind of small automation an LLM now generates in seconds. Davenport's answer — that writing them is the point, not the artifact — is the same reasoning the wiki's craft-and-deskilling essays keep circling, arriving from a different direction: if the AI writes your automation, you own none of the knowledge that made the task legible.

## Related pages

This essay extends [[The Day of a New Command-Line Interface Shell]] by offering the counter-case in the shell-design conversation: that essay asks what a *new* command-line interface should look like, while Davenport argues an unfashionable 2006 design already solved portability and standard-library problems Bash never did. It nuances [[Command Line Interface Guidelines]] with a worked example of discoverable, self-documenting CLI conventions (`Get-Help` over any script) — the guidelines' principles embodied in a shell. And it complicates [[Vibe Coding and the Maker Movement]] and [[Slowing the Fuck Down]] from the tool-choice direction: here the resistance to delegation isn't about quality of output but about preserving the maker's own competence, with AI deliberately scoped to a local troubleshooting assistant.

---
*Sources: [[raw/powershell-is-actually-good]], [[summary/powershell-is-actually-good]]*
*Last updated: 2026-10-24*
