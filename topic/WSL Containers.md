# WSL Containers

Microsoft's first-party answer to Linux container development on Windows: a built-in CLI and API for running Linux containers through WSL, replacing the need for Docker Desktop or other third-party container tooling. Currently in public preview, aiming for GA in fall 2026.

---

## Key Quotes

> "WSL containers simplify this experience by providing a built-in, enterprise-ready way to create, run, and manage Linux containers on Windows, without requiring additional third-party tooling."

Commentary: The "without requiring additional third-party tooling" is the thesis statement. This is Microsoft saying: Docker Desktop on Windows is no longer a hard dependency for Linux container workflows. It's the platform absorbing a category that was previously served by a partner ecosystem — the classic integrator-to-competitor move. Docker Desktop, Podman Desktop, and Rancher Desktop are named as tools that "build on top of WSL" and will "benefit from these lower level platform changes." The framing is generous; the power dynamic is not.

> "This CLI tool has a familiar format and capabilities, meaning you can use your existing muscle memory when running Linux containers."

Commentary: `wslc.exe` is pitched as a drop-in replacement for the Docker CLI. The examples — running a Linux desktop container, checking GPU access with CUDA, exposing ports — are all classic `docker run` patterns. The alias (`container.exe` → `wslc.exe`) suggests Microsoft expects developers to think "container" not "wslc." The muscle-memory argument is the right one: CLI tools live or die on whether they fit into existing finger habits.

> "Windows applications can now also directly use containers as part of their application logic. WSL now ships a Nuget package that is available at nuget.org."

