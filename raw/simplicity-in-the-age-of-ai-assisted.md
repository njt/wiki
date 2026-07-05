---
url: https://gogogolems.substack.com/p/simplicity-in-the-age-of-ai-assisted
date_fetched: 2026-07-05
backfilled: true
---

# Simplicity in the age of AI-assisted coding

### On complexity, judgment, and building only what the problem requires

A key attribute of good software has always been simplicity; the ability to say: this is the problem, and this is the smallest thing that solves it. The layers of technology we’ve added since the early days of computing of are mostly overhead we’ve added for ourselves, not for the problem.

**LLMs don’t change what good programming looks like, but they require us to change our relationship to simplicity.**

Let’s take a simple example: a user wants to be able to disable the sticky header on a static content website. That’s it. That’s the whole feature.

## Four ways to say the same thing

The user expresses their desire in natural language:

I have a really hard time navigating the site because I keep having to scroll back to the top to access navigation.


The product manager writes a ticket, who knows that other users want to hide the header because it eats up too much space:

As a user, I want to toggle the sticky header on or off, so that I can either reclaim vertical screen space, or easily navigate the site while reading long documents


The product manager then suggests using a settings file:

```
header:
  sticky: true
```
The CSS designer, translating that into the necessary CSS:

```
.header { position: sticky; top: 0; }
```
The frontend developer would add the actual toggling:

```
if (settings.header.sticky) {
  document.getElementById(”header”).classList.add(”sticky”);
} else {
  document.getElementById(”header”).classList.remove(”sticky”);
}
```
The backend developer who thinks in computation, not constraints (me – I always struggle with CSS and wouldn’t even know about the `position: sticky` attribute) could do the whole thing like this:

```
if (settings.header.sticky) {
	<!-- sticky: header outside scrollable area -->
	<body>
	  <header>...</header>
	  <article class=”scrollable”>...</article>
	</body>
} else {
	<!-- not sticky: header inside scrollable area -->
	<body>
	  <div class=”scrollable”>
	    <header>...</header>
	    <article>...</article>
	  </div>
	</body>
}
```
All of these produce the same pixels on the screen. But they exist in completely different linguistic worlds, adapted to different brains, different roles, different ways of thinking about the problem.

## Why all four exist

Each notation corresponds to a human layer. The PM thinks in user stories and scoping – deciding it should be a setting is itself a design choice, a way to express the idea to the user in the simplest terms; the designer thinks in visual constraints and layout systems, which makes CSS a reasonably well adapted language for their brain; a programmer (me) is much more used to computational thinking: what instructions get executed, what data gets modified, what state changes (which maybe explains why developers struggle with CSS so much and deride it as a poor language – it doesn’t make for computational expressibility, because it was never trying to).

**The layers exist for communication, not computation.**

The computer doesn’t need three interrelated ways of doing “sticky header on or off.” Humans do, because we’ve organized ourselves into specialties, and each specialty has developed its own language. CSS exists so designers can talk to browsers without thinking about instruction pointers, but instead in terms of layout constraints and design systems. User stories exist so PMs can talk to end-users and to a technical team both. Each language lets its users offload cognitive burden onto notation that fits their brain, while also making it possible to communicate across the boundaries – the designer’s encoding of the request is very similar to the PM’s, even though the notation carries an iceberg of complexity underneath.

So we end up with layers of language that correspond to human structures: a human communicates their desire, the PM knows how to formulate it as a feature, the designer knows how to make it happen with a CSS class, the developer knows how to set that class on the header when the setting is true, or to emit different markup. Each translation step works, but each one is also a potential point of miscommunication, which is why we iterate – making sure the linguistic telephone line actually produces the right pixels at the end.

## The complexity explosion

Now the PM says: “the setting should persist across devices.”

Suddenly we need a database schema:

```
CREATE TABLE user_preferences (
  user_id UUID PRIMARY KEY,
  preferences JSONB NOT NULL DEFAULT ‘{}’,
  updated_at TIMESTAMP DEFAULT NOW()
);
```
A migration system to manage that schema over time:

