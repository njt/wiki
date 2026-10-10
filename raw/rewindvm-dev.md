---
url: https://rewindvm.dev/
date_fetched: 2026-10-10
---

Deterministic Linux VMs

# Scrub any run back to the step that broke it.

Rewind VM runs a Nix build, a test suite or any Linux command inside a deterministic KVM virtual machine. The same inputs produce the same events at the same steps, every time. When it fails, drag the timeline back to where it went wrong, look around, and fork from there.

The engine and CLI are open source under the MIT license. The desktop app is $49 for personal use and free to evaluate.

## How it works

Nothing is recorded while the workload runs. A run is a function of its inputs, so any step of it can be computed again.

- 
            ### Pack the inputsThe inputs, a Nix closure or any root filesystem, become a read-only erofs image mapped straight into the VM's memory. A small Linux kernel boots and runs your command. 
- 
            ### Run deterministicallyOne vCPU on stock KVM, so code in the VM runs on the real CPU. The VM's kernel carries a small Rewind platform: interrupts arrive only when the VM hands control to Rewind, time moves only then, and the timestamp counter and hardware RNG are hidden. A step is one of those handoffs. 
- 
            ### Keep keyframesEvery quarter second of wall time, Rewind snapshots the machine using KVM's dirty-page log. Pages go into a content-addressed store (BLAKE3, zstd), so a page shared by keyframes, runs and forks is stored once. 
- 
            ### SeekTo reach a step, Rewind restores the latest keyframe at or before it and runs forward. Same inputs, same events at the same steps. 

## In the desktop app

The real app, on the bug the tutorials follow.

- 
                  **The timeline**The whole run, with where it parted from the passing one.
- 
                  **Build log**What the build printed, up to the playhead.
- 
                  **Processes and files**What was running, and what was written.
- 
                  **The step's event**Here the SIGSEGV, and where the runs parted.
- 
                  **Fork from here**Reruns from this step under a new schedule.
- 
                  **Every run**All runs and forks of the build, as a tree.

## Bring a command, or bring a derivation

Any Linux workload

### A root filesystem and a command

`$ rewind run --root mylib.tar --cwd /src -- make check`
              The root is a directory, an erofs image, or a tarball such as
              `docker export` writes. The VM mounts it under a writable overlay, and
              nothing the command writes reaches your disk.
            

- flaky tests
- 
                  `rewind check`reruns the command under perturbed schedules until one ends differently, then names the step whose reschedule decides it.
- CI failures
- A failing run is a directory holding its inputs and every event with its step. Keep it and scrub back from the crash instead of reading a log that ends there.
- heisenbugs
- A race that vanishes when you add a printf stays put. A failing run fails the same way every time it runs.
- exploration
- Fork a passing run at a step under new schedules to try other interleavings from that point on.

For Nix users

### Nix builds are already almost a closed world

```
$ rewind nix .#mylib
$ rewind check .#mylib
```
Pinned inputs, no network, fixed paths. Point Rewind at a flake attribute and most of the work of making the build deterministic is already done.

- .drv
- The derivation hash is the unit of reproduction.
- the machine
- The VM's kernel is a derivation too, so the whole machine is pinned.
- rewind check
- 
                  `nix build --check`says the output "may not be deterministic".`rewind check`builds under perturbed schedules, stops at the first build that ends differently, and names the step that decides it: without that one perturbation, the build passes.
- the output
- 
                  `rewind nix`hashes each output the VM builds and says whether it matches the copy already in your store, so you can see the VM built what`nix build`does.
- the clock
- Builds see today's date: the VM's clock starts at midnight UTC on the day of the run. The date is saved with the run, so a replay a year later sees the same one.

## Tutorials

The first two follow one real bug, a shutdown race in a small C thread pool, from install to fix; the third is the rest of the tools on the same bug. Every transcript in them is real output.

### A flaky Nix build

              Install with Nix, let `rewind check` find the interleaving that crashes a
              derivation's tests, then replay it, fork it and check the fix.
            

### A flaky test in a container

The same bug from a Docker image with no Nix: export the filesystem, find the failing schedule and replay the crash exactly.

Read the tutorial → Advanced### Advanced features

