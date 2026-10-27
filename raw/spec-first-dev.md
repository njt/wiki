---
url: https://spec-first.dev/
date_fetched: 2026-10-28
---

I want a small todo app. Add todos, tick them off, filter open and done. Browser only, no backend.

              **Read** .spec-first/memory/MEMORY.md

              **Read** src/main.ts

              It's an empty Vite starter, and I'll use TypeScript and Vitest as you prefer. Should todos survive a reload?

              Yes, localStorage is fine.

              /spec-first:create-spec

              **Write** 2026-10-27-7b2d-todos-first-version.yaml

              Here is the spec. Shall I put it on its own branch?

              Looks right, go.

              **Bash** git worktree add -b spec/2026-10-27-7b2d …

              Spec 2026-10-27-7b2d is on its own branch: 3 steps, 3 done criteria.

              /spec-first:review-spec

              **Bash** check-evidence.py evidence resolves: 1 spec

              **Agent** Review spec 2026-10-27-7b2d fresh context

              **Edit** 2026-10-27-7b2d-todos-first-version.yaml

              BUILD WITH CUTS. Drag to reorder is out, a check for empty titles is in. Both applied.

              /spec-first:apply-spec

              Plan: the store first, then the list, then the tests. Go?

              Go.

              **Write** src/todos.ts

              **Write** src/TodoList.ts

              **Write** src/todos.test.ts

              **Bash** npm test 9 passed

              **Write** .spec-first/decisions/2026-10-27-7b2d.yaml

              Built and tested. One decision on record: localStorage over IndexedDB, because a few hundred todos fit easily.

              /spec-first:deliver-spec

              **Bash** git commit -m "feat: a first todo app (2026-10-27-7b2d)" gate: build, test passed

              **Bash** gh pr create #12

              **Bash** npm run dev localhost:5173

              The gate passed and pull request #12 is open. The app runs on localhost:5173.
