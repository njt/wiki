---
url: https://mainbrella.com/blog/lovable-style-builder-cloudflare-inference/
date_fetched: 2026-10-11
---

# Lovable style Builder via CF Inference GLM-5.3, Kimi K2.7 Code, and GLM-5.3 Flash

We’ve built an app builder with the workflow familiar from Lovable: describe what you want, inspect a working preview, and keep talking to the agent to change it. Mainbrella Build runs GLM-5.3, Kimi K2.7 Code, and GLM-5.3 Flash through Cloudflare Workers AI. We wanted Git history from the first build without asking a new user to get a GitHub account, so we store the source and Git bundles in our own R2 bucket.

Cloudflare describes the inference part as “Run Serverless inference on GPUs”. We send the model a request; Cloudflare operates the GPU infrastructure that answers it. Around that inference call, we built the tools to edit source, run a compiler, recover from an interrupted attempt, and retain the result after the build machine goes away.

## From a prompt to a checked preview

Suppose you want a small expense tracker. In Build, describe it, answer two optional questions about the kind of app and its visual style, and follow the conversation as the agent writes React and TypeScript. Ask for another view, a CSV export, or a different layout, and it continues working on the same app.

The backend invokes the model through `env.AI.run(model, payload)`, Cloudflare’s Workers AI binding. The request includes conversation history, tool definitions, and the selected reasoning settings. The model can request operations such as reading a file, replacing a file, running a build command, or generating an image. Mainbrella validates and executes those requests, returns their results to the model, and continues the loop. Cloudflare Workflows coordinates the build steps; the Linux guest runs the project’s toolchain.

There’s an easy mistake to make here: treat the model’s “done” as evidence that the application works. Our platform runs its own check after the model finishes. It installs dependencies, then invokes the TypeScript compiler and Vite directly:

```
# Inside the app's Linux workspace
./node_modules/.bin/tsc --noEmit
./node_modules/.bin/vite build
```
The actual check joins those commands so a TypeScript failure stops compilation. Calling the tools directly matters: the app’s generated `package.json` can’t redefine `npm run build` to print “success” and pass. Compiler errors go back into the conversation for another repair attempt. Once the check passes, we serve the compiled `dist` directory for the preview. You’re looking at the output of a build, rather than relying on a development server that might tolerate errors. The build workflow implements that boundary.

A clean compilation doesn’t establish that an expense tracker calculates everything correctly. You still need to try the behavior you care about. It does give “built successfully” a meaning independent of what the model said.

The model menu currently offers GLM-5.3, Kimi K2.7 Code, and GLM-5.3 Flash through Cloudflare Workers AI. We expose their supported reasoning choices; Kimi’s reasoning is fixed, while the GLM options offer high and max effort. If the app needs original imagery, the agent can use FLUX. Those images become local assets, served in the preview and included in source exports. A plain expense table doesn’t need a picture just because image generation is available.

The conversation also needs to explain an interrupted attempt. We added an operation journal that retains command identities, output, and the source associated with an operation. A compiler exit code, a truncated model response, and a disconnected stream are different outcomes. If the stream ends halfway through a tool request, we don’t execute the partial arguments. If execution might have started but no result was retained, the journal records the uncertainty.

## Git history without a GitHub signup

Once a conversation starts changing the app, we need to check the code into Git. “Add reporting” is an experiment, and the next request might be “put it back.” A pile of overwritten files gives the user no useful way to inspect that experiment or recover the version they liked.

We didn’t want the first app idea to turn into a GitHub signup and a request for repository permissions. So we built our own storage for the repository using Cloudflare R2. Native Git runs in the Linux machine. R2 keeps source objects, generated assets, and Git bundles; D1 keeps the version records and current head. The repository belongs to the Mainbrella app from the start, with no GitHub connection required.

The source storage layer addresses files by their SHA-256 content hash under a prefix scoped to the account and app. A manifest maps filenames to those stored objects. This lets the dashboard read the files without a running guest, and lets a later build reconstruct the source in a fresh machine. Our expense tracker’s preview can expire while its source remains available to browse, inspect, or export.

The interface offers a source ZIP and a downloadable Git bundle: a file Git can clone into a normal repository. You can inspect versions, review which files changed, and restore earlier work inside Mainbrella, or take the history with you.

Restoring is particularly useful when a follow-up gets too ambitious. Say version A is the small tracker you wanted, and version B adds a complicated reporting interface. Restoring A creates a new commit containing A’s files, following B in history. B is still there if you later decide its report was worth keeping.

Making that history portable produced an interesting implementation change. The first Git implementation saved a full-history bundle for each new version. A long conversation would repeatedly upload the same ancestors. The new version stores incremental bundles: the initial history once, then the objects needed for each subsequent commit. An unchanged save reuses its parent’s bundle.