```
// migrations/20260301_add_user_preferences.js
exports.up = (knex) =>
  knex.schema.createTable(”user_preferences”, (t) => {
    t.uuid(”user_id”).primary();
    t.jsonb(”preferences”).defaultTo(”{}”);
    t.timestamp(”updated_at”).defaultTo(knex.fn.now());
  });
```
An API endpoint:

```
app.put(”/api/preferences”, auth, async (req, res) => {
  const { sticky } = req.body;
  await db(”user_preferences”)
    .insert({
      user_id: req.user.id,
      preferences: { header: { sticky } },
    })
    .onConflict(”user_id”)
    .merge();
  res.json({ ok: true });
});
app.get(”/api/preferences”, auth, async (req, res) => {
  const row = await db(”user_preferences”)
    .where({ user_id: req.user.id })
    .first();
  res.json(row?.preferences ?? {});
});
```
A frontend fetch to call that API:

```
const prefs = await fetch(”/api/preferences”, {
  headers: { Authorization: `Bearer ${token}` },
}).then((r) => r.json());
if (prefs.header?.sticky) {
  document.getElementById(”header”).classList.add(”sticky”);
}
```
A config for the database connection:

```
# config/database.yml
production:
  host: db-prod-cluster.internal
  port: 5432
  database: app_production
  pool: { min: 2, max: 10 }
  ssl: true
```
Monitoring, because now the database can go down and take the sticky header with it:

```
# datadog/monitors.yml
- name: user-preferences-db-latency
  type: metric alert
  query: avg(last_5m):avg:postgresql.query.time{service:user-preferences} > 500
  message: “Preferences DB latency > 500ms. @devops-oncall”
```
A Dockerfile, because the API needs to be deployed somewhere:

