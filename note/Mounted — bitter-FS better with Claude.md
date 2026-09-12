# Mounted — bitter-FS better with Claude

Tomasz Mloduchowski's war story of using Claude Code (Opus 4.8, running as root) to recover a 41 TB BTRFS filesystem destroyed by a ten-month dual-mount iSCSI corruption — a condition every standard recovery tool failed on. Claude diagnosed the problem from first principles (two divergent transaction histories, one unreferenced but intact), wrote custom Python scanners to catalog 4 million metadata nodes, found intact tree roots the superblocks didn't point to, rebuilt 19 dead metadata leaves from the extent tree's back-reference index, and remounted with zero data loss. The human contributed a passphrase and two multiple-choice answers. The repo of tools Claude wrote in the ~4h session is public.

---

## Key Quotes

> "A dm-snapshot overlay turns a one-shot recovery into unlimited retries. All experiments hit the overlay; the underlying copy never changes; every destructive idea becomes a reversible experiment."

This is the single most transferable pattern in the piece. `blockdev --setro` + dm-snapshot before touching anything. It's the block-level equivalent of [[Harness Engineering]]'s "harness beats vigilance" — mechanism over intent. Mloduchowski calls it "step zero" of any recovery, and he's right. The overlay costs a few hundred GB of scratch space and buys you the courage to try anything.

> "Everything starts from the superblock, and the chunk tree is the map from logical addresses to physical disk offsets. The super can bootstrap only the tiny SYSTEM area by itself; lose the chunk root and every logical address where the real metadata lives becomes unmappable — for the kernel and for every recovery tool at once."

A single fifty-byte pointer is the linchpin. This is a design smell in BTRFS (and COW filesystems generally): superblocks that are last-writer-wins, with a bootstrap that's too small to self-repair. But it's also a reminder that complex systems often have single points of failure hidden in plain sight. The recovery worked because Claude understood this coupling — it didn't try to fix the fs tree first, it went for the chunk root because it knew *nothing else would work without it*.

> "The superblock's world ends at generation 24312 — yet here is metadata from 548 transactions later. There were two divergent transaction histories interleaved on the disk. Git users: picture a repo where someone force-pushed every ref to a corrupted commit, while the real history sits intact in the object store."

The Git analogy is the cleanest mental model in the piece. COW filesystems and Git share the same fundamental insight: data is never overwritten, only referenced. When the refs break, the data survives — you just need to find the right commits and repoint the branches. Claude understood this immediately; the standard tools, built to follow pointers, couldn't see past the broken ones.

> "`btrfs check` reported 64.8 million errors. Claude adjudicated by picking one complained-about extent and cross-examining it through the kernel's FIEMAP: both the extent record and the file mapping existed and agreed. The 64.8 million 'referencer count mismatch' errors were false positives."

The number that should terrify anyone who runs `btrfs check` on a damaged filesystem. One early cascading failure produced 64.8 million false alarms — while the 19 real dead nodes went unreported because btrfs-progs' walkers silently stop reporting after "Ignoring transid failure." This is [[Guardrails and Feedback Loops]] in the negative: when your verification tools lie both ways simultaneously (over-reporting noise, under-reporting signal), you're worse off than having no tools at all.

> "The extent tree is a reverse index. Back-references can resurrect dead fs-tree leaves byte-for-byte, and a compressed filesystem hands you structural validation — do the frames decode? — for free."

This is the genuinely clever recovery insight. BTRFS stores back-references in the extent tree for every data block — a reverse index from data to metadata. When the fs-tree leaves describing a file's extent map were destroyed, the extent tree still knew what data belonged to which file at which offset. And because BTRFS compresses data, every recovered extent could be validated by decompressing it: if the zstd frames decode cleanly and tile perfectly, the reconstruction is correct-by-construction. This is [[Correct by Construction]] as a recovery technique, not just a design principle.

> "Claude stopped trusting walkers entirely and wrote its own: visit every node of every current tree from B's roots and verify every parent→child edge — expected bytenr, generation, owner, level — against the 4-million-node catalog."

The crucial escalation. First you try the standard tools. Then you try the standard tools with fixes. Then you stop trusting *anyone's* tools and write your own verifier from first principles. Claude's exhaustive edge verifier checked 4,036,519 parent→child edges and found exactly 19 dead nodes. The ground truth turned out to be 0.0005% damage, not 64.8 million errors. This is what it means to [[Compound Engineering|compound]] verification: each layer catches what the previous one missed.

> "The filesystem that took ten months to die took one evening to fix. Twice."

The epilogue where Claude independently re-derived the entire diagnosis on the original LUN (not the copy) and applied the same 320 KB of writes is the clincher. Same 19 dead leaves, same locations, same fix. And then a 635 KB undo-backup *before* writing anything — paranoia as engineering discipline.

