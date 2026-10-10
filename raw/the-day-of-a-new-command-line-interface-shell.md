---
url: https://arcan-fe.com/2022/04/02/the-day-of-a-new-command-line-interface-shell/
date_fetched: 2026-10-10
---

This article continues the long-lost series on how to migrate away from terminal protocols as the main building block for command-line and text-dominant user interfaces. The previous ones (Chasing the dream of a terminal-free CLI (frustration/idea, 2016) and Dawn of a new Command-Line Interface (design, 2017)) might be worth an extra read afterwards, but they are not prerequisites to understanding this one. The practical demonstration to this can be found in Whipping up a new Shell – Lash#Cat9 and the series was concluded with Sunsetting Cursed Terminal Emulation.

The value proposition and motivation is still that such a critical part of computing should not be limited to device restrictions set in place some 50-70 years ago. The resulting machinery is inefficient, complex, unreliable, slow and incapable. For what is arguably a strong raison d’être for current day UNIX derivates, that is not a strategic foundation to neither rely nor expand upon.

The focus this time is about the practicalities of the user facing ‘shell’ — the cancerous and confused mass that hides behind the seemingly harmless command-line *prompt*. The final article will be about the developer facing programming interfaces themselves as application building blocks, how all of this is put together, and the design considerations that go into such a thing.

This article is structured as follows:

- What is ‘Shell’ – gives a short primer about the specific role the CLI ‘shell’ plays.
- ‘Simplifying and Exemplifying’ – shows how the current stack is reworked.
- ‘Gains’ – goes into the capabilities that new shells can take advantage off.

The following clip is a very quick teaser from using one in-progress replacement shell that has been built using the tools that will be covered here. While the shell itself will be presented in greater detail in another future article, it is available for adventurous souls to poke around with [and can be found in this GH repository].

Starting with other/related work; many have attempted to deal with the embarrassing legacy of terminals and their inherent limitations.

“NOTTY” focused on replacing the “in-band” signalling and command format. This addresses some of the protocol issues, but has an impedance mismatch with what the rest of your desktop or basic rendering expects, and the emulator-shell split remains.

Some, like “Hyper” and “Upterm“, rewrites the terminal emulator and shell in more and more advanced UI frameworks to get better cooperation with an outer graphical shell — inviting in all the complexities of rancid behemoths like Electron and GTK/Qt while still leaving the protocols and TUI libraries in their currently poor state.

Others, like “Notcurses et al.” replaces the key TUI libraries like Curses, Readline and so on. These fix *neither* the emulator *nor* the protocols. Worse still, a few make the protocol situation *worse* by introducing sidebands, hard-coding escape sequences or introducing new ones.

Then there are a number of attempts like jc and relational-pipes that proxy or modifying the exchange format between stdin/stdout in a single pipeline, but that is mostly orthogonal to the problem discussed; solving for the others would provide another pathway for negotiating *multiple*, concurrent, exchange formats.

## What is ‘S*hell*‘?

First, if you want better and deeper reading into the subject, I would suggest:

- (Oil)Why a new Shell – Disambiguation of programming interface versus user interface roles.
- Terminal Vision – Verbose walkthrough of the entire space.
- Windows Command-Line: Background – Condensed walkthrough with MSWIN bias.

There are many more to be had, but piling them on would mainly add to the existing confusion between terms (console, shell, terminal, tui, gui, ..); these terms have contextual and historical interpretations that are slightly incompatible depending on where you are and where you come from, which makes discussing the topic even harder.

Here is a rough breakdown of different components and roles sufficient for the scope of this article:

Here, ‘Shell’ (as part of providing a textual shell as a command-line) is the first in line to consume and work with a terminal emulator, through a preassigned set of file descriptors (0, 1, 2) mapped to a terminal or pseudo-terminal device. Shells and some applications alike test and change their behaviour depending on the state of these descriptors (isatty(3) to tcgetattr(3) to ioctl(2)), sometimes referred to as an ‘interactive’ mode. These continue (unless explicitly told not to) to share and inherit into new jobs over this serial communication line.

The protocol/IPC marked blocks mask quite a bit of nuance as to how data exchange work and they are not created equal; the *sockets* used to communicate with the display server may be unpleasant, but are still infinitely better than this mix of “tty” devices, signalling groups, sessions and stdio used after the ‘terminal emulator’ stage — you do have to sit down and implement *both* consumer and producer side of the terminal instruction sets to get a fair grasp on just how bad things are. For the Linux kernel alone, the TTY layer is one that not even the most seasoned of developers wants to touch.