```
FROM node:20-slim
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
EXPOSE 3000
CMD [”node”, “server.js”]
```
Kubernetes config, because we need orchestration:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: preferences-api
spec:
  replicas: 2
  template:
    spec:
      containers:
        - name: api
          image: registry.internal/preferences-api:latest
          ports:
            - containerPort: 3000
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: url
```
We now have SQL, a migration DSL, JavaScript, HTTP, JSON, YAML config, a monitoring DSL, Docker, Kubernetes. We have a devops person to manage the cluster. A DBA, or at least someone who knows about connection pools and migrations. A backend developer for the API. The frontend developer. The designer. The PM.

**All of this to remember whether the header should be sticky.**

The base problem – persistence across devices – is genuinely hard. But the solution we reach for introduces a massive amount of incidental complexity: a database, a distributed system with network calls, a deployment pipeline, monitoring, on-call rotations. And once it’s there, it never leaves.

## The vicious cycle of complexity

Here’s where it gets self-perpetuating.

We have a database, so we need someone who understands databases. We have Kubernetes, so we need someone who understands Kubernetes. We have a frontend that talks to an API that talks to a database that runs in a container that’s orchestrated by a cluster – so we need a frontend developer, a backend developer, a DBA (or at least someone who can write migrations without breaking prod), and a devops engineer to keep the infrastructure alive. We need a PM to coordinate all of them. We probably need a QA person because changes now cross so many boundaries that no single person can hold the full picture in their head.

Each of these roles is skilled and necessary *given the architecture*. That’s the key phrase. Given the architecture.

Each role creates complexity, in fact each role will argue for the necessity of said complexity. The devops engineer, reasonably, introduces Terraform to manage the infrastructure as code. The backend developer, reasonably, adds an ORM because writing raw SQL across a growing codebase is a maintenance nightmare. The frontend developer, reasonably, adds a state management library because the preferences now come from an async API call and you need to handle loading and error states. The DBA, reasonably, adds connection pooling and read replicas because the preferences table is now queried on every page load. The QA person, reasonably, needs a staging environment that mirrors production, so now we have two Kubernetes clusters.

Each decision is reasonable. Each one adds another language, another config file, another thing that can break, another thing someone has to understand. And each one deepens the need for the role that introduced it.

So you end up with a team of eight people and a mass of infrastructure, and when a new person joins they look at the stack and think: this is how you build a web application. They don’t question the database for storing a boolean, because there are migration files and monitoring dashboards and API tests – clearly this is important, clearly this is load-bearing. The new person adds their own reasonable layer on top, and the cycle continues.

**The architecture encodes its own legitimacy.**

I think we underestimate how much of what we consider “professional software engineering” is just this cycle, ossified into best practices. The twelve-factor app; microservices; CI/CD pipelines with fourteen stages. These aren’t wrong – they solve real problems that emerge when teams of humans coordinate changes to complex systems over time. But the problems they solve are often caused by the previous generation of solutions to the same coordination problem. We had monoliths, which were hard for large teams to work on simultaneously, so we got microservices, which introduced distributed systems problems, so we got service meshes and observability platforms and platform engineering teams, which introduced their own coordination overhead.

The complexity justifies the roles. The roles produce the complexity.

The structure perpetuates itself not because anyone is doing anything wrong, but because each individual decision is locally reasonable. **It’s just that nobody ever steps back and asks: are we still talking about a sticky header?**

## Enter LLMs

### They experience the same spectrum

An LLM dealing with `sticky: true|false` in a config has a trivial job. Ask it to flip the default, done.

Ask it to modify the CSS and it’ll probably get it right, especially if the classes are named clearly. But CSS is a constraint system – the effects of a change are often non-local and invisible in the code itself. The LLM can’t “see” layout, which is why tools like Playwright exist to give it screenshots as feedback. The language is legible to designers precisely because it maps to visual thinking, but that visual model is mostly absent from the text the LLM actually processes.

Ask it to trace through the full stack – the frontend fetch, the API endpoint, the database schema, the migration, the Kubernetes config – and it starts to struggle. Not because any single piece is hard, but because the assumptions are spread across many files and many languages. The sticky header boolean travels through HTTP, JSON, SQL, JavaScript, YAML, and Docker before it reaches the CSS class that actually does the work. The LLM has to hold all of that context and understand the conventions at each layer.

This isn’t a coincidence. The legibility spectrum applies to LLMs exactly as it applies to humans. The clearer the language, the better both humans and LLMs perform. That’s the same property: communicability.

### They inherit the cargo cult

Here’s the thing I think people don’t appreciate enough:

**the LLM will not question your architecture.**

If your codebase has a database and an API layer for user preferences, and you ask the LLM to add a new preference, it will write a migration. It will add an API endpoint. It will update the frontend fetch. It will add monitoring. It will generate maybe 200 lines across 8 files, because that’s what the existing patterns tell it to do.

It’s been trained on millions of repos that do exactly this; it is the ultimate cargo cult participant. In fact, it often *adds* unnecessary complexity and will convincingly argue for it: just like a human would, it will bring up fair, reasonable points.

Here’s what an LLM would do in a complex codebase:

```
prompt: “add a preference for the user to choose between
         light and dark mode”
# in the complex codebase, the LLM generates:
# - migrations/20260315_add_theme_preference.js  (12 lines)
# - routes/preferences.js – modified              (15 lines)
# - api/preferences.test.js – modified            (25 lines)
# - frontend/hooks/usePreferences.js – modified    (8 lines)
# - frontend/components/ThemeToggle.jsx           (35 lines)
# - config/datadog/monitors.yml – modified         (6 lines)
# - docs/api.md – modified                        (12 lines)
#
# total: ~113 lines across 7 files to store one more boolean
```
However, give the LLM the same prompt, in a codebase where preferences are a simple local-first store with sync:

```
prompt: “add a preference for the user to choose between
         light and dark mode”
