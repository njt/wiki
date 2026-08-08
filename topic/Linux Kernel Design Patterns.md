# Linux Kernel Design Patterns

Neil Brown's three-part LWN series (2009) extracted ten design patterns from the Linux kernel source, applying Christopher Alexander's pattern-language methodology to operating system code. The patterns span three scales: fine-grained reference counting, mid-level data structure design, and subsystem architecture. The series' most enduring contribution is the "midlayer mistake" — a pattern/anti-pattern diagnosing why imposed intermediate abstraction layers fail and prescribing library-based alternatives.

---

## Key Quotes

> "The core thesis of the 'midlayer mistake' is that midlayers are bad and should not exist. That common functionality which it is so tempting to put in a midlayer should instead be provided as library routines which can used, augmented, or ignored by each bottom level driver independently."

This is the article's central claim, and it's a strong one. Brown isn't saying abstraction is bad — he's saying *imposed* abstraction is bad. A library is opt-in; a midlayer is mandatory. The distinction sounds subtle but cascades through every design decision in a subsystem.

> "Here we see the first reason to dislike midlayers - they encourage special cases. When writing a midlayer it is impossible to foresee every possible need that a bottom level driver might have, so it is impossible to allow for them all in the midlayer."

This is the structural argument. Midlayers are prediction engines, and prediction engines are always wrong eventually. The special cases accumulate because the midlayer author couldn't have known what every driver would need. A library, by contrast, doesn't need to predict — it just offers functions. Drivers that need something different simply don't call the ones that don't fit.

> "This arrangement is another tell-tale for a midlayer. When a primary data structure contains a pointer to a subordinate data structure, we probably have a midlayer managing that primary data structure."

One of Brown's most concrete diagnostics. The embedded anchor pattern (from part 2 of the series) is the fix: the bottom-level driver allocates its own structure with the library's structure embedded inside it. The allocation flows from the driver up, not from the midlayer down. `struct inode` in 2.6 is the positive example; `struct request_queue` in the block layer is the cautionary one.

> "A more library-like approach would have the VFS pass a path to the filesystem and allow it to do the lookup, either by calling in to a cache handler in a library, or by using library routines to pick out the name components and doing the lookups directly against its own stored file tree."

This is the dcache critique distilled. The VFS gets the top layer right (thin `vfs_` functions → `_operations` function pointers), and the page cache gets the library pattern right (ext3 uses 8 of 13 generic functions, wraps 2 more, ignores the rest). But the dcache imposes its services on every filesystem — even network filesystems where loop detection must happen on the server, and virtual filesystems where all names are already in memory.

> "Here too ends our little series on design patterns in the Linux kernel. There are doubtlessly many more that could be usefully extracted, named, and illuminated with examples."

The modesty is genuine. Ten patterns from one codebase, each with multiple examples and counter-examples. This is applied Alexander methodology — name the recurring problem, describe the solution, show where it works and where it doesn't. The article is both a contribution (the patterns themselves) and a proof of concept (that kernel code has pattern-level structure worth extracting).

## Key Themes

#pattern #design-pattern #linux-kernel #software-architecture #abstraction #midlayer #library-design #reference-counting #embedded-anchor

### The Midlayer Mistake

The centerpiece pattern. A midlayer sits between a generic top layer (VFS, block layer) and specific bottom-layer drivers (ext3, SCSI controller). It receives requests, preprocesses them, and passes them down. The problem: midlayers impose functionality on every driver, accumulate special cases as unforeseen requirements emerge, and group unrelated code together. The fix: a thin top layer calling directly into drivers, backed by a rich library that drivers opt into.

**Tell-tale signs of a midlayer:**
1. Code imposed on lower layers whether they need it or not
2. Special cases accumulating in the common code path
3. Primary data structures containing pointers to subordinate data structures (embedded anchor is the fix)
4. Unrelated functionality bundled into one support function (`__make_request()` calling `blk_queue_bounce()` alongside elevator logic)

### The Three Subsystem Case Studies

**Block layer elevator:** Once a true midlayer (`make_request()` imposed on all), now partially reformed. `blk_queue_make_request()` lets drivers register their own handler — virtual drivers (md, dm, loop, drbd) and even some physical drivers (umem) bypass the elevator entirely. But `struct request_queue` still mixes block-interface fields with elevator-specific fields, and `__make_request()` still calls unrelated bounce-buffer code.

**VFS page cache vs. dcache:** The page cache is the exemplary library — ext3 uses generic functions for 8 of 13 file operations, wraps 2 with filesystem-specific logic, and ignores the rest. The dcache is the counter-example: imposed on every filesystem, even network filesystems (where loop detection belongs on the server) and virtual filesystems (where names are already in memory). The dcache has accumulated special-case flags like `DCACHE_NFSFS_RENAMED` — exactly the symptom Brown predicts.