While the ’emulator’ part is often stripped and just referred to as ‘the terminal’ it *is* very much an emulator of ancient hardware (or rather the amalgamation of tens to hundreds of different ones). That fact should be stressed to emphasise the absurdity of it all — especially given the end goal of reading key presses and writing characters into a grid of cells. 

There is valuable simplicity to TUIs (out of which CLIs are but one possibility), but that simplicity is wholly undone by the complexity of terminals and how the ‘instruction set / device model’ they expose makes the shell user experience itself unnecessarily hard to develop and provide.

It should be emphasised that the *terminal emulator is also a poor take on a display server*. This will become relevant later on. As such, it is at a disadvantage against better display servers for many reasons – one being that each job/client is not given a distinct bidirectional connection for data exchange, but instead share a single triplet of “files” (stdin, stdout, stderr), combined with a protocol that was never designed or intended for this.

This shared triplet as well as ‘multiplexing’ is important. Say that one of the shell-launched jobs is another cli shell itself, like gdb or glorious ed. Since the data is in-band over the shared set of stdio slots mapped to the kernel provided device, the previous shell(s) either needs to be full emulators on their own, or it they cannot *safely* intervene or layer other things on top of whatever the job is doing.

Even then it has few options for *reliably* restoring the emulated device state. This is why accidentally cat:ing something like /dev/random will quickly give you a screwed up prompt; it is likely that some of the many sequences that change character map, cursor or flow control was triggered – yet if the shell continues on unawares, the scroll-back history is forever tainted.

There are certainly was to hardcode and reset some state between jobs explicitly – and some shells do – but that also serve to mask the danger and fundamental issue with the design; it is executing random instructions in a complex and varying instruction set.

Back to the model. The ‘graphical shell’ and ‘textual shell’ refer to the abstraction the user actually interacts with. The other ‘shell’ (as in bash, zsh, …) serves at least two roles. First you have a primary role as a ‘window manager’ of sorts. This provides the “prompt”, parsing the command-line into ‘built-in’ command execution; constructing processing pipelines or executing ‘fullscreen’ applications and choosing which pipeline that is currently being “presented”.

The other thing it provides is a scriptable programming environment (as in the *scripting* part of shell-scripting). This is a secondary feature at best, and not at all necessary. In a command-line environment free from the legacy of terminals, current shells can continue to play this specific role — even if that is a job that should be left to more competent designs.

For the ‘window manager’ role: these range from (visually) simple ones like the foreground/background of bash, zsh and fish to more complex such as tmux and screen (sometimes referred to as multiplexers). The first ones tend to focus on how to articulate jobs and their data exchange, while the second on tiling like window management.

In order for these more complex ones to achieve window management, they also went through the strides of writing additional terminal emulators and embedding other shells (recursive premature composition) in order to output to yet another terminal emulator as there is no proper handover or embedding mechanism in place. It is terminal emulators all the way down and a reason why we have to talk about how shell is complicit in all of this – even the basics of what is expected from ‘reading a line’ requires dipping into terminal drawing, cursor and flow control commands.

This division also reflects the ‘modes’ of how the terminal protocols operate, which, in turn, tie back to what the computer output device actually was in various parts of the timeline. Recall that once upon a time the output was a printer (“line-based”) and only later became monitors (“screen-based”) with incrementally added luxuries like colour and interesting tangents like vector graphics. Moving the cursor around arbitrarily back across previous lines is a privilege — not a right.

You can see some of this bleed through with ‘scroll-back/history’ working poorly (or not at all) in the screen mode, with “tab/context” suggestion popups causing scrolling and weird wrapping visuals when the prompt is at the edge of the last ‘line’ on the ‘paper’ or – heavens forbid – you try to erase previous characters across pages or newlines.

If you want the worst of both worlds, go no further than regular ‘gdb’ (as in the GNU debugger) and go back and forth between ‘tui enable’, for the luxury of seeing the source code you are debugging at the cost of scrolling back through the data you needed, and ‘tui disable’ where every ephemerally relevant output gets committed to the ‘paper’ and the data you needed quickly scroll off into the far away distance.

You can also see it in the ‘tab completion’ output in a line-mode shell having to ‘add lines’ in order to fill in the completion, and those are kept there, polluting the history — as well as in the special treatment certain characters like ‘erase’ received. The man page to ‘stty‘ (or worse still, how a tty driver is written) is a brief yet still frightening look into the special properties of the underlying device itself.

For both modes, the protocols restrict what these two kinds of text-dominant shells can do and how they can cooperate with an outer graphical one. In the model presented so far, there is zero real cooperation between the text and the graphics shells. In reality, there is some, but implemented in a near impenetrable soup of hacks involving a forest of possible sideband protocols — and availability vary wildly with your choice of emulator, the protocol set it is defined to follow, and the contents of a capability database (terminfo/termcap).