Downloading shouldn’t require starting a paid machine. The Worker assembles a self-contained bundle from the retained objects. There’s a subtle Git constraint here: restoring old files can cause an object to appear again in a later pack, with a different delta representation. Those packs can’t safely be combined by simply concatenating their object sections. We disabled delta packing for increments, retaining ordinary zlib compression, so the export can combine whole compressed objects and recalculate the pack’s object count and checksum. This trades some compression efficiency for an export that needs no running sandbox.

The snapshot includes the dependency lockfile and referenced generated images, and excludes `node_modules`, `dist`, and environment files. The platform publishes the new history head only after the stored objects exist. Once you download the repository, the handoff is ordinary Git:

```
git clone -b main mainbrella-app.bundle expense-tracker
cd expense-tracker
git log --oneline
npm ci
npm run dev
```
You can keep working in your editor or host the app elsewhere. Retained project storage has its own funding and retention rules; separating it from a guest’s lifetime doesn’t mean unlimited storage forever. It means the guest can stop without being your only copy of the app.

## Start with a repository instead

You may already have the code. At mainbrella.com/try, paste a GitHub URL and prepare a setup prompt for ChatGPT, Claude, or your coding agent. The agent inspects the project and proposes a runtime, working directory, installation command, start command, and port. You review the configuration before starting a machine.

A branch name isn’t enough to describe what you ran. The launcher resolves the repository to a specific commit and checks out that commit on the guest. If installation fails, the terminal and failed command output remain available so you can repair that checkout. Refreshing the page doesn’t automatically rerun a failed setup. The launcher’s recovery rules keep the same machine and command identities through that debugging work.

Private repositories use **Mainbrella Import**, a separate GitHub App intended for read-only access to repositories you select. Importing your project doesn’t require permission to push to it. Access checks use the intersection of your GitHub access and the Import app’s access, rather than trusting an installation ID submitted by the browser.

The private import implementation fetches the Git objects through the backend and transfers them to the guest. The guest reconstructs a shallow checkout at the selected commit; it receives source, without receiving the GitHub access token. You can make local edits and commits there, but pushing needs a separate write-capable connection.

One distinction is easy to miss: connecting Import gives Mainbrella access to the repository. It doesn’t give ChatGPT or Claude access. The agent preparing the setup still needs its own authorized way to inspect the source, such as your existing checkout.

## Let a service keep running

A temporary machine is a good place to try a repository or compile a prototype. It’s an awkward place to host the API a friend expects to use tomorrow. We’ve introduced two lifecycles, exposed separately in the dashboard:

| Lifecycle | What keeps it running? | 
|---|---|
| Ad Hoc | A temporary session, bounded by idle and hard deadlines. | 
| Production | Renewable compute reservations, available funding, and your spending allowance. | 

Production doesn’t mean reserving an infinite lease and hoping billing catches up. The account reserves compute in five-minute increments and extends the runtime’s deadline only after securing the next reservation. The runtime enforces the deadline even if the account coordinator stops sending heartbeats. Leaving the dashboard open is unnecessary; funding and the monthly spending cap determine how long the service can run.

There are also two different failures to handle. If the application process exits, a shell supervisor attempts to restart the startup command, with a delay that grows to at most 60 seconds. If the runtime itself stops, the account can attempt recovery using a fresh reservation, the pinned image, and the saved startup command. The logical service identity survives that runtime recovery. These mechanisms are in the production lifecycle change.

**Recovery starts from the image and startup command.** Unsaved local files and RAM don’t come back. For our expense tracker, records that need to survive belong in external durable storage; a SQLite file written only to the guest would be lost on recovery. This first version has no persistent disk, replica failover, HTTP health monitoring, or uptime SLA. Machine readiness also doesn’t establish that your application is answering HTTP requests. These are the current limits of the production design, whose live rollout needs qualification.

## Keep the address when you change the app

A development preview provides a temporary, isolated URL without DNS setup. Possession of its address grants access, and the grant expires. That’s useful while building. A bookmarked application needs an address attached to the project.

Projects add a stable public endpoint. You publish a running machine and application port behind that endpoint, then change the binding when you want a different application to serve it. The project’s address stays the same. A project endpoint doesn’t start a machine or extend its lease, and publishing a replacement generation is an explicit operation.

The project gateway has a different transport policy from a development preview. Public applications can use their own Authorization headers, cookies, streaming responses, and WebSockets. The gateway removes platform credentials and prevents response cookies from claiming a shared parent domain. Your app still needs its own authentication if it serves private information.

Custom domains alias that same endpoint. The management flow checks an account-and-project-specific TXT record to prove ownership, then routing DNS and HTTPS certificate readiness before serving the hostname. Entering a domain in a project’s metadata doesn’t publish it. Default endpoint support and custom-domain support are deployment capabilities, and configured support alone doesn’t establish completed live DNS and TLS qualification.

For a first try, build a small app and download its repository. Or launch a GitHub project you’ve wanted to inspect. When you decide to leave something running, put its durable data outside the guest, make startup reproducible, and publish it through a project endpoint. The useful test is whether you can change the machine without losing either the work or the link you gave somebody.