**MD/RAID layer:** A mess produced by history. Major number scarcity forced all RAID levels under one driver, creating a midlayer mindset that persists. `md_check_recovery()` bundles unrelated tasks (metadata update, bitmap flush, device removal, recovery check). `md_register_thread()` is a genuine library function — but md takes it upon itself to call `md_unregister_thread()` rather than leaving that to the personality module. The path to dm/md unification, Brown argues, starts with replacing the midlayer.

### The Full Pattern Catalog

Brown closes with a summary of all ten patterns from the three-part series:

- **kref**: Reference counting when the object is destroyed with the last external reference
- **kcref**: Reference counting when the object can persist after the last external reference is dropped
- **plain ref**: Reference counting when object lifetime is subordinate to another object
- **biased-reference**: An anti-pattern — adding a bias to a reference counter to store one extra bit of information
- **Embedded Anchor**: Embedding a generic structure (list head, kobject) inside the driver's own allocation, so the driver owns the memory and the library operates within it
- **Broad Interfaces**: Many specific function calls instead of one overloaded function; don't squeeze use-cases together
- **Tool Box**: Provide composable tools rather than a complete solution; let drivers build what they need
- **Caller Locks**: When in doubt, the caller takes the lock, not the callee — gives the client control
- **Preallocate Outside Locks**: Allocate memory before acquiring locks; obvious but widely practiced
- **Midlayer Mistake**: Provide libraries, not imposed midlayers

## Critical Analysis

**The article is strongest when it's specific.** Brown's three case studies each surface different failure modes: the block layer shows midlayer-to-library migration in progress, the VFS shows a subsystem that got the top layer right but fumbled one service, and MD shows what happens when historical constraints fossilize into bad architecture. This is pattern documentation done right — each example sharpens the pattern, and the counter-examples (page cache as good library, dcache as problematic midlayer) prevent it from becoming a slogan.

**The dcache defense is worth taking seriously, and Brown does.** The dcache isn't just laziness — there are real concurrency problems with directory renames that benefit from a single implementation. "Rather than have every filesystem potentially getting these wrong, they can be solved once and for all in the dcache." Brown acknowledges this, and the article is stronger for admitting that the pattern has exceptions. The question isn't "is the dcache bad?" but "could it be a library that filesystems opt into?" — and Brown's honesty about not having a comparison point is refreshing.

**The series prefigured modern software architecture debates.** Reading this in 2026, the midlayer mistake maps cleanly onto every framework-vs-library debate since. React: library. Angular: midlayer. Express: library. Django: midlayer. The pattern isn't about Linux — it's about the structural tendency of abstraction layers to harden from optional services into mandatory intermediaries. Brown's four tell-tale signs work as a diagnostic for any layered system.

**What's missing: when midlayers win.** Sometimes the imposed solution is the right one. The dcache case study hints at this (correctness guarantees that per-driver implementations would get wrong), but Brown doesn't develop a theory of when the trade-off favors imposition. Memory safety through language design rather than programmer discipline is another example — Rust's borrow checker is a midlayer, and it works. The pattern needs a companion: "justified midlayer," with criteria for when the correctness/safety benefit outweighs the flexibility cost.

**The series is also a methodology demonstration.** Ten patterns from one codebase, each named, documented with examples and counter-examples. This is exactly what [[A Pattern Language (Christopher Alexander)]] prescribed for the built environment and what the Gang of Four attempted for object-oriented design. Brown shows it works for kernel code too — and laments that nobody continued the work. Fifteen years later, the kernel has only grown more complex, and the pattern-level design knowledge remains mostly oral tradition. An LLM-assisted extraction of kernel design patterns from git history and mailing list discussions would be a fascinating project.

**The library-over-midlayer prescription maps to modern API design.** When [[The Log — Unifying Abstraction for Real-Time Data]] argues for the log as a universal abstraction, it's making a library argument: here's a primitive, use it where it fits. When microservices advocates argue for smart endpoints and dumb pipes, that's the same pattern. When [[Software Engineering Craft]]'s hub page catalogs "systems ideas that sound good but fail 9 out of 10 times," many of those failures are midlayer mistakes in different clothing — pluggability frameworks, over-abstracted DI containers, premature platform teams. The midlayer mistake isn't a Linux pattern; it's a software pattern that Linux happened to surface particularly clearly.

---

*Sources: [[raw/336262]], [[summary/336262]]*
*Last updated: 2026-08-08*
