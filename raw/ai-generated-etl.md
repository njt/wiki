---
url: https://blog.sqlauthority.com/2026/09/25/ai-generated-etl/
date_fetched: 2026-10-03
---

**AI generated ETL was the thing I was most sure would go badly. It went well, and that turned out to be worse. The pipeline ran clean the first time, and it took six weeks to find out what it was missing.**

The job itself was nothing special. A vendor drops a CSV file on a share every morning. We check it, fix up a few columns, and merge about three hundred thousand rows into a warehouse table. If you work with data, you’ve built this job more than once.

I described the source, the target and the rules, and asked AI to write it. What came back was good. It had a staging table, a proper merge, logging, and sensible error handling around the file read. It ran on the first try, the numbers matched, and everybody was happy. I’ll admit I was a little disappointed. I had a whole “I told you so” ready.

The job it replaced was written in 2016 by someone who had left the company years ago. It wasn’t pretty. It had odd IF checks, a hash lookup, and a wait loop nobody could explain. On day one, the new code was better, and I’d have said so to anyone who asked.

## Six Weeks of Surprises

**The Monday file was late.** Not missing. Late. The new job ran at six, found no file, logged an error and stopped. That’s exactly what I told it to do. The old job used to wait until nine before raising an alarm, because this vendor’s Monday file often runs behind. That’s what the mystery wait loop was for. It went in after an incident in 2019, and nobody ever wrote it down.

**The same file came in twice.** Now and then the vendor fixes a mistake and sends the whole file again. The new pipeline happily loaded it a second time, and a batch of adjustment rows got counted twice. The old job kept a hash of every file it had already loaded and skipped repeats. So that’s what the hash lookup was doing.

**The file was perfect, except for the numbers.** One morning every amount in the file was zero. The columns were right, the data types were right, and the row count was right. Every validation passed, and it loaded without a hiccup. The old job had an undocumented rule for this. If today’s total is under half the seven-day average, stop and page someone. That rule was there because it had happened before.

**The names turned into gibberish.** The vendor upgraded their system, and the character encoding changed with it. Accented names started arriving mangled, so José showed up in the warehouse as JosÃ©. Nothing failed, because gibberish is still a perfectly valid string. The old job had handled that too.

## I Missed It Too

It’s tempting to blame the AI here, and I don’t think that’s fair. I asked for a pipeline that loads a file, and it wrote a good one. Not one of those four problems was in my specification. I’m the one who wrote the specification.

The old job was a pipeline plus nine years of bad mornings. Every strange check was added the day after something broke. None of it was in a document or a wiki. It lived only in the code, where it looked like mess.

Some of it was mess. There’s a check in there that rejects negative quantities. Is that a real business rule, or did someone see one negative number in 2018 and panic? The code can’t tell you, and the person who wrote it left in 2021. So AI gives you the clean version, the one that exists before real life gets to it.

## A Contractor Would Have Asked

If I’d handed the same specification to a good contractor, I’d have gotten roughly the same code back. The difference is that a contractor knows they’re new, so they ask questions. What happens if the file doesn’t show up? Has this vendor ever sent you a bad file?

Those questions are how you get knowledge out of people who don’t know they have it. Ask the team for requirements, and nobody mentions the Monday file. Ask what happens when the file is late, and someone will say, “Oh, Mondays. Let me tell you about 2019.” The AI didn’t ask. So nobody remembered, and it didn’t get built.

## What I Do Now

I don’t ask for the code first anymore. I ask the AI to list every way this pipeline could fail. That includes the quiet ones, where nothing errors but the data is wrong. It’s excellent at this. You get late files, duplicate files, partial files, schema changes, encoding problems and totals that look fine but aren’t.

Then I take that list to the people who run the old job. I ask them which of these have happened here. That conversation takes about twenty minutes, and it’s where the real requirements come out. Only after that do I ask for the pipeline, with each of those failures handled on purpose. Same tool, a much better result.

## Before You Replace the Ugly Job

Clean ETL code should make us a little nervous now. An old pipeline full of odd checks and hard-coded exceptions looks like bad engineering. Every new person who sees it wants to rewrite it. Now anyone can generate the clean replacement in an afternoon.

Some of that mess is bad engineering. Some of it is the only record of everything that has ever gone wrong. So before you delete the old job, go through it line by line. Ask why each odd check is there. You’ll get an answer for about half of them. This time, write the answers down.

I wrote about this habit in **Stop Asking AI for Answers**. It’s an essay about making AI argue with you before you trust what it hands you. It’s one of thirty essays in my book **AI: Nobody’s in There. But we’re still in here.** All of them are **free to read online**, and the paperback is on **Amazon**.

**A brand new pipeline is not missing code, it is missing every bad morning the old one survived.**

Published by **Pinal Dave** on SQLAuthority. More of my work at pinaldave.com.

### Discover more from SQL Authority with Pinal Dave

Subscribe to get the latest posts sent to your email.