# LLM generates:
# - config/preferences.yaml – modified:
#     theme: light|dark
# - components/ThemeToggle.jsx  (18 lines)
#
# total: ~20 lines across 2 files
```
The first case could easily take half a million tokens of opus-4.6, and be hard to review. The second can be done by a 7B model in 200 tokens, and is almost trivial to verify. Not because the task is different, but because the prompt maps almost 1:1 to the formal notation, and because there is no technical complexity.

### Simplicity is the real unlock

There are several reasons that complexity shouldn’t have been there.

The most common: *the complexity was assumed from the start*. A webapp needs a database – that’s just what you do. Nobody sat down and asked “do we actually need persistence for this?” The database was in the project template, or it was the first thing the backend developer set up because that’s how you start a project. The sticky header preference got a database row not because someone decided it needed one, but because there was already a database and a preferences table seemed like the obvious place to put it. The complexity wasn’t introduced to solve a problem. It was there before the problem existed.

The second reason: *the complexity was needed once, but isn’t anymore*. This is almost impossible to infer from the code itself, especially once the self-perpetuating cycle has set in. The monitoring dashboard has alerts; the migration history has twenty entries; the API has tests; everything *looks* load-bearing. But maybe the feature that required real-time sync was removed six months ago, and the database is now just storing booleans that could live in a cookie. Nobody removes it because nobody knows it’s safe to remove, and the cost of being wrong is a production incident, while the cost of keeping it is just... the ongoing weight of everything and the justification for your job.

The third: *a new technology solves the problem at a different level*. Maybe the browser now has a sync API; maybe the platform your app runs on handles user preferences natively; maybe edge storage or local-first architectures have matured to the point where the entire backend layer for preferences is redundant. The database and the API and the Kubernetes config are still running, still monitored, still maintained - solving a problem that no longer exists, because nobody made the connection between the new capability and the old architecture.

A moment of reflection (and this has to be human reflection, because the LLM will pattern-match to its training data, which is millions of repos that all have databases; beceause it will pattern-match to the existing codebase and do so myopically) might reveal that persistence isn’t actually needed; maybe the user wants different settings on their phone anyway; maybe “remember my preference” is a feature nobody asked for and the PM assumed; maybe local storage is fine for 99% of users and the cross-device case isn’t worth a distributed system; or maybe a less orthodox choice would get you there: a simple file sync, a browser extension, a platform-level preference system. These aren’t decisions an LLM will make on its own, because they require stepping outside the conventions of the codebase and the industry.

**The hard part is the human part: recognizing which complexity is essential and which is inherited.**

The LLM can’t do that, because the conventions that created the complexity are the same conventions it was trained on. But once a human makes that call, the LLM makes acting on it almost free.

### LLMs allow us to throw complexity away

Disposability gets a lot of attention in the LLM-coding conversation – the idea that because LLMs generate code fast, you can throw things away and regenerate. And that’s true and useful. But disposability is a means, not the end.

This is where LLMs actually help, but not in the way people usually mean. I don’t mean just “LLMs can generate code faster” - the real value is that regenerating a simpler version is cheap. If a human can step back and say “we don’t need the database for this,” the LLM can produce the simpler version in minutes – not the weeks of careful refactoring it would take a team to excise a database from a running system without breaking anything. The LLM doesn’t remove the complexity through careful surgery, adding more layers of backwards compatibility and feature flags. It just builds the thing again without the complexity, because it was never aware of it in the first place.

**What LLMs actually give you is the possibility of simplicity: the removal of complexity that shouldn’t have been there.**

## Back to the sticky header

Remember the user’s original request: *I have a hard time navigating the site because I keep scrolling back to the top.* That’s a person describing a problem with their experience. Somewhere between that sentence and a Kubernetes deployment manifest, we stopped solving their problem and started solving ours.

The stack — the database, the API, the migrations, the monitoring, the on-call rotation — exists because humans needed to coordinate with other humans. If LLMs collapse that coordination, then the layers to remove are the ones that were only ever about us, not about the problem. What’s left will be simpler, clearer, and closer to what the user actually asked for. Maybe it’s `position: sticky` and a cookie. Maybe it’s even less.

That’s not a new idea. It’s the oldest ideas in programming. But for decades, acting on it meant mass refactoring, rewriting, convincing eight people to delete their own work. Now the cost of starting over with less is close to zero — which means the only thing standing between you and simplicity is the willingness to look at a running system and ask: *is this load-bearing, or is this just a sticky header?*
