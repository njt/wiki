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
| https://matijacniacki.com/blog/openviktor | OpenViktor | password-protected | 2026-05-15 |

## Dead / unreachable

Domain down, 404, or cert errors. Low chance of recovery.

| URL | Wiki Page | Failure | Date |
|-----|-----------|---------|------|
| (empty — all 3 removed 2026-05-18, domains dead) | | | |

## Tweets (mirror failures)

X.com blocked, xcancel.com also failing. Resolved via personal Chrome profile + surf.

| URL | Wiki Page | Failure | Date |
|-----|-----------|---------|------|
| (empty — all 3 resolved via surf/x.com on 2026-05-18) | | | |

## Structural

Content exists but can't be text-extracted.

| URL | Wiki Page | Failure | Date |
|-----|-----------|---------|------|
| (empty — ThoughtWorks resolved via pdftotext on 2026-05-18; was misclassified as image PDF) | | | |
- (localai.io resolved 2026-07-04 — homepage+features surf grab re-ingested via ingest.sh --content; page enhanced)