---

## Key Themes

- #pattern **dm-snapshot overlay as recovery harness**: `blockdev --setro` + dm-snapshot before any recovery work. Unlimited retries, zero risk to the original. Applies to any block-level repair, not just BTRFS.
- #concept **COW filesystems die pointer-first**: The data survives; the refs break. Recovery is finding the right commits and repointing. Same insight as Git's object store.
- #concept **The chunk root is the single point of failure**: Lose one ~50-byte pointer and every tool goes blind simultaneously. A COW filesystem design smell with real operational consequences.
- #tool **Claude Code as recovery engineer**: Not a coding assistant — a first-principles diagnostician operating as root on a 41 TB patient. Bash-history archaeology, raw-disk Python, synthetic leaf surgery, verification harness writing — all in one session.
- #pattern **Exhaustive, multi-layer verification**: Probe sweep → `btrfs check` (discarded as unreliable) → kernel-level reads → custom edge verifier. Trust no single tool. The final word came from a screenful of Python.
- #concept **Extent tree as reverse index**: BTRFS stores back-references from data to metadata. When forward references die, the reverse index can reconstruct them — and compressed data provides structural validation for free.
- #pattern **iSCSI LUN locking as operational basics**: Two kernels writing to the same block device for ten months with no alert. Persistent reservations exist for a reason.

---

## Critical Analysis

**This is the best Claude Code war story I've read.** Better than benchmarks, better than productivity anecdotes — this is Claude doing something a human expert couldn't do (or at least couldn't do as carefully, as quickly, and with as little bitterness). The story works because it's specific: 41 TB, 4,063,210 nodes, 19 dead leaves, 320 KB of writes. Not vibes — engineering.

**The dm-snapshot pattern should be in every sysadmin's toolkit.** It's the most transferable technical lesson here and it's barely a paragraph. `blockdev --setro` followed by a dm-snapshot overlay: zero risk to the original, unlimited retries, and the courage to attempt things you'd never try on live data. This is [[Harness Engineering]] at the block layer — mechanism beats vigilance every time.

**The "don't trust your tools" thread cuts deeper than it looks.** `btrfs check` produced 64.8M false positives while silently skipping the 19 real problems. `btrfs-find-root` ballooned to 59 GB RSS and never converged. The standard Python CRC32C library wasn't installed on the recovery box. Every layer of the recovery involved writing a custom tool because the existing one was either wrong, missing, or both. This is a specific instance of a general lesson: agent-built verification harnesses are often more reliable than established tools because they're purpose-built for *this* filesystem, *this* corruption, *this* moment.

**The human's role is fascinating and honest.** Mloduchowski doesn't inflate his contribution: "one passphrase, two multiple-choice answers, and an increasingly incredulous expression." But the two decisions he *did* make were high-leverage: (1) the choice to attempt forensics on the damaged files rather than accept partial recovery, and (2) the final sign-off to write the fix to the pristine copy. This is [[From AI Studio to AI Forge]]'s "human changes altitude" pattern — the human picks the strategy; Claude executes the tactics.

**The COW-as-Git analogy is powerful but has limits.** Mloduchowski is honest about this: "Don't over-generalize this: freed blocks do get reused — that's precisely the mechanism that shredded history A's view." History B survived because its refs were still live; history A's weren't. In Git terms, B's commits were still on a branch; A's had been garbage-collected. The analogy works precisely because both systems share the same fundamental design choice (copy-on-write, never overwrite), and it breaks where they differ (Git's GC is explicit; BTRFS's block reuse is continuous and silent).

**The zstd validation trick deserves more attention.** Recovering extent maps from the back-reference index and then validating by decompressing is elegant — structural correctness falls out of a property you were going to check anyway. This is the recovery equivalent of [[Correct by Construction]]: design the reconstruction so that failure is impossible to miss.

**What's missing: the happy path.** The piece is a recovery story, not a design critique — but it raises the question of why BTRFS (and COW filesystems generally) don't have a "find alternate transaction histories" mode built in. The dual-mount problem is a known failure mode for iSCSI; the recovery path shouldn't require writing custom Python scanners against raw disk. The tools exist in a narrow band between "mount with rescue flags" and "restore from backup" — and this story suggests there's a vast territory of recoverable states between them that current tooling simply can't see.

**The blog post itself being 90% Claude-written is a flex, not a disclaimer.** It's better written than most human technical writing, and Mloduchowski's decision to include Claude's raw draft in the public repo is a transparency move more authors should emulate. The human's role shifts from "writer" to "editor and publisher" — deciding what to say yes to, not composing from scratch.

---

*Sources: [[summary/mounted-bitter-fs-better-with-claude.md]]*
*Last updated: 2026-06-09*
