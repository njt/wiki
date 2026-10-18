# talat — Local-First Meeting Transcription

talat is a two-person-company transcription app for macOS and Windows that records meetings, dictates into any app, and imports recordings — all processed on-device, with summaries from a built-in Qwen3.5-4B or a user-supplied cloud LLM key, and every transcript in a local database that never leaves the machine. Its most interesting feature for this wiki is an MCP connector that pipes meetings into assistants like Claude, making it a local-first personal data layer for agents rather than just a note-taker.

---

The landing page is a straight privacy-by-architecture pitch, and the interesting part is how much of it is *structural* rather than promissory: no account, no cloud processing by default, voice prints that "stay on your device," and a lifetime license that keeps working because "everything lives on your computer." Compare that to the usual AI notetaker stack — bot joins the call, audio goes to someone else's servers, retention is a billing tier.

> "Most notetakers treat your data like a byproduct. You're treating it like it belongs to you."
> — Julian Francis, Product Hunt

That quote is the thesis in one line. The category leader Granola — described by a testimonial as "'Granola but completely local'" — sends audio to the cloud and trains on it unless you opt out; talat's comparison table makes the opt-out-vs-architecture distinction the axis of the whole market.

> "If you'd prefer to use a cloud LLM for summaries (they usually give better results on longer meetings), that's your choice. You choose the provider, you supply the API key, you decide who to trust and when."

Honest concession, and the right pattern: local inference is the default, cloud is a deliberate per-user trust decision with the user's own key — not a vendor middleware claim. This is the same trust-boundary logic as [[A Home for Personal Context (Duncan Davidson)]], except talat implements it by keeping the data home instead of building a canonical repository.

> "Your meetings in your AI tools, via MCP."

The MCP connector is what lifts this above a privacy accessory. Meetings become an agent-reachable local source — summaries, transcripts, speaker attribution — which is exactly the substrate personal agents otherwise lack. See [[Guardian Angels]] for why a persistent, personal, continually-updated record of what was said is the hard part of a useful digital twin; talat is a plausible capture layer for it.

## Key Themes

#tool #personal-agents #local-first #privacy #mcp

## Critical Analysis

It's a marketing page, so the comparison table deserves skepticism — every vendor's table is drawn with itself in the winning column, and " recognises people across meetings; voice prints stay on your device" is doing a lot of load-bearing work without published accuracy numbers. The $189 lifetime price from a two-person team raises the standard sustainability question every local-first indie faces: on-device transcription has no per-seat server cost, but also no recurring revenue to fund the Qwen-model treadmill.

Still, the product design is coherent in a way most AI notetakers aren't. The local/cloud split is honest about the quality gap ("cloud LLMs usually give better results on longer meetings") instead of pretending on-device models are equivalent — a useful contrast with the local-model boosterism in [[Local Qwen Is Not a Worse Opus]]. And the MCP-connector-as-feature move suggests where the category is going: the meeting archive becomes tool-calling context, and privacy becomes a feature you can charge for rather than a compliance checkbox.

## Related Pages

- [[Whistle — On-Device Speech Recognition]] — the engine-side counterpart: Cactus' 16.9 MB on-device ASR shows the component technology talat's pitch depends on is real and improving fast, which strengthens talat's "local can match cloud" bet.
- [[Summarize Meetings Skill]] — Harper Reed's batch transcript-summarisation skill is the DIY version of what talat productises; talat adds capture, diarisation and a local database where Reed supplies his own transcripts.
- [[A Home for Personal Context (Duncan Davidson)]] — Davidson argues personal context should live in a user-controlled repository agents request permission to use; talat is an instance of keeping that context local by default, complicating the "repository" framing with plain on-device storage.
- [[Guardian Angels]] — Gwern's digital-twin vision needs exactly the kind of long-horizon, user-owned conversational record talat accumulates; this source shows a market forming around the capture layer.

---
*Sources: [[raw/talat-app]], [[summary/talat-app]]*
*Last updated: 2026-10-18*
