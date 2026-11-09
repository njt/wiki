---
url: https://www.eventcatalog.dev/
date_fetched: 2026-11-09
---

# Give every team a shared

understanding of your architecture

The documentation tool designed for software architecture. Document, visualize and explore your domains, systems, services, APIs and events in one connected model.

Self-hosted by design. Private by default.

## The knowledge exists. It’s just scattered.Ownership in spreadsheets. Diagrams in Miro. Schemas in registries. Context in people’s heads. Nobody can see the whole system.

### Understand your architecture in one connected model. Bring domains, systems, services, messages, APIs, owners and dependencies together so teams can see what exists and how it connects.

**Document with architecture primitives.**

Building blocks, not blank wiki pages. Document your domains, systems, services and messages as connected resources.

**Keep the context next to the code it explains.**

Attach schemas, specifications, decisions and owners to each resource, so teams find the why and who without asking around.

### See how everything fits together. Stop digging through docs to piece the system together. Start with the domains, zoom into services and messages, and get the right detail at the right time.

**Ordering**v1.0.0

**Checkout System**v1.0.0

Orchestrates checkout: reserves stock, takes payment, creates the order.

**Order Management System**v1.0.0

The source of truth for orders and their lifecycle.

**Shopper**Actor

**Payments**v1.0.0

Takes payment for orders and processes refunds.

**Fulfilment**v1.0.0

Picks, packs and ships confirmed orders.

**Checkout API**

Entry point for checkout.

ordering-platform

**Checkout Orchestrator**

Runs the checkout saga.

ordering-platform

**Order Service**

Creates and tracks orders.

ordering-platform

**Order Database**

PostgreSQL · orders

ordering-platform

**Domains, systems and actors.** The context: who uses your systems, which domain each one belongs to, and how they relate to the rest of the business.

**Move through every layer of information.**

Start with the domain view, then zoom into systems, services, and the messages they exchange.

**Follow a business process from start to finish.**

Document workflows like placing an order as flows. See every step, the services behind it, and which team owns each part.

### Plan what's next, together. Design changes on a live canvas in EventCatalog Studio, starting from the services and messages you already document. Your team and your AI agents work on it with you, and the decision stays on record.

**Talk it through on the design.**

Everyone on a canvas sees each other's pointers and edits. Comment on a node, flag a risk, and mark the design proposed or accepted.

**Your agent joins the canvas.**

Connect Claude, ChatGPT or Cursor through the MCP server. Agents add resources from your catalog and connect them while you watch.

### Documentation you can actually trust. EventCatalog keeps your catalog in sync with your code and your specs, so what you read is what's running.

**Keep documentation in sync with your code.**

EventCatalog Agents update the affected docs as code changes across your organization, and your teams review every update.

**Fits into your workflow.**

Plug EventCatalog into the CI/CD pipelines and processes you already have, with integrations, generators and the SDK.

### Your agents are guessing. Give them context. Agents can read your code, but not your architecture. EventCatalog tells them what a change touches, who owns it and why it was built that way.

EventCatalog`showResource`

**Orders Service**v1.2.0Open in EventCatalog

Orders Service accepts **PlaceOrder** and listens for **PaymentTaken**. It publishes **OrderCreated** and **OrderCancelled**, which Fulfilment and Notifications consume.

**Ask your architecture from any agent.**

Connect Claude, Cursor or any MCP client to the EventCatalog MCP server to query services, schemas and owners as you work.

**Give every team the context they need.**

Scope MCP servers to a domain or system so teams and AI tools get focused, trusted architecture information exactly when they need it.

**Start with one team.** *Skip the Confluence sprawl. Scale when you’re ready.*

**Used by teams around the world.**Self-host EventCatalog and keep full ownership of your architecture documentation and data.

**33,000+**Catalogs created

**3,000+**GitHub stars

**1,200+**Community members

**40+**Contributors

**Trusted by teams building complex systems.**What architects and engineers say about EventCatalog.

## Your architecture is a system. Document it like one.

Rolling out across many teams? Start with the Success Programme or see pricing.
