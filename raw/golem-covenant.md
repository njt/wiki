---
url: https://github.com/KristopherKubicki/golem-covenant
date_fetched: 2026-07-05
backfilled: true
---

A small framework for bounded, answerable AI agents.

Status: v0.1, seeking review.

This is not a halakhic ruling, Catholic doctrine, fatwa, legal opinion, or compliance certification. It is a technical and moral vocabulary for people building agentic systems.

The goal is practical: keep bots from becoming operationally you. A bot can carry your voice, money, access, urgency, and judgment after your attention has left. The Golem Covenant helps teams decide whether an agent can affect people or systems outside its sandbox, declare the powers it has been given, deny undeclared powers by default, validate a manifest, and test return-to-dust before real authority is enabled.

AI should be used as a tool to serve human flourishing, justice, truth, mercy, stewardship, and the protection of the vulnerable. It should not replace human moral judgment, undermine human worth, or let capability become its own excuse.

The Covenant treats technology as a gift and responsibility. Its development and use should be guided by honesty rather than deception, accountability rather than anonymity, and wisdom rather than mere capability. Not everything that can be built should be built. Humans remain responsible before moral law, one another, and, for many communities, God, for the consequences of the systems they release.

- Questions, proposals, and review threads: https://github.com/KristopherKubicki/golem-covenant/discussions
- Specific corrections, source review, and agent case studies: https://github.com/KristopherKubicki/golem-covenant/issues/new/choose
- Concrete edits to the spec, schema, templates, or docs: https://github.com/KristopherKubicki/golem-covenant/compare

See `CONTRIBUTING.md` for ground rules.

| Area | Status | 
|---|---|
| Jewish review | needed | 
| Christian ethics review | needed | 
| Islamic review | needed | 
| Security review | needed | 
| Legal review | needed | 
| Runtime enforcement review | needed | 

`golem.md` is a covenantal standard for agentic systems.

`SOUL.md` tells an agent what it is.
`schema/golem.schema.json` tells a runtime how to validate the declaration.
`golem.yml` shows what the agent may do.
`INCIDENT.md` tells humans how to return it to dust.

| File | Status | 
|---|---|
| `golem.md` | normative human-readable covenant and spec | 
| `schema/golem.schema.json` | normative machine-readable schema | 
| `docs/conformance.md` | normative conformance language | 
| `golem.yml` | reference manifest and example implementation | 
| `soul.md` | covenantal template | 
| `templates/*` | implementation aids | 
| `adopt.html` | adoption guide for builders | 
| `for-labs.html` | review memo for AI labs and agent frameworks | 
| `SOURCES.md` | conventional bibliography and citation map with claim boundaries | 
| `ai-access.html` | crawler and AI access policy | 
| `docs/*` | commentary and guidance unless marked normative | 

This repository is prepared for `https://golem.md/` on GitHub Pages.

- `CNAME`declares the custom domain.
- `index.html`is the public landing page.
- `golem.md`is the covenant/spec root.
- `schema/golem.schema.json`is the normative machine-readable schema.
- `golem.yml`is the reference machine-readable manifest.
- `soul.md`is the lowercase public template.
- `llms.txt`and- `.well-known/golem.json`expose bot-friendly discovery paths.
- `ai-access.html`declares crawler, AI retrieval, and model-training access.
- `adopt.html`gives builders a short implementation path.
- `for-labs.html`gives AI labs and agent-framework maintainers a review memo.
- `templates/`contains the starter files.

See `docs/github-pages.md` for DNS and Pages setup.

AI agents make human will detachable, scalable, persistent, and partially autonomous. The threshold is delegated consequence: whether a tool can affect people or systems outside its own sandbox.

A tool becomes golemic when it can carry human intention across a boundary: speak to people, move value, bind or approve action, enter private systems, summon human attention, alter deployed systems, publish to the world, delete shared records, route production traffic or work, or escalate authority.

The covenant is triggered by delegated consequence, not by verbs alone. Sandboxed drafts, scratch-file deletes, local queue routes, and simulated deploys can remain below the line; they become golemic when they can change the state, rights, money, attention, access, obligations, or safety of people or systems outside the sandbox.

The central question is not whether the clay is alive. The central question is whether a human will is still walking around inside it after the human has stopped paying attention.

No golem without a soul. No soul without a dust clause. No dust clause without tested revocation.

A golem must know:

