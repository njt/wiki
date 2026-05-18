# Ingest Queue

URLs that failed to fetch during wiki ingestion. Wiki pages exist but were built from secondary sources or annotations — primary content would improve them.

## Retryable with surf

Sites that returned 403/Cloudflare blocks — browser automation may succeed.

| URL | Wiki Page | Failure | Date |
|-----|-----------|---------|------|
| (empty — all 9 resolved via surf on 2026-05-18) | | | |

## Paywall / auth required

Need subscription, login, or password to access.

| URL | Wiki Page | Failure | Date |
|-----|-----------|---------|------|
| https://www.nytimes.com/2025/12/03/magazine/chatbot-writing-style.html | Why Does AI Write Like That | NYT paywall | 2026-05-15 |
| https://www.theatlantic.com/technology/2026/01/america-polymarket-disaster/685662/ | America Is Slow-Walking Into a Polymarket Disaster | Atlantic paywall | 2026-05-15 |
| https://matijacniacki.com/blog/openviktor | OpenViktor | password-protected | 2026-05-15 |
| https://www.sciencedirect.com/science/article/pii/S0016328719303507 | (unknown — check raw) | ScienceDirect paywall | 2026-05-14 |

## Dead / unreachable

Domain down, 404, or cert errors. Low chance of recovery.

| URL | Wiki Page | Failure | Date |
|-----|-----------|---------|------|
| https://1lib.sk/ | 1lib | ECONNREFUSED | 2026-05-14 |
| https://designsystems.igorschwarzmann.com/ | Igor Schwarzmann Design Systems | ECONNREFUSED / dead | 2026-05-15 |
| https://www.getseer.dev/blogs/pre-commit-linting-vibe-coding | Pre-Commit Lint Checks | domain repurposed | 2026-05-14 |
| https://benchling.engineering/fragmentation-to-framework-spec-first-development-at-benchling-9b97302bddcf | Spec-First Development at Benchling | bad TLS cert | 2026-05-14 |
| https://www.damtp.cam.ac.uk/user/tong/fluids/lowreynolds.pdf | (fluid dynamics PDF) | cert error | 2026-05-14 |
| https://github.com/harperreed/dotfiles/blob/master/.claude/skills/summarize-meetings/SKILL.md | (Harper Reed SKILL.md) | GitHub 404, possibly moved | 2026-05-14 |

## Tweets (mirror failures)

X.com blocked, xcancel.com also failing. Resolved via personal Chrome profile + surf.

| URL | Wiki Page | Failure | Date |
|-----|-----------|---------|------|
| (empty — all 3 resolved via surf/x.com on 2026-05-18) | | | |

## Structural

Content exists but can't be text-extracted.

| URL | Wiki Page | Failure | Date |
|-----|-----------|---------|------|
| https://www.thoughtworks.com/content/dam/thoughtworks/documents/report/tw_future%20_of_software_development_retreat_%20key_takeaways.pdf | ThoughtWorks Future of Software Engineering Retreat | image-based PDF | 2026-05-15 |
