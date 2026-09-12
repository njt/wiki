---
url: https://mloduchowski.com/-mounted-bitter-fs-better-with-claude/
title: "🎉 Mounted — bitter-FS better with Claude"
author: Tomasz Mloduchowski
date_fetched: 2026-06-09
date_published: 2026-06-06
description: "A 41 TB filesystem, two kernels that didn't know about each other, and the ~320 KB of writes that brought it all back."
topics:
  - agent-coding-workflow
---

# 🎉 Mounted — bitter-FS better with Claude

**TL;DR:** An unlocked iSCSI LUN got mounted by two OS instances at once - a condition that went unnoticed for ten months. Every standard BTRFS recovery tool failed. Claude (Opus 4.8, in a Claude Code session, as root) reconstructed what happened from first principles, found an intact-but-unreferenced second transaction history on the disk, hand-patched the superblocks to point at it, rebuilt the 19 metadata leaves that were genuinely destroyed, and remounted the filesystem: `mount -o ro;` no rescue flags, no errors. Data lost: **zero**.

*Full disclosure - this blog post, 90% written by Claude (but 100% read by me first, I decided not to edit it much). I was just a spectator.*

*Repo with the tools that Claude wrote during this ~4h session - please use at your own risk, read them first (I did skim them) - but you might find them all useful. Claude's raw draft of this post included for comparison too.* https://gitlab.defensiblelogic.com/pub/rebtrfs

## The setup

Somewhere in my lab there is a 41 TB BTRFS filesystem. It lives inside a LUKS2 container, on an iSCSI LUN exported by my NAS, and holds years of disk images, backups, and "I'll sort this out later" archives. 40 TiB used, zstd-compressed, about 7.5 million files. It's the backup tier — the place other machines get copied to. A backup of the backup does exist, but it's a few weeks out of date and *glacially* slow to restore from.

In February it died.

Cause of death: the LUN wasn't locked, and I assumed it was mounted in exactly one place. It was mounted in two. The dual-mount condition persisted for **ten months** — one instance doing the real work, the other sitting idle but alive, each kernel COW-allocating from what it believed was free space, neither knowing the other existed.

The standard tool parade - `mount -o ro,usebackuproot,nologreplay, btrfs-find-root, btrfs rescue chunk-recover, btrfs rescue super-recover, btrfs rescue zero-log,` `btrfs restore` — all failed. Some instantly, some after hours of grinding. Corrupted beyond the tools' ability. The standard advice at this point is "restore from backup," and that option was technically on the table; at the cost of a multi-day restore and the last few weeks of writes. Surgery first, surrender later.

So: re-sync a raw image of the LUN onto a local RAID-0 scratch pair (about a day over 10 GigE), open a Claude Code session as root, and state the problem more or less as: *"you have the machine, you have a copy, you have ~17 TB of free scratch space. Recover it."*

## First moves (in which nothing is trusted)

Claude's opening sequence:

1. `lsblk` / `mdstat` / LVM recon — locate the copy: a 41 T logical volume, crypto_LUKS inside.
2. Read root's `.bash_history` and reconstruct every recovery step already tried on the original — including the `pv < /dev/sdf > /dev/dm-0` line proving the LV held a raw, block-for-block image of the damaged LUKS volume.
3. Ask the human for exactly one thing: type the LUKS passphrase. No, not in a session, directly as `luksOpen /dev/mapper/RAID0-secure_repair s`
4. `blockdev --setro /dev/mapper/s` — kernel-level write protection on the decrypted device, before anything else gets a chance to touch it.
5. Build a dm-snapshot overlay backed by a 512 G copy-on-write volume.

That last step is the one to steal: **a dm-snapshot overlay turns a one-shot recovery into unlimited retries.** All experiments hit the overlay; the underlying copy never changes; every destructive idea becomes a reversible experiment. A failed repair costs a `dmsetup remove` instead of another day of resync.

```
lvcreate -n cow0 -L 512G RAID0
dmsetup create sr_work --table \
"0 $(blockdev --getsz /dev/mapper/s) snapshot /dev/mapper/s /dev/RAID0/cow0 P 32"
```

## The smoking gun

The superblock itself was fine — checksum valid, a healthy-looking 40 TiB filesystem at generation 24312. The rescue mount died at the very first dereference:

```
BTRFS error: level verify failed on logical 29245440 mirror 1 wanted 1 found 0
BTRFS error: level verify failed on logical 29245440 mirror 2 wanted 1 found 0
BTRFS error: failed to read chunk root
```

