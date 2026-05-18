# Ingest Queue

URLs that failed to fetch during wiki ingestion. Wiki pages exist but were built from secondary sources or annotations — primary content would improve them.

## Retryable with surf

Sites that returned 403/Cloudflare blocks — browser automation may succeed.

| URL | Wiki Page | Failure | Date |
|-----|-----------|---------|------|
| https://www.jazzguitar.be/blog/all-of-me/ | All of Me Jazz Standard Analysis | Cloudflare 403 | 2026-05-15 |
| https://colossus.com/article/education-broligarchy-silicon-valley-canon/ | The Education of the Broligarchy | 403 | 2026-05-15 |
| https://openai.com/index/inside-our-in-house-data-agent/ | Inside OpenAI's In-House Data Agent | 403 | 2026-05-15 |
| https://openai.com/index/harness-engineering/ | Harness Engineering (OpenAI) | 403 | 2026-05-15 |
| https://blog.dochia.dev/blog/idempotency/ | Idempotency Is Easy Until the Second Request Is Different | 403 | 2026-05-15 |
| https://wesmckinney.com/blog/mythical-agent-month/ | The Mythical Agent-Month | 403 | 2026-05-14 |
| https://selfhostllm.org/ | Self-Hosted LLMs | 403 | 2026-05-14 |
| https://consultwithgriff.com/dapper-nvarchar-implicit-conversion-performance-trap | Dapper Performance Trap | timeout | 2026-05-14 |
| https://www.pragmaticsummit.com/ | The Pragmatic Summit | JS-heavy | 2026-05-15 |

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

X.com blocked, xcancel.com also failing.

| URL | Wiki Page | Failure | Date |
|-----|-----------|---------|------|
| https://xcancel.com/hedgiemarkets/status/2013417027713548718 | AI Livestream Factories | all mirrors 503/403 | 2026-05-15 |
| https://xcancel.com/alexandr_wang/status/2041909376508985381 | Muse Spark and the Rough Edges Admission | xcancel 503 | 2026-05-15 |
| https://xcancel.com/manthanguptaa/status/2015780646770323543 | Memory Is a Mistake | xcancel 503 | 2026-05-15 |

## Structural

Content exists but can't be text-extracted.

| URL | Wiki Page | Failure | Date |
|-----|-----------|---------|------|
| https://www.thoughtworks.com/content/dam/thoughtworks/documents/report/tw_future%20_of_software_development_retreat_%20key_takeaways.pdf | ThoughtWorks Future of Software Engineering Retreat | image-based PDF | 2026-05-15 |