Short recipes: which thread held the CPU, watchpoints, the kernel's side of a crash, tools inside the VM, more CPUs, counting the schedules that fail from a step, and sharing a failing run.

Read the tutorial →## Case studies

Rewind VM on Nix's own test suite and on devenv. Every transcript in them is real output.

### A SIGPIPE in Nix's gc-closure test

              Rewind's first run of Nix's test suite hit a SIGPIPE flake in
              `gc-closure.sh` that Nix's issue tracker has no report of: a pipe into
              `head -n1` under `pipefail`, fixed with a here-string.
            

### A hang in Nix's store schema migration

              A known SQLite schema-migration hang in Nix master, reproduced in Rewind and pinned to
              `SQLITE_BUSY_SNAPSHOT` with gdb inside the VM.
            

### Lost task output in devenv

              A known devenv bug that dropped a task's last lines, reproduced in Rewind and traced
              with gdb to `tokio::select!` taking the child's exit before a buffered
              line.
            

## Download

For x86_64 Linux with KVM. The engine and CLI are open source on GitHub, and the desktop app is free to download and use while you evaluate it.

### With Nix

Run either straight from the flake, or install both into your profile:

```
$ nix run github:fzakaria/rewindvm -- pmu status
$ nix run github:fzakaria/rewindvm#app
$ nix profile install github:fzakaria/rewindvm github:fzakaria/rewindvm#app
```
On NixOS, add the flake as an input and turn on its module:

```
inputs.rewind.url = "github:fzakaria/rewindvm";
# in your configuration, with inputs.rewind.nixosModules.default imported;
# it also adds rewindvm.cachix.org to Nix's substituters
programs.rewind.enable = true;
programs.rewind.app.enable = true;
# AMD only: make the branch counter exact at every boot
programs.rewind.amdBranchCounterWorkaround = true;
```
### Without Nix

`$ curl -fsSL https://rewindvm.dev/install | sh`Or take the tarballs from the latest release yourself:

```
# the rewind command, with the VM's kernel
$ curl -L https://github.com/fzakaria/rewindvm/releases/latest/download/rewind-x86_64-linux.tar.gz | tar xz
# the desktop app
$ curl -L https://github.com/fzakaria/rewindvm/releases/latest/download/rewind-app-x86_64-linux.tar.gz | tar xz
# the kernel's debug symbols for rewind gdb, next to the command (150 MB)
$ curl -L https://github.com/fzakaria/rewindvm/releases/latest/download/rewind-debug-x86_64-linux.tar.gz | tar xz
```
          If `/dev/kvm` is not readable and writable by you, add yourself to the
          `kvm` group.
        

## Pricing

Evaluate the app free for as long as you like, with an occasional reminder. Buy a license when it earns its keep.

### Engine and CLI

Free MIT license

The engine and the command line. The VM's kernel is Linux with a patch and config of ours, GPL-2.0 as Linux is.

- 
                `rewind run`,`nix`,`check`,`fork`and`diff`
- 
                Any step at the prompt: `log`,`ps`,`cat`,`where`,`shell`and`gdb`
- Open source, for any use

### Personal

$49 once

The desktop app, for you.

- All your machines
- 3 years of updates
- Offline license key by email

### Commercial

$99 per seat

The desktop app, for use at a company.

- One seat per person using the app
- 3 years of updates
- Offline license key by email

The command line answers one question at a time. The app puts the whole run on one screen: a timeline to scrub from boot to the end, the failing run beside a passing one with where they first differ in words, the log, processes, files and source line at the playhead, a tree of every fork and schedule tried, and a fork, gdb or a shell at any step in a click.

A personal license is for one person, bought by that person, on all their machines. Use at a company takes a commercial license: a seat for each person who uses the app, on as many machines as they use it on.

Every license comes with a full refund within 30 days, for any reason. See the refund policy, and the app's license for what a license covers.

## Limits

- Linux on x86_64 with KVM.
- One virtual CPU. Your threads take turns, and speed comes from running many machines at once.
- A loop that spins without a system call is never interrupted. Locks, sleeps and I/O are fine.
- A run replays on other recent machines with the same CPU vendor, Intel or AMD.
- No network, other than loopback.