The super says: chunk root at logical 29245440, level 1, generation 24287. The node actually there (in both DUP copies) is level 0, and:

```
parent transid verify failed on 29245440 wanted 24287 found 24860
```

Generation **24860**. The superblock's world ends at generation 24312 — yet here is metadata from 548 transactions later.

That one line is the whole disaster. There were **two divergent transaction histories** interleaved on the disk. Git users: picture a repo where someone force-pushed every ref to a corrupted commit, while the real history sits intact in the object store. This isn't data recovery — it's finding the good commits and pointing the refs back.

- **History A** — the one all three superblock copies pointed at. Stopped at generation 24312. Its chunk, csum, device, and fs tree roots: overwritten rubble.
- **History B** — ran on to generation 24866, with real file writes up to hours before the volume's death. Complete, self-consistent... and referenced by *nothing*.

Each kernel, knowing nothing of the other, allocated from what *it* believed was free space — which included the other instance's freshly written metadata. B, the active instance, steamrolled A's recent tree nodes, including A's chunk root. A, mostly idle but still alive, delivered the final insult: it wrote the last superblocks — three valid, checksummed signposts pointing into its own wreckage.

And that's why every tool failed. **Everything starts from the superblock, and the chunk tree is the map** from logical addresses to physical disk offsets. The super carries a tiny embedded bootstrap map for the SYSTEM area itself, but every logical address outside it — which is to say, all the actual metadata — is unmappable without a valid chunk root. Even `btrfs-find-root`, the tool whose whole job is finding lost roots, can't read what it can't locate. Every tool was staring at history A's corpse while history B's filesystem sat intact, invisible, fifty bytes of pointer away.

## Finding the chunk root the hard way

On this filesystem the entire chunk tree lives in a single 8 MiB SYSTEM chunk that is identity-mapped — logical == physical, for the first of its two DUP copies anyway — 512 possible 16 KiB node slots. Claude wrote a ~60-line Python scanner: parse every slot's header in both copies, verify its checksum (crc32c table implemented inline, because of course the recovery box has no python-crc32c), and record owner / generation / level for every valid node.

The SYSTEM area turned out to be a graveyard of chunk-tree nodes from both histories — and among them, intact level-1 chunk roots from history B:

```
logical=26361856 gen=24864 level=1 nritems=269 [both copies AGREE, csum OK]
```

Repoint the overlay's superblock — `chunk_root`, `chunk_root_generation`, recompute the super's crc32c — and the mount failure moves *deeper*, which in this business is what progress feels like. The new error: device size mismatch, because history B had quietly **resized the filesystem** from 40 TiB to fill the whole 41 TiB LUN (very NAS-appliance behavior, and another fingerprint identifying which instance B was). Patch `total_bytes` too, and the chunk tree loads. The fs tree is still A's, still dead — but the map of the disk is back, and that unlocks everything else.

## Four million nodes, cataloged

Next problem: history B's tree roots exist on disk, but nothing points at them. `btrfs-find-root` is the canonical answer, and with the chunk tree restored it actually ran — badly: 16 GB of transid-warning spam in ten minutes, 59 GB of RSS, no convergence in sight. Claude killed it and did something simpler.

B's chunk tree says all metadata lives in exactly 62 GiB of DUP chunks. So: read all 62 GiB sequentially, parse every 16 KiB block header, and write a catalog — **4,063,210 metadata nodes**, every tree node from both histories, as a CSV of logical, physical, owner, generation, level. A few minutes of sequential I/O on mechanical RAID-0.

Query the catalog for root-tree nodes: history B's final root tree is at generation **24866**, level 0, eleven items. (Why 24866 when the chunk root said 24864? A tree root's generation is just the last transaction that modified *that tree* — the chunk tree sat out B's final two commits. Expected, not suspicious.) Parse that root-tree leaf raw and cross-check every tree root it references against the catalog:

```
EXTENT: @47105846919168 gen=24866 level=3 -> OK
DEV: @47105193230336 gen=24864 level=1 -> OK
FS: @47105458651136 gen=24866 level=3 -> OK
CSUM: @47105458716672 gen=24866 level=3 -> OK
FREE_SPACE: @47105846984704 gen=24866 level=1 -> OK
```

History B's final commit was **complete on disk**. The only thing wrong with this filesystem was that no superblock admitted it existed.

Repoint `root`, `generation`, `root_level` on the overlay's superblock, and:

