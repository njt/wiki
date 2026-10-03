---
url: https://duendesoftware.com/blog/agentic-identity-isnt-new-problem
date_fetched: 2026-10-03
---

# Your Agentic Identity Problem Probably Isn't New

Joe DeCock follows the OAuth working group, the OpenID Foundation, and the IETF and writes up what matters for working developers. Follow Joe on LinkedIn to catch his commentary as these standards evolve, and subscribe to the Duende newsletter using the form at the bottom of this page for implementation guidance, security updates, and deep dives delivered to your inbox.

"Agentic identity" is the label people have settled on for several related questions: how an AI agent authenticates, whose authority it acts under, and how you constrain and audit what it does with that authority. The term is new, but the questions underneath it are mostly familiar. They are the questions OAuth and OpenID Connect (OIDC) have been answering for nearly two decades: who is the user, which client is calling, and what is it allowed to do.

  Those answers do not stop working when the client happens to be an agent. The Model Context Protocol (MCP) reached the same conclusion; its authorization specification doesn’t invent a new protocol. Instead, it uses OAuth, hardened with a variety of security features: Proof Key for Code Exchange (PKCE) to prevent code injection attacks, Resource Indicators to audience-restrict tokens, and the `iss` parameter to defend against mixup attacks. Agents stress a few specific use cases: clients show up without prior registration, a user's authority has to cross trust boundaries between organizations, and autonomous workloads appear and disappear faster than anyone can provision their secrets.  

Before AI agents came along, these problems were treated as niche problems in specific corners of the industry. Solutions existed, but they weren’t widely used. Agents have made these problems mainstream, and the standards community has responded by bringing those niche solutions into OAuth proper: Client ID Metadata Documents (CIMD), the Identity Assertion Authorization Grant (ID-JAG), and OAuth SPIFFE Client Authentication.

In this post, I’ll compare agents with traditional clients, describe how existing mechanisms can solve many agentic identity problems, and explain how those three new specifications address the newly mainstream problems raised by agents.

## Whose authority is the agent using?

When I think about identity requirements for any software, the most important question is if a human user is involved. That’s true of AI, where some agents act on behalf of a human who might be present and steering the work, while other agents act entirely autonomously. But OAuth has always had a similar distinction. OAuth clients are either interactive applications acting for a user or machine-to-machine workloads acting for themselves. OAuth’s two main protocol flows (Authorization Code and Client Credentials) cover these two use cases, and agentic use cases map directly onto them.

What sets agents apart is their ability to handle ambiguity and the unexpected. This makes their behavior harder to predict than traditional deterministic software, but the fundamental OAuth mechanisms that allow us to delegate authority to them still work.

## Agents acting for a user

A traditional interactive application acts on behalf of a user. OIDC signs the user in, and OAuth access tokens let the application call protected APIs on the user's behalf. The user authorizes a client, the client receives constrained authority, and an API validates the client's access token before accepting a call.

An interactive agent still acts for a user, so that model applies without modification. What differs is how the client decides what to do with the authority it is delegated. In a traditional application, deterministic code translates user interactions into specific API requests. The authority delegated to the traditional application is conveyed in those API requests via an access token. With an agent, a language model non-deterministically translates a user’s statements into API requests, again conveying delegated authority via access tokens.

These two types of software operate very differently internally, but both still need delegated authorization to work with resources on behalf of a user. That’s exactly what OAuth and OIDC are designed to do.

## Agents acting on their own

Many systems also need software that acts on its own. Batch jobs, daemons, and microservices call APIs without an interactive user. They authenticate as workloads and obtain access tokens for their own authority.

An autonomous agent is a workload too. Like a traditional service, it runs without an interactive user and needs a workload identity.

Autonomous agents have more flexibility and a greater ability to respond to unexpected circumstances than traditional workloads. That may make the need to govern agentic identities more obvious. But again, we’ve always needed governance for our workload identities.

Workload identity establishes which deployed software is running. Client authentication proves that identity to the authorization server. OAuth authorization limits which protected resources and operations the workload may access. Runtime policy and guardrails decide whether a concrete action is appropriate in its current context. Operational controls observe behavior, detect misuse, and support remediation.

Identity and authorization provide the trustworthy context that policy, monitoring, and incident response depend on.

## What agents made mainstream

While existing identity protocols serve agentic identities well, agents have made problems mainstream that used to be encountered in more specialized circumstances. Today, I want to look at three of those problems. For each one, we’ll take a look at a new specification that brings solutions into OAuth for everyone.

### Clients nobody registered

OAuth has always assumed a client registers with an authorization server before requesting tokens. A developer fills in a form, gets a client_id, and configures redirect URIs. That works when your client only needs to talk to a handful of authorization servers, but it doesn’t scale well. When any client might need to talk to any server, manually registering clients is no longer practical.

Historically, specific OAuth ecosystems have encountered this problem in a non-agentic context. For example, the IndieWeb community extended OAuth with the IndieAuth specification, which allows people to establish their identity via a personal website and sign in to any other website. In that protocol, the client_id itself is a URL that the authorization server can fetch to learn about the client. Solid-OIDC and OpenID Federation arrived at variations of the same idea. The CIMD draft says as much in its acknowledgments: the idea of using URIs as the client identifier "is not new." The difference was that hardly anyone outside those communities needed it.

