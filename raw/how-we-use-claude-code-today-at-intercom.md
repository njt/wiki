---
url: https://www.linkedin.com/pulse/how-we-use-claude-code-today-intercom-brian-scanlan-eb7cc
date_fetched: 2026-07-05
backfilled: true
---

# How we use Claude Code today at Intercom

We built an internal Claude Code plugin system at Intercom with 13 plugins, 100+ skills, and hooks that turn Claude into a full-stack engineering platform. Here's what we learned building it - the highlights that went viral on X this week (180K+ views).

### The wildest one: a read-only Rails production console via MCP

Claude can now execute arbitrary Ruby against production data - feature flag checks, business logic validation, cache state inspection. Safety gates: read-replica only, blocked critical tables, mandatory model verification before every query, Okta auth, DynamoDB audit trail. I launched it by saying "It is either the worst thing in the world that will ruin Intercom, or complete genius." It is used a lot. No issues so far. Last time I looked the top-5 users weren't engineers - design managers, customer support engineers, product management leaders were all actively using it! The console is part of a broader Admin Tools MCP that gives Claude the same production visibility engineers have: customer/feature flag/admin lookups etc. A skill-level gate blocks all these tools until Claude loads the safety reference docs first. No cowboy queries.

### Full lifecycle observability with OpenTelemetry

We instrumented every Claude Code lifecycle event with OpenTelemetry. SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, PermissionRequest, SubagentStart... 14 event types flowing to Honeycomb. Privacy-first: we explicitly never capture user prompts, messages, or tool input. Session transcripts sync to S3 (with username SHA256-hashed for privacy). We can analyze how people actually use Claude at scale. On SessionEnd, a hook analyzes the entire session transcript with Claude Haiku looking for improvement opportunities. It auto-classifies gaps (missing_skill, missing_tool, repeated_failure, wrong_info) and posts to Slack with a pre-filled GitHub issue URL. This creates a feedback loop: real sessions -> detected gaps -> GitHub issues -> new skills -> better sessions.

### A forensic flaky test fixer

Our flaky test fixer is a 9-step forensic investigation workflow with a 20-category taxonomy of flakiness patterns. Hard rules: - NEVER skip a spec as a "fix" - NEVER guess root cause without CI error data - Downloads failure data from S3, classifies against the taxonomy Sweeps for "sibling" instances of the same anti-pattern. Fixes common patterns widely. This matters a lot when you've got hundreds of thousands of tests.

### PR workflow enforcement at the shell level

Claude Code hooks enforce our PR workflow: 1. A PreToolUse hook intercepts raw "gh pr create" and blocks it unless the create-pr skill was activated first 2. The skill extracts business INTENT before creating - asks "why?" not just "what changed?" 3. Another hook blocks ALL modifications to merged PR branches (push, commit, rebase, edit) 4. After PR creation, a background agent auto-monitors CI checks using ETag-based polling (zero rate-limit cost)

## Recommended by LinkedIn

### Evidence-based permissions and tool management

After 5 permission prompts in a session, a hook suggests running the permissions analyzer. It scans your last 14 days of session transcripts, extracts every Bash command approved, and classifies them into GREEN (safe), YELLOW (caution), and RED (never auto-allow). Then writes the safe ones to your settings. Evidence-based, not prescriptive. We also maintain good defaults! A separate PostToolUse hook detects "command not found" errors and BSD/GNU incompatibilities in real-time. Spots things like "grep -P" failing on macOS. Once per session, suggests the fix. Installs via Homebrew and updates Claude's config files so that it knows the tool exists in future sessions. Self-improving developer environment!

### QA and video analysis

Video transcript skill: feed it a Google Meet recording, get a markdown transcript with intelligently-placed inline screenshots at moments where the speaker says "as you can see" or "look at this." QA follow-up skill: takes QA session documents through a 7-stage pipeline that identifies issues, investigates the codebase, filters for quality, and creates GitHub issues to track. Far easier QA!

### Claude4Data: beyond engineering

Our data team built a Claude4Data platform with 30+ analytics skills - Snowflake queries, Gong call analysis, finance metrics, customer health reports. Sales reps, PMs, and data scientists all use it. One internal quote: "Friends at other tech companies are nowhere near this level of sophistication."

### Keeping it all running

We automatically ship our marketplace and keep it up to date on our Macs using JAMF. We run reports on skill creation and usage, and keep an eye on quality. The most used skills have high quality evals and are reviewed regularly.

### And there's more

          
      


The wild thing is we're just getting started. All technical work and our entire SDLC is getting skill-ified. Remote agents will accelerate things even more.

Amazing Brian, lots to learn from here. The telemetry based continuous improvement loop is something I built into a skill I'm using but you guys are streets ahead. Good initiative on the local dev support for non-technical users, this is key from what I'm seeing with broader adoption of Claude in our org.

The SessionEnd hook that feeds gaps back into skill development is the part worth paying attention to. I run parallel Claude Code sessions across projects daily. The hardest problem isn't the code quality, it's that every session surfaces the same friction points independently. Missing context, repeated permission prompts, the agent hitting the same dead ends across different projects. You fix it in one session and the next session has no memory of the fix. The feedback loop here (sessions → auto-classified gaps → GitHub issues → new skills) turns that into a system. Each session makes the next one slightly less broken. That's a different proposition to just adding more skills upfront and hoping they cover everything. Curious about the conversion rate: how many of those auto-detected gaps actually become new skills vs. sitting in backlog?

Amazing work! This is the way to go 🚀 /cc Ulrich Frank Martin

Lot of really great useful advice in here. Especially loved instrumenting the session hooks, I’m definitely borrowing that