Commentary: The API surface (C, C++, C# via NuGet) is the strategic play that elevates this beyond a CLI convenience feature. This isn't just "run containers from your terminal" — it's "embed Linux containers as a runtime dependency in your Windows application." Combined with MSBuild/CMake integration, Microsoft is positioning Linux containers as a build artifact that ships alongside Windows binaries. This is the architectural inversion: containers as a platform capability, not a developer tool.

> "A new default networking mode for WSL container: 'consomme' which aims to improve compatibility."

Commentary: The name is absurd — "consommé" as in the clarified soup, because it clarifies networking — but the architecture is real: relay Linux network traffic through the Windows host so containers inherit Windows' networking environment, security policies, and enterprise integrations. This is a genuine technical contribution, not just packaging. VPNs, proxies, and corporate network setups are the hardest part of container networking, and relaying through the host sidesteps the entire problem class.

> "We aim to make this feature generally available in the upcoming fall of 2026."

Commentary: The timeline matters. Public preview at Build 2026 (May), GA in fall 2026 — that's a ~6-month preview cycle. Aggressive for an enterprise platform feature that touches filesystems, networking, and security tooling. The pre-release-only availability means early adopters are effectively unpaid testers for the underlying virtiofs and consomme subsystems.

---

## Key Themes

#tool #microsoft #container #linux #windows #dev-containers #enterprise

**Platform absorption of the container tooling category.** WSL containers isn't a better Docker — it's Microsoft saying Linux container runtimes are infrastructure, and infrastructure belongs in the operating system. The same play as Windows Defender absorbing antivirus, or Hyper-V absorbing virtualization. Third-party tools (Docker Desktop, Podman, Rancher) are recast as UX layers over Microsoft's runtime. The "also benefit from these lower level platform changes" language is polite but unmistakable.

**Enterprise management as the differentiator.** The Intune integration (registry allowlists, distro vs. container control) and MDE container event monitoring are features Docker Desktop can't match. Microsoft is targeting the enterprise IT admin who needs to answer "how can I control which Linux images are allowed in my organization?" — and doing it through the same management surface they already use for Windows. This is the moat: not better container performance, but better organizational control.

**The API is the sleeper feature.** Most coverage will focus on `wslc.exe` as a Docker CLI replacement. But the NuGet-packaged API with MSBuild/CMake integration is more architecturally significant. It means Windows applications can declare Linux container dependencies the same way they declare library dependencies. CI/CD pipelines build Windows binaries and Linux containers in one pass. This blurs the line between "Windows application" and "application that happens to use Linux containers" in a way that could reshape how Windows developers think about platform boundaries.

**Underlying infrastructure improvements are the real product.** virtiofs (2× faster Windows file access), consomme networking, and improved memory reclaim are improvements to the WSL kernel and VM layer that benefit everyone — including Docker Desktop users. The container feature is the visible launch vehicle, but the long-term impact is in the substrate improvements that make all Linux-on-Windows workflows faster and more reliable.

---

## Critical Analysis

**The Docker relationship is tense.** Microsoft carefully names Docker Desktop, Podman Desktop, and Rancher Desktop as tools that "build on top of WSL" and will benefit from platform improvements. This is true but incomplete. WSL containers directly competes with Docker Desktop's core value proposition: "run Linux containers on Windows." The Docker Desktop team has spent years building WSL 2 backend support, filesystem sharing, and networking integration. WSL containers makes much of that work redundant. The question is whether Docker pivots to enterprise features (SBOM, supply chain, registry management) or becomes a thinner and thinner wrapper over Microsoft's runtime.

**The timeline is ambitious for enterprise infrastructure.** virtiofs and consomme are touching the two scariest subsystems — filesystem and networking — and shipping them as defaults in a public preview. The memory reclaim improvements are also in the "mission critical" category. Shipping all three simultaneously, with a ~6-month path to GA, is either evidence of unusual confidence or a schedule that will slip. The fact that these are container-only for now (not default WSL) suggests the team knows the risk and is using containers as the proving ground before wider rollout.

**The Intune registry allowlist addresses a real pain point but raises questions.** "How can I control which distros/Linux images are allowed in my organization?" is the top customer ask, per the post. The allowlist solves it, but also creates a new class of enterprise policy work: maintaining and updating image allowlists across organizations. The GPO/ADMX-first, Intune-later deployment order is the right call for enterprises that need this today, but it also means the feature ships to the most conservative deployment surface first.

**VS Code dev containers support is table stakes, not differentiation.** The `0.462.0-pre-release` support for wslc as the Docker path in VS Code dev containers is necessary for adoption — most Windows developers using containers are doing so through VS Code. But it's catch-up, not lead. The real question is whether wslc offers anything over Docker for the VS Code dev container use case beyond "one less thing to install." If not, it's a platform play (reduce dependencies) rather than a product play (better experience).

**The quote from a user at the end of the post is telling.** The final paragraph — a user asking how to access host web services from inside a wslc container — isn't typical blog-post structure. It reads like a support question accidentally appended, or a deliberate inclusion to show "we know there are rough edges." Either way, it signals that networking (specifically host-to-container communication) is still being worked out, which is consistent with consomme being experimental.

**For the agentic development angle:** A built-in, API-accessible Linux container runtime on Windows has implications for coding agents. If a Windows application can programmatically spin up Linux containers via a NuGet package, then a coding agent running on Windows can do the same — without installing Docker, configuring WSL, or setting up any third-party tooling. Combined with [[Dev Containers]], this makes Windows a first-class platform for container-isolated agent execution. The MSBuild/CMake integration also means agent-generated projects can declare container dependencies as build steps, which is a cleaner pattern than "the agent runs `docker build` as a shell command."

---

See also: [[Dev Containers]] (VS Code dev container spec, now supports wslc as backend), [[Windows in Docker]] (the inverse: running Windows in Linux containers), [[Container-Maker]] (devcontainer.json CLI tooling), [[Domenic Denicola's Agentic Coding Setup]] (disposable VMs for agent isolation), [[Semantic Kernel]] (Microsoft's agent middleware, same ecosystem), [[How We Contain Claude]] (Anthropic's container isolation patterns for agents).

---
*Sources: [[raw/wsl-container-is-now-available-for-public-preview]], [[summary/wsl-container-is-now-available-for-public-preview]]*
*Last updated: 2026-08-07*