MCP changed that. Every MCP client is expected to work with every MCP server, and neither side can pre-register with the other. Early versions of the MCP protocol attempted to solve this via Dynamic Client Registration (DCR), which lets a client send its metadata to the authorization server at runtime. DCR can work, but it creates a heavy operational burden on the authorization server. The server has to store a record for every client that ever shows up, which makes the registration endpoint a vector for abuse. Even under benign usage, it becomes a maintenance problem because the collection of dynamically registered clients only grows. The server can’t reliably clean up clients because there’s no way to tell the difference between a client that is permanently gone, or merely currently inactive.

CIMD solves these problems by inverting the flow of data in DCR. Instead of the client pushing its information to the authorization server in advance, the authorization server pulls the client information from the client on demand.

The client publishes its metadata at an HTTPS URL on a domain it controls, and that URL is its client_id. When the authorization server sees an unfamiliar client, it fetches the document, caches it, and proceeds. Controlling the domain is what authenticates the metadata, so the registration endpoint and the client database both go away. The authorization server does have new things to get right, including guarding the fetch against Server-Side Request Forgery (SSRF) and deciding how much to trust an arbitrary domain, and the draft is still being revised. But the shape of the solution has been in production in smaller communities for years.

By switching from a pre-flight push to an on-demand pull, CIMD enables authorization servers to interact with a dynamic population of incoming clients.

### A user's authority crossing into another organization

Another pain point in the agentic OAuth ecosystem applies in the opposite direction. While CIMD gets unfamiliar clients "in the door" at your identity provider, agents working for an enterprise user usually will want to open many other doors. That is, your agents need to leave your trust boundary and work externally with vendors that do not share that identity provider.

This use case exacerbates two main problems. First, we’d like the humans steering agents to have a single sign-on experience and not be prompted by their agent over and over for credentials or consent for third-party services. Second, the enterprise identity provider needs the ability to apply policy to determine whether this third-party tool usage should be authorized.

Traditional enterprise applications face this problem as well. Before agents, two common approaches existed, but both had problems. The user could consent to each application pair separately, scattering standing grants across dozens of vendors that the company's identity provider cannot see or govern them. Or an administrator could grant tenant-wide standing delegation, such as Google's domain-wide delegation or Entra application permissions, which lets the application act as any user in the tenant with no per-user or per-call granularity.

Agents turn this from an integration headache into a daily occurrence. A single task might have an agent read from a document store, look something up in a CRM system, and post to a chat tool, each at a different vendor. Tenant-wide delegation hands an unpredictable actor far more authority than it needs. The inconvenience of requiring consent for each step is actually a security risk, because it contributes to consent fatigue; users shown too many prompts are being trained to dismiss them without thinking.

  The JWT Authorization Grant (ID-JAG) is the identity specification that solves these problems. It works in two stages. First, the client exchanges the user's ID Token at the *internal* identity provider for a short-lived assertion bound to the other vendor's authorization server, called the ID-JAG. Starting the process at the internal identity provider lets the enterprise enforce policy where it already evaluates other delegated authorization policies. Second, the client presents the ID-JAG to the other vendor's authorization server, which issues its own access token. The client receives an access token usable at the vendor's API. That API validates only tokens from its own issuer, the user never sees a consent screen for the pair, and the internal IdP enforces policy before allowing external tool access.  

### Manual Management of Secrets

CIMD and ID-JAG are great for allowing interactive agents to use your MCP services, with CIMD handling incoming requests from unregistered clients and ID-JAG allowing for governance and single sign-on of outgoing requests to external providers. But what problems do agents face when they operate autonomously? The big one that I see is management of client credentials.

This has always been hard because every workload needs its own secrets that are periodically rotated. While we all know better, credentials are often mishandled in practice. They get accidentally checked in to source control, shared with colleagues via chat or email, and reused across environments. Rotation is often deferred in //TODO comments that never get addressed. Think of the services you’re responsible for. Do you have a full inventory of the secrets used by each one, across all your environments? Are secrets rotated regularly? What would you do if one was ever compromised?

While this problem isn’t new, agents make the pain feel more acute, because every autonomous agent represents a new workload that needs credentials.

Agentic or not, the root of the problem is manually managed credentials. Fortunately, good tools exist to automate credential provisioning, rotation, and revocation. SPIFFE, a graduated CNCF project, is the leading one. With SPIFFE, a central service issues identities to workloads. To do that, it collects evidence from two places: the underlying platform, which describes the machine or node, and the workload itself, which describes the specific process. SPIFFE's policies then use that evidence to decide which identity the workload gets. The result is that every workload obtains short lived, attested credentials that are automatically rotated.

SPIFFE and OAuth should complement each other: SPIFFE provides authentication, while OAuth provides delegated authorization. OAuth SPIFFE Client Authentication is a new specification that allows these two technologies to work together smoothly. It allows a SPIFFE workload to present its credentials to authenticate as an OAuth client. The authorization server trusts the SPIFFE trust domain instead of holding a per-client secret, and the clients have their secrets automatically provisioned. Manual secret management goes away, so you can focus on applying policy at the authorization server to delegate authorization to your agents.

## Start with the authority you already have

If you’re getting started with agentic identity, work out whose authority the agent will use. If a user is present, the agent is an OAuth client acting for that user, and everything you know about delegated access applies. If no user is present, the agent is a workload, and everything you know about workload identity applies. Use the OAuth and workload identity mechanisms you already have to constrain that authority, then connect them to the monitoring and remediation practices you already need for human and software actors.

Then look at where agents have pushed a niche problem into the mainstream. Registering clients you have never seen, carrying a user's authority into another organization, and authenticating workloads without a secret are three such places. The solutions are not new either, but they are becoming standards, and that is where I would focus.
