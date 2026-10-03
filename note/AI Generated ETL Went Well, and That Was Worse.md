# AI Generated ETL Went Well, and That Was Worse

Pinal Dave's field report on replacing a 2016 legacy ETL job with an AI-generated one: the new pipeline was cleaner and worked on the first run, then spent six weeks revealing that the old job's "mess" was actually nine years of encoded incident history that no specification captured.

---

Dave asked AI to write a mundane daily job — vendor CSV lands on a share, fix a few columns, merge ~300K rows into a warehouse table. What came back was genuinely good: staging table, proper merge, logging, sensible error handling. It ran first try. Then, over six weeks, reality delivered four lessons:

1. **The Monday file is late, not missing.** The new job ran at six, found nothing, logged an error, stopped — exactly as specified. The old job's inexplicable wait-until-nine loop existed because of a 2019 incident nobody documented.
2. **The same file comes in twice.** The vendor resends corrected files; the new pipeline double-counted them. The old job's hash lookup existed for this.
3. **The file is perfect except every amount is zero.** All validations passed; the data was garbage. The old job had an undocumented rule: if today's total is under half the seven-day average, stop and page someone.
4. **The vendor upgraded and the encoding changed.** José became JosÃ©. Nothing failed — gibberish is a valid string.

> "A brand new pipeline is not missing code, it is missing every bad morning the old one survived."

This is the whole argument in one line, and it inverts the usual rewrite calculus. The odd checks in legacy code are either cruft or scar tissue, and the code itself cannot tell you which. Only the humans can.

> "It's tempting to blame the AI here, and I don't think that's fair... I'm the one who wrote the specification."

Dave refuses the easy villain. The AI delivered exactly what was asked; the specification was the gap. This is the sharpest available illustration of why [[AI Agents Need Clear Specs]] matters: agents amplify the completeness of what you say, not what you meant.

> "Ask the team for requirements, and nobody mentions the Monday file. Ask what happens when the file is late, and someone will say, 'Oh, Mondays. Let me tell you about 2019.'"

The contractor insight is the best part: the value of a good human contractor isn't better code, it's better *questions*. Counterfactual prompts ("what happens if the file doesn't show up?") extract knowledge from people who don't know they have it. AI doesn't ask, so the memory stays locked.

## Key themes

- #concept — legacy code as an incident archive; "mess" vs. scar tissue
- #pattern — failure-mode enumeration before code generation: ask the AI to list every way the pipeline could fail (including quiet ones), then interview the humans about which have actually happened, then generate code with each handled on purpose
- #concept — quiet failures: data that passes every validation but is wrong

## Analysis

This is one of the strongest short pieces on AI-assisted maintenance of *operational* code (as opposed to greenfield app code). Its contribution is the failure taxonomy of ETL specifically: late files, duplicate files, plausible-but-wrong data, encoding drift — none of which are type errors or logic errors, and all of which pass a clean implementation's own tests.

It complicates the "clean code" orthodoxy: Dave notes that clean ETL generated in an afternoon makes rewrite temptation *cheaper and more dangerous*, since every new engineer wants to rewrite the ugly job and now anyone can. The prescription — go through the old job line by line, ask why each check exists, expect an answer for about half, and **write the answers down this time** — is honest about partial recoverability rather than romanticizing legacy code.

The workflow he lands on is a practical synthesis of spec-first practice and tribal-knowledge archaeology, and it pairs naturally with [[How to Write a Good Spec for Agents]] (spec quality as the binding constraint) and [[AI-Assisted Database Work — The Machine Reads, The Human Decides]] (the human as the keeper of operational judgment in data work). It also strengthens [[The Lindy Effect in Software]] from a different angle: the old job survived not because it was well-designed but because every design flaw had already been paid for.

---
*Sources: [[raw/ai-generated-etl]], [[summary/ai-generated-etl]]*
*Last updated: 2026-10-03*