As an exercise for the reader, try to work out how and why you can- or why you cannot-

- Paste a block of text from your desktop clipboard into the current command-line.
- Drag and drop a file into your shell and have it stored into the current working directory.
- Click a URL in the command-line buffer history and have it open in your browser.
- Redirect the output of a previous command to another window or tab.
- Fold / unfold the presentation of output from previous commands.
- Have an accurate clock in your prompt that updates by itself.

Neither of these are particularly exotic use-cases, some would even go so far as to say that these are fairly obvious things that should be trivial to support — yet if you think the answers to any of these are simple and easy, you missed something; there is a Lovecraftian horror hiding behind each and every one.

## Simplifying and Exemplifying

Using the model from the previous section, we restructure it to this:

The terminal emulator is *gone*, the rightfully maligned ‘tty’ layer hiding in the kernel is *gone*. There are now a whole lot of ways for the graphical shell to *cooperate* with the textual one.

For this to work and provide enough gains, a lot of subtle nuances of the IPC system need to be in place; the one in Arcan (shmif) was specifically designed for this as one of several ‘grand challenges’ that were used to derive the intended feature set many, many, many years ago.

One of the main building blocks is ‘handover allocation’ – where the shell requests new resources in the  graphical shell on behalf of an upcoming job, and then *forwards* the primitives needed to inherit a connection into the job, *retaining* the *chain of trust* and custody. Another is the live migration used as part of crash resilience, which eliminates the need for multiplexers as each client can redirect at runtime to other servers by design.

The main Arcan process takes the role of the display server. With that comes a pick of graphical shell, ranging from the modest ‘console‘ to the more advanced (durden, pipeworld, safespaces). Do note that there is a choice in building Arcan as the system display server with authority on GPUs, input devices and so on – *or* as a regular graphical client that you would run in place of your terminal emulator inside Xorg or some Wayland compositor. You will lose out on several performance gains, and some nuances in how window management integrates, but many features will remain.

For the ‘text shell’ block, there is a little bit more to think about. While it is perfectly valid and intended to use *libarcan-tui* to write your own here, one also comes included in the box. A regular Arcan build produces ‘afsrv_terminal’ (or arcterm as it is referred to internally).

This is a terminal emulator with a secret; if the argument “cli” is passed, it switches to an extremely barebones built-in text-shell and skips all the terminal emulation machinery. It is intended to provide only the absolutely necessary bits for something like booting a recovery image for an OS. If you are C inclined, this is a fair basis to expand on or borrow from.

In the following clip from the Pipeworld article (a graphical shell), you can see it in use in the form of the small CLI cell where I am typing in commands.

While things go quite fast, you might be able to spot how it transitions from a command-line as part of the *graphical* shell at 0:02 into a terminal emulator liberated *textual* CLI shell. You can then see that the jobs which spawn are their own separate processes and do not multiplex over the same pseudo-terminal devices (as that would make more than one ‘tui’ like job impossible or require nesting composition/state through something like screen or tmux, reintroducing the premature composition problem).

The twist is that these jobs are *negotiated* with the graphical shell being *aware* of their *purpose* and *origin*. This is a feature that runs deep and dates far back, already in use at the time of the One Night in Rio – Vacation Photos from Plan9 article. It is also the reason why jobs spawn as new detachable windows, yet retain hierarchy in the optics of the window management scheme.

Another setup can be found in this clip, also from Pipeworld:

Here we demonstrate how a processing pipeline can be built with separate outputs for each task in a pipeline, while at the same using stdin/stdout to convey the data that is to be processed. Any single one of these can be a strict text client, an arcan-tui one or wrapped around a terminal protocol decoder – yet both interacted with- and tracked- independently.

The ‘cli’ mode takes another argument, =lua. This enables a Lua VM, maps in API bindings, loads a basic script harness that provides some very crude and basic commands, but allows for plugging in a custom shell, like the one mentioned at the beginning of the article.

In this clip we can see a prompt from that shell where we run a job, and popup the ongoing results from that job into a window of its own with a hex view. The graphical shell, here operating in a tiling window manager setup, respects the request for this window to be a tab to the current one and creates it as such.

To add legacy to injury, this clip shows running a new job as a separate vertical-split window, wrapped around a terminal emulator. The standard error output, however, gets tracked and mapped into the shell view of ongoing jobs. Paste actions from the graphical shell has been set to accumulate into the data buffer of a job. This feature disabled and the paste action instead copies into the readline completion set, inserting at the cursor if activated.

