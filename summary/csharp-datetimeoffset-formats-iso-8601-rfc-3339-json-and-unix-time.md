---
url: https://sebnilsson.com/blog/csharp-datetimeoffset-formats-iso-8601-rfc-3339-json-and-unix-time
title: C# DateTimeOffset Formats — ISO 8601, RFC 3339, JSON, and Unix Time
author: Sebastian Nilsson
site: sebnilsson.com
date_fetched: 2026-08-07
topics:
  - software-engineering-craft
---

A practical field guide to formatting and parsing `DateTimeOffset` in C# for API boundaries, files, logs, and frontends. Uses a single timestamp (2026-07-14T09:11:30.123+02:00) to compare 13 formats against the only metric that matters: what survives the round trip. The table at the top is the article's thesis in one glance.

The core recommendation: send RFC 3339 with an explicit offset unless you have a specific reason to do otherwise. ISO 8601 is too broad to serve as a contract on its own; RFC 3339 is the narrow profile most APIs actually mean. The article walks through custom format strings (`"yyyy-MM-dd'T'HH:mm:ss.fffK"` for offset, `"yyyy-MM-dd'T'HH:mm:ss.fff'Z'"` for UTC), `System.Text.Json` defaults (which write UTC as `+00:00`, not `Z`), a custom `JsonConverter` for enforcing a fixed wire contract, and the three traps at the TypeScript/JavaScript boundary (`new Date` with offsetless strings, midnight-UTC date-only strings, and the seconds-vs-milliseconds unit confusion).

Unix time gets its own treatment (`ToUnixTimeSeconds`/`ToUnixTimeMilliseconds`), as do `DateOnly`/`TimeOnly` for values that aren't timestamps, and `TryParseExact` for validating multiple accepted formats. The article's closing principle: keep timestamps as `DateTimeOffset` inside the application, format only at boundaries, and pick the format based on what offset and precision it preserves — because a wrong timestamp is still valid, which is why these mistakes rarely raise an error.