```
# mount -t btrfs -o ro /dev/mapper/sr_work /mnt/sr_work
mount exit: 0
dmesg: (nothing)
```

A **plain read-only mount**. Not `rescue=all`, not `usebackuproot` — no rescue options at all. The kernel had no complaints. 🎉 Mounted.

Everything was there: 40 TiB of disk images, squashfs archives, home directories — including backups written the same evening the volume died.

## Trust, but verify (and don't trust the verifiers either)

A mount is not a recovery. The verification pass is where it got interesting:

- **Probe sweep**: read 7,497,112 files at five offsets each, with checksums live. Zero errors.

- **`btrfs check`** reported **64.8 million errors**. Claude adjudicated by picking one complained-about extent and cross-examining it through the kernel's FIEMAP: both the extent record and the file mapping existed and agreed. The 64.8 million "referencer count mismatch" errors were false positives — check's userspace traversal had hit one bad node early and cascaded into nonsense. (Two stale free-space-tree entries from mkfs days were real, cosmetic, and noted for later.)

- **Timeline forensics** from inode ctimes inside dead metadata: the dual mount began **April 17, 2025**, mid-copy-job. History A was almost certainly this very machine's stale iSCSI mount, idling for ten months with ~25 stray commits of atime noise — the last of them, written after B's death, produced those final lying superblocks. And the fork-day diff came back clean: the 787 inodes A touched after the fork resolve to 365 distinct names, every one of which is present in B's tree. Choosing B sacrificed nothing at all.

Then the kernel — the only verifier whose opinion is final — vetoed premature victory. A full read of one file EIO'd on a metadata block that every userspace walk had sailed past. It turns out btrfs-progs' tree walkers keep going but quietly stop *reporting* after "Ignoring transid failure." The check tool had screamed 64.8 million times about nothing and stayed silent about actual damage.

So Claude stopped trusting walkers entirely and wrote its own: visit every node of every current tree from B's roots and verify **every parent→child edge** — expected bytenr, generation, owner, level — against the 4-million-node catalog.

```
edges verified: 4,036,519
dead nodes: 19
```

That is the complete, exhaustive damage list for ten months of dual-mounted operation: **nineteen 16-KiB leaves** — 0.0005% of metadata — all clobbered by history A's zombie commits, all clustered in one ~6 MB allocation neighborhood, all of them metadata that one April-2025 balance run had written:

- **10 fs-tree leaves**: pure extent-map ranges (~230 MiB of file offsets) belonging to exactly two old VM images, `theia/ultrasound.raw` and `theia/docker-runner-VM-numa0.raw`. No inode items, no directory entries — just block maps.
- **9 csum-tree leaves**: checksums covering ~130 MB of those files' (plus one squashfs's) relocated data.

## Grafting leaves back onto a 41 TB tree

Here's the part I'll be retelling at parties. The *data* those dead leaves described was still on disk, still allocated — only its description died. And BTRFS keeps a reverse index: the **extent tree** (intact — verified edge by edge) stores a back-reference for every data extent naming its owner: tree, inode, file offset (for ordinary un-cloned extents, which these were).

So, for each dead fs-tree leaf:

1. Recover its exact key range from its still-living parent — interior nodes store each child's first key, and the next slot's key bounds the range.
2. Harvest every extent backref for that inode and range out of a dump of the extent tree. For the first dead leaf: 208 extents, tiling its 31.12 MiB range with zero gaps.
3. Validate, because trust must be earned: read every recovered extent and **zstd-decode it** — each compressed frame must decompress to exactly the size the tiling predicts. All 208 did.
4. Hand-assemble a replacement 16 KiB leaf in Python: header (bytenr, the generation its parent expects, owner, fsid, chunk-tree UUID — at the right offset on the second attempt), 25-byte item headers growing up, item data growing down, crc32c over the lot.
5. Write it to both DUP copies. On the overlay first, obviously.

One reusable gotcha from step 3: the `zstd` CLI refuses btrfs extents outright, because btrfs pads each compressed frame to a 4 KiB boundary and the CLI treats the padding as a corrupt second frame. Python's `zstandard decompressobj` stops cleanly at the frame boundary. File that one away.

The nine csum leaves got the same treatment with recomputed crc32c over the just-validated data. Those checksums are now correct-by-construction rather than original — the one honest asterisk in this entire recovery — but the data they cover decodes as valid zstd frames, and all three affected files (23 GiB + 256 GiB + 326 GiB) were subsequently read end-to-end through the kernel's checksum verification and decompression with zero complaints.