In this clip we go even further – the shell opens a media resource, requests it to be embedded into its window. The resource scales, repositions and folds accordingly, yet the user can ‘drag it out’ should she so desire. The video playback in this case is delegated to one-shot dedicated processes, no parsing or exotic dependencies are imposed on the shell process itself.

The process responsible for composition gets to composite and gives the user independent controls for lossless decomposition.

In the following clip we see other forms of metadata interaction – the shell requests that the user picks a file, which it then redirects into a local copy inside the current working directory. The file picking it outsourced to whatever an outer graphical shell provides, and the chosen descriptor is forwarded into the text shell process that then saves it to disk. The process is repeated by picking an image file that is then opened and embedded similarly to the previous clip. Had the textual shell been running remotely or in some distant container, the transfer would have gone through just the same. The underlying mechanism works for explicit load-store, cut and paste as well as drag and drop.

## Gains

There is much more to be had than the parlour tricks shown in the previous section. What can, immediately, without rose-tinted glasses and speculation, be gained by leveraging this infrastructure?

** Data Communication** – With an actual IPC system to connect through, it is possible to:

- Leave STDIN/STDOUT/STDERR as pure data channels, not mixing in UI events or draw commands.
- Accidentally catting a binary file or device cannot break the UI state machine.
- Explicit serialisation of state (store / restore runtime config) without filesystem trails.
- Every single command in a pipeline is left alone and kept separable between jobs *and*they do not interfere with shell communication. Thus each tool in a pipeline can provide both in-stream processing*and*an interactive user interface at the same time.
- All interfaces strongly encourage asynchronous processing.
- Binary blob transfers pass as file descriptors locally, and scheduled/multiplexed over the network.

** Input – **Having an event model that is not limited to a range of reserved values in the ASCII table delivered over a pipe allows:

- Non-ambiguity – there is a discernible difference between pressing the ‘ESCAPE’ key and the ESC-ASCII character that was used to mark the beginning of an escape sequence.
- Modifiers exist, CTRL+C is a symbolic C key with CTRL modifier, that does not equal ^C or it being magically translated to broadcasting SIGINT.
- Pasting is separated from entering text, is separated from pressing keys, and can undo as a discrete whole.
- Mouse input is predictable and reliable and can be combined with keyboard modifiers.
- Keyboard shortcuts are announced and semantically tagged, letting the graphical shell provide automation, rebinding and mapping to assistive devices.

** Integration: **With the same language for expressing graphical clients as for textual ones:

- Clients can be redirected between shells at runtime, even across a network connection.
- Graphical shell capabilities can be leveraged for universal file picking.
- Decorations like borders and scrollbar are deferred to the outer graphical shell, avoiding mixing data with metadata in the grid by drawing ‘line characters’.
- If the graphical shell is sufficiently capable — notifications, alerts, file picking and popups become available and behave according to the rules of the graphical shell.

*Visuals and Performance:*

The rendering responsibilities have been moved to the display server end of the equation, while the fonts currently in use are being passed as reference objects for features that need it (ligatures, …). There are no pixel buffers being passed around from the ’emulator’ client and the shell is explicit about when it is time to synchronise content onwards.

- Tear-free updates and resizing.
- Presentation buffer back-pressure control is deferred to the job, no more heuristics in the emulator.
- Colours are always 24-bit or from a semantic palette (no more “red” is now “green”).
- Embeddable interactive media content (* assuming an outer graphics shell support it).
- Synchronised presentation, atomic commit of change sets and only updates between sets are synched.
- Glyph caches can be shared between multiple shell instances and other clients.
- Glyph indices and availability are queryable so fallbacks can be chosen.
- Single buffered ‘chasing the beam’ style rasterisation for lowest possible latency.

** Accessibility/Internationalisation**:

- Separate on-demand alt-views to propagate compacted accessibility friendly contents.
- Semantically tagged input lets screenreader say ‘paste into job #1…’ rather than ctrl-v, proper separation between I/O streams makes it trivial to build ‘audio-only’ shell.
- Locale properties on input language and presentation language can change at runtime and is the property of a window, not a process global passed through environment variables.
- Everything is unicode.

These are features with direct impact for writing better shells. Then there are parts for writing better TUI applications and other command-line tools in general, but that is for another time.

Wow, I really like your ideas and examples. I am all in on your vision of a nextgen command line interface!

The point about the terminal being limited by 50-70 year old device restrictions really hits home. It’s a powerful reminder of why rethinking such a core tool is necessary, even if the inertia to change is massive.
