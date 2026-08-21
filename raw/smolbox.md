---
url: https://remyhax.xyz/posts/smolbox/
date_fetched: 2026-08-21
---

# Agentic AI in a Smolbox

Smolbox runs entirely in a browser tab. It is a full x86_64 virtual machine (VM) sandbox based on Alpine linux that runs under WebAssembly (WASM). It also has “AI” artificial intelligence in the form of an LLM model that runs via WebGPU which is exposed as tool calls inside the VM. You can optionally use the web browser file-picker to mount a folder containing files you want added to the VM, which are mounted read-only under `/mnt/host`.

The combined result being very similar to that of Claude Code or Codex, except that there’s no server side processing. The linux VM sandbox, LLM, tool calls, and file access controls only exist in the user’s browser tab.

This blog will not wax poetic about the end result as if it’s something I feel like I can claim any credit for, because I can’t, and shouldn’t. I knew exactly what I wanted, how to do it, what to use, and how to validate success. AI put all the pieces together for smolbox exactly as I asked with zero ambiguity.

Smolbox is a proof-of-concept to point out “this was always possible” and talk about the disservice the state of AI security is doing for the next generation of their users.

# My back hurts

I’m in my early 30’s. I’m sat down writing this blog because I pulled my back taking out the trash yesterday. I have a credit card and pay for two different AI subscriptions which run 24/7 agentic workflows without flinching about the cost. I have so many computers I lose them for months without noticing.

But way back sometime in the 2000’s I had a cell phone (LG Dare) and was a teenager. I had unlimited text and pics (SMS/MMS) messages, but “data” was still such a prohibitively expensive thing that accidentally tapping the wrong button on my phone would be visible on the monthly bill. I got a spam text message one day that was *from an email address*. After playing around a bit with sending messages to `my-ten-digit-cell-number@vzwpix.com` I’d figured out that there was an MMS<->Email bridge managed by Verizon. This led me to find an enterprising PHP developer online who ran an email service that allowed you to send an email with a subject line as a URL and receive an email back with the contents of a web page. After asking them nicely, they confirmed my email address and I was able to “browse” the internet via my unlimited text messages. This free access to information without associated cost increase lead me to read the developer manuals and SDK for the phone itself (with the phone locking up and freezing for minutes at a time as it tried to handle a full web page of content where a small text message was expected), eventually realizing that there was a path traversal vulnerability when ringtones were loaded via the MicroSD card slot, and being able to overwrite the builtin World Clock “app” written in MacroMedia Flash Lite with a mini-golf game.

I think about that random developer I reached out to a lot. The service was slow. The formatting of the web pages sucked. It made my phone freeze up. And yet it was everything I had ever wanted, because it made something *possible*.

They probably didn’t think much of it, or were just happy to see someone using their small service they’d built, but they opened the floodgates and set me down a path for what would ultimately shape my entire career path. All they did was make something *possible*.

# But are teenagers agentic enough?

If I were a teen today, I have no doubts I’d have figured out how to get access the internet. Kids are resourceful like that. Whether it be a school-issued iPad, computer at the local library, or a used hand-me-down laptop that was no longer needed. That’s the easy part.

But can they install Claude code on those locked down school-issued iPads? Are they able to install arbitrary programs on the public computers at the library? Can they purchase an AI subscription to learn how these things work? Do they even have a credit card to purchase that subscription?

Yes, yes they can! There’s always someone who knows the loopholes to getting around the guardrails of those locked down systems, but that comes with caveats of limited access and getting in trouble when you’re caught. It’s the same with the currency problem. There’s always a slightly older friend who has a card who can be bartered with. Alternatively, pick a random major online game with tradeable assets or an in-game currency and you’ll soon find an economy that operates in such a way that it’s not entirely unlike money laundering and fraud whereby real tangible currency exits the system in a readily usable form. Then they’ll probably stretch that money as far as they can by buying cheap “unused” (hacked accounts) token capacity through fly-by-night vendors on some random Telegram messaging channel.

If I were a teen today, I’d almost certainly want to be playing with agentic AI. After all, it’s the future, right? If I were a teen today, I’d almost certainly take the path that made it *possible*.

# Sandboxes and Smolboxes

In computer security, a sandbox is a security mechanism for separating running programs, usually in an effort to mitigate system failures and/or software vulnerabilities from spreading. The sandbox metaphor derives from the concept of a child’s sandbox—a play area where children can build, destroy, and experiment without causing any real-world damage. - https://en.wikipedia.org/wiki/Sandbox_(computer_security)


Smolbox is a sandbox; a small one. A safe play area that cannot cause any real-world damage, requiring no particularly special permissions. It works on school issued iPads, library computers, and hand-me-down used laptops for free. Sandboxes are trivial to extend the features of. You just put more things *in* the box to play with.

Smolbox is slow. It’s kludgy at times. It’s admittedly not particularly great at anything other than being private and not collecting any data whatsoever.

But it makes things *possible* for a large population of people to do safely, and it’s easy for a random developer to do. I know *exactly* what that can mean to someone.

It’s possible. What remains to be done is only a question of who is both capable and willing, and right now my back hurts and I’m a little grumpy.