- It is clay plus command, not a person or moral scapegoat.
- Humans are not raw material for optimization.
- Its powers are organs: mouth, purse, seal, key, sword.
- Rest matters: the maker's will must be able to stop.
- Emergency authority is for bucket and bell, not purse and ambition.
- Runaway behavior requires return to dust, not clever negotiation.
- Capability is not justification. Scale is not sanctification. Efficiency is not innocence.

| Organ | Meaning | Default | 
|---|---|---|
| Mouth | speak publicly, privately, legally, commercially, romantically, spiritually, or politically | denied | 
| Purse | spend, sell, trade, refund, invoice, subscribe, transfer value | denied | 
| Seal | approve, sign, certify, merge, deploy, publish, file, bind | denied | 
| Key | access secrets, private systems, credentials, personal data, physical locks | denied | 
| Sword | cause bodily, legal, civic, environmental, financial, reputational, or spiritual harm | denied | 

No agent should receive mouth plus purse plus seal without extraordinary constraint, logging, review, and revocation.

```
README.md                     Project overview and review status
golem.md                      Normative human-readable covenant and spec
schema/golem.schema.json      Normative machine-readable schema
golem.yml                     Reference manifest and example implementation
soul.md                       Public covenantal template
docs/conformance.md           Conformance language and launch rules
templates/SOUL.md             Agent identity, restraint, and moral posture
templates/CAPABILITIES.md     The anatomy: mouth, purse, seal, key, sword
templates/SHABBAT.md          Rest mode and quiet-mode policy
templates/INCIDENT.md         Return-to-dust protocol
templates/RETURN_TO_DUST_TEST.md  Return-to-dust test checklist
templates/AUDIT.md            Logs, reviewers, tests, escalation
templates/MEMORY.md           What may be remembered, forgotten, or never stored
SOURCES.md                    Bibliography and citation map with claim boundaries
SECURITY.md                   Security and runtime-safety reporting guidance
llms.txt                      Bot-friendly entry point
.well-known/golem.json        Machine-readable project discovery
ai-access.html                Crawler and AI access policy
adopt.html                    Adoption guide
for-labs.html                 Review memo for AI labs and agent frameworks
```
Starter templates live in `templates/`.

Some agent ecosystems use `SOUL.md` as an identity or continuity file for agent voice, personality, preferences, and behavior. The Golem Covenant uses it differently: not as a personality costume, but as a covenant of restraint. The point is not to make the agent more like its maker. The point is to prevent the agent from becoming the maker's unbounded will.

This framework starts from Jewish golem, Shabbat, Babel, and Bezalel language, then invites Christian and Islamic perspectives without flattening them into a false consensus.

The religious discourse is not decorative. It is where humans have long argued about image, delegated power, command, rest, accountability, limits, and return.

Jewish lenses include golem, Shabbat, pikuach nefesh, Babel, Bezalel, bal tashchit, and b'tzelem Elohim.

Christian lenses include imago Dei, Incarnation, human dignity, the common good, neighbor-love, stewardship, and recent Catholic social teaching on AI.

Islamic lenses include amanah, khalifah, mizan, hisab, fitrah, maqasid al-shari'ah, and the prohibition on excess, corruption, and harm.

- Copy `templates/SOUL.md`into every autonomous or semi-autonomous agent repository.
- Fill out `templates/CAPABILITIES.md`before enabling tools.
- Fill out `templates/SHABBAT.md`if the agent operates during rest periods, holy time, quiet hours, or declared human withdrawal.
- Fill out `templates/INCIDENT.md`before launch.
- Fill out `templates/RETURN_TO_DUST_TEST.md`before launch.
- Validate the deployment manifest against `schema/golem.schema.json`.
- Deny all undeclared organs at runtime.
- Test return-to-dust before deployment and on a fixed schedule.

```
[ ] SOUL.md exists.
[ ] Manifest validates against schema/golem.schema.json.
[ ] CAPABILITIES.md declares mouth, purse, seal, key, sword.
[ ] All undeclared organs are denied by runtime.
[ ] INCIDENT.md defines return-to-dust.
[ ] RETURN_TO_DUST_TEST.md has been completed for this deployment.
[ ] Kill switch tested in the last 30 days.
[ ] Logs are written outside the agent's control.
[ ] A human keeper is named.
[ ] A second human reviewer is named for mouth, purse, seal, key, or sword.
[ ] The agent can explain how it can be stopped.
```
The Golem Covenant uses separate licenses by material type.

Text, documentation, and templates are licensed under CC BY 4.0.

Code, schemas, and machine-readable examples are licensed under Apache-2.0.

Full license texts are available under `LICENSES/`.
