# Make the Easy Change Hard

A practitioner's case study in inverting Kent Beck's refactoring maxim. Instead of "first make the hard change easy, then make the easy change," drmorr deliberately made an easy feature request hard by refactoring SimKube's architecture first -- replacing shared mutable state with message passing -- then implemented the originally-simple feature on better foundations.

---

## Key Quotes

> "tokio documentation suggests that it is 'easy' to not use them, but I spent quite a while trying to figure out a suitable way to architect it without using tokio::Mutex and eventually gave up."

> "Hopefully you learned something about Kubernetes and/or Rust along the way!"

## The Problem

SimKube traces Kubernetes cluster activity for simulation replay. When tracking both Deployments and ReplicaSets, owned resources get duplicated -- the tracer records them AND the controller manager creates them during simulation. The fix sounds simple: filter out resources that have an OwnerReference pointing to another tracked resource.

But the existing architecture made the simple fix dangerous. Multiple watchers held `Arc<Mutex<TraceStore>>` references, and extending ownership lookup to all resource types would create brutal mutex contention.

## The Deliberate Detour

Instead of bolting the filter onto the existing architecture, drmorr spent multiple weeks refactoring:

1. **Replaced shared state with channels.** Watchers now send messages via `mpsc::Channel` to a coordinator, rather than directly mutating the TraceStore through a shared mutex.

2. **Fought tokio's ownership model.** Standard library primitives don't compose with async Rust. Tokio's alternatives have their own constraints -- `tokio::mpsc::Receiver` needs `&mut self`, tokio tasks must be `'static`, dynamically-created channels aren't. Each solution opened the next problem.

3. **Moved filtering to export time.** Ingest-time filtering fails because message ordering isn't guaranteed -- a ReplicaSet can arrive before its owning Deployment. Export-time filtering sidesteps this at the cost of a rare race condition, accepted as a practical tradeoff.

Two PRs: #199 (architecture) and #200 (the actual feature). The feature PR was straightforward because the architecture PR did the real work.

## Key Themes

#software-craft #refactoring #rust #complexity

This is a textbook illustration of the tension between tactical and strategic programming. The tactical path -- bolt the filter onto the mutex-guarded store -- would have shipped faster but made every subsequent change harder. The strategic path cost weeks upfront and paid off in a cleaner substrate.

The piece also documents a genuinely useful pattern for async Rust architecture: replacing `Arc<Mutex<T>>` with channel-based message passing to avoid contention, and the specific friction points (tokio vs stdlib, `'static` lifetime requirements, receiver mutability) that make the transition non-trivial.

## Critical Analysis

This is a solid engineering diary, not a thought piece. Its value is in the specifics: the exact sequence of architectural decisions, the dead ends (ingest-time filtering), and the honest accounting of tradeoffs (the export-time race condition). Most "refactoring war stories" are retrospective justifications; this one shows the mess in progress.

The weakness is that drmorr doesn't articulate *when* you should invert Beck's maxim. The implicit answer is "when the easy change would deepen architectural debt," but that's always true at some level of abstraction. The harder question -- how much architectural investment is justified for a given feature -- goes unaddressed. The multiple weeks of refactoring produced a better SimKube, but SimKube is a niche Kubernetes simulation tool. Was the investment proportional? The author seems to think so, and the improved architecture validates it, but the general principle needs a cost function that isn't here.

The async Rust content is the most transferable part. The tokio vs stdlib friction is real, poorly documented outside of scattered GitHub issues, and bites every Rust developer who moves beyond toy examples. This is one of the better practical accounts of navigating it.

## Cross-Links

- [[Elements of Code]] -- "wrong in correctable ways" is precisely what the architecture refactor enabled
- [[Simplicity in the Age of AI-Assisted]] -- the inherited complexity (Arc/Mutex soup) that LLMs would faithfully reproduce
- [[Systems Ideas That Sound Good]] -- the original architecture was a DIY async trap; the refactor escaped it
- [[The Mythical Agent-Month]] -- at SimKube's scale, the brownfield barrier is architectural, not just LOC
- [[Software Engineering Craft]] -- this is craft: the judgment to refactor before feature work

---
*Sources: [[raw/make-the-easy-change-hard]]*
*Last updated: 2026-05-14*