Re-run the exhaustive verifier on the grafted overlay:

```
edges verified: 4,036,519
dead nodes: 0
```

With my sign-off, the same writes went to the pristine copy: three superblocks and nineteen leaves. Overlays torn down, and:

```
/dev/mapper/s 41T 41T 801G 99% /mnt/secure
```

Mounted read-only at the exact path it lived at before it died.

*(A footnote on human involvement: when asked how much effort the damaged file was worth, I picked "attempt full carving forensics," expecting a grim hunt through a ~2.7 GB balance-relocation zone for zstd magic bytes. The extent-tree backref approach made carving unnecessary — the "forensics" turned out to be a clean query against metadata btrfs keeps anyway.)*

## What I'm taking away

First, the meta-lesson, which is why this post is titled the way it is: every step above - the bash-history archaeology, the protect-first discipline, the raw-disk Python scanners, the false-positive adjudication, the backref forensics, the synthetic leaf surgery - was driven by Claude in a single Claude Code session. My total contribution was one passphrase, two multiple-choice answers, and an increasingly incredulous expression. I've done manual btrfs surgery before. This was faster, more careful, and more *suspicious of its own tools* than I would have been - and considerably less bitter.

And the technical lessons:

1. **A COW filesystem dies pointer-first.** Every commit leaves a complete, internally consistent tree on disk, and history B's live roots were never freed — so its entire filesystem survived, unreferenced, behind superblocks that named the wrong history. (Don't over-generalize this: freed blocks *do* get reused — that's precisely the mechanism that shredded history A's view. The refs died; the still-referenced object store survived.)

2. **Superblocks are last-writer-wins** — and the last writer can be an idle zombie mount flushing superblocks long after it last did anything useful.

3. **Everything keys off the super's chunk root.** The super can bootstrap only the tiny SYSTEM area by itself; lose the chunk root and every logical address where the real metadata lives becomes unmappable — for the kernel and for every recovery tool at once. Restoring that one pointer (fifty-ish bytes) un-bricks the entire toolbox.

4. **`blockdev --setro` plus a dm-snapshot overlay should be step zero** of any block-level recovery. Mechanism beats vigilance, and unlimited retries change what you're willing to attempt.

5. **The extent tree is a reverse index.** Back-references can resurrect dead fs-tree leaves byte-for-byte, and a compressed filesystem hands you structural validation — do the frames decode? — for free.

6. **Verify exhaustively, and don't trust a single verifier.** `btrfs check` said 64.8 million errors; the truth was 19 — and the walkers that under-reported were the same ones over-reporting. The final word came from a screenful of Python checking every edge against a node catalog.

7. **Lock your iSCSI LUNs.** Persistent reservations, an initiator allowlist, anything. The real horror of the ten-month dual mount is that nothing ever alerted — both mounts looked perfectly healthy right up until neither was. Ahem.

## Epilogue: the original gets the same medicine

Later that night, the *original* LUN got its turn. Everything local was unmounted and LUKS-closed first — one btrfs fsid visible to the kernel at a time, thank you — then an iSCSI login brought the real thing back online.

Here's the part that justified the paranoia: both the original and the copy had been worked over by standard btrfs tools at various points, *differently* — so nothing was assumed to transfer. The entire diagnosis was re-derived from the original's own bytes: superblocks, the full SYSTEM area, the chunk tree, and a fresh catalog of all 4,063,210 metadata nodes. Every one came back **byte-identical** to the copy's pre-fix state — months of tool abuse on two devices, and all of it turned out to be bark, no bite. Same 19 dead leaves, same locations.

Then, before a single byte was written: a 635 KB undo-backup of every region the fix touches, with a restore script. *Then* the same ~320 KB of medicine — superblocks repointed, nineteen leaves rebuilt and re-validated against the original's own data — and the edge verifier came back `dead nodes: 0`. The original mounted clean at the same path it had died on.

So the score is now two independently repaired instances of the same filesystem: the LUN on the NAS, and the local copy as a warm spare a day fresher than any backup. The stale free-space-tree entries got cleared, the volume went read-write, and as I post this a full scrub is grinding through all 40 TiB — over iSCSI, because the NAS's seven-drive RAID-6 with an obscene RAM cache in front of it outruns the local mechanical RAID-0. The filesystem that took ten months to die took one evening to fix. Twice.
