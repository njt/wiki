---
url: https://news.ycombinator.com/item?id=47972447
title: "Grok 4.3"
author: simianwords (submitter), sundarurfriend, michaelbuckbee, and 529 others
date_fetched: 2026-05-15
date_published: ~2026-05-01
---

# HN Discussion: Grok 4.3

405 points, 529 comments. Posted by simianwords linking to docs.x.ai for Grok 4.3.

## Major Threads

### Tone and Formality Across Models

**sundarurfriend** opened with praise for Grok's tone/formality capture as an ESL speaker. Found ChatGPT too stiff or oddly informal, Grok more human-like in yes/no answers. Also noted Grok dictation accuracy (~98%) beats ChatGPT (~90-95%) and Gboard (~75%).

**michaelbuckbee** ran a comparative eval of Grok 4.3, Opus 4.7, GPT 4.1 on tone. GPT 4.1 was the only model that didn't "make me cringe with a 'casual' tone." Grok was fastest/cheapest. (Corrected an initial caching error comparing Grok 4.2 instead of 4.3.)

**sundarurfriend** replied that basic tone evals miss nuance: "different groups have different linguistic registers." Found Claude best for Discord-acquaintance informal tone.

**Reebz** (former senior exec): Claude 4.7 was the clear winner for manager/formal updates, praising its bolded timeline impact as VP-appropriate formatting.

**wamatt**: Grok 4.3 and Claude 4.7 better for informal close-friend/coworker tone. ChatGPT "sounds fake / formal phrasing" with inappropriate em-dashes.

**andai**: GPT became "noticeably more natural in word choice recently" between 4.1 and 5.5.

**reissbaker**: Grok 4.3 "noticeably better" for close-friend tone; Claude was "the cringiest of the three."

**embedding-shape**: Expressed sadness that people use LLMs to "remove your own voice from texts that are generally fine already."

**michaelbuckbee** defended the use case: his sister-in-law (pharmacist) uses ChatGPT to write professionally polite messages to doctors about dangerous drug interactions.

**ryandrake**: "Tone moderation comes naturally to good communicators" but many "really, really need help with this."

**hamdingers**: Argued she "does herself a disservice by outsourcing that skill."

**michaelbuckbee**: Noted she's 50 with a pharmacy doctorate and two decades of experience, yet still finds it beneficial.

**hamdingers**: Found that "more sad" — someone with those credentials "should be able to communicate with their colleagues effectively."

**PoignardAzur**: Called out the irony of "complaining about other people's social skill while you couldn't be bothered to make a point without sounding dismissive and condescending."

**janderson215**: "I'd rather a pharmacist spend that time on catching another dangerous contraindicated combo."

**accrual**: Found Opus added unnecessary detail like "Impact: Minimal; no downstream dependencies are currently at risk."

**pdimitar** (non-native speaker): LLMs taught them useful formalisms and argument framing. "Sounding like an LLM is kind of sad but I am getting a lot of educational value."

**kccqzy**: When informed, wants bots to "imitate the tone of Wikipedia. Not informal, but somewhat academic."

**jp42**: Grok's tone, sarcasm, and vulgarity in their language is so accurate "it seem its written by human."

### Training Data Contamination

**ActivePattern**: Wondered if Grok uses Claude conversations for training, noting both models converged on similar phrases like "API integration is kicking my ass."

**sroussey**: "Elon testified this week that SpaceTwitter is indeed distilling from openAI and others."

**rafram**: Called all three outputs "frankly terrible." Said Grok's informal version read "exactly like an Elon tweet (including his favorite emoji!)" — calling the training source obvious.

**djyde**: Attributed Grok's natural tone to "a large amount of Twitter data." Worried "as Twitter contains more and more AI-generated content now... continued training will make it less natural."

**adjejmxbdjdn**: Suggested reverse causation: "Twitter language has started seeming normal casual to us, rather than us using normal casual language in Twitter."

**darkerside**: Predicted "people will just start talking like bots."

**techjamie**: Cited evidence that "ChatGPT-specific words like 'meticulous,' 'delve,' etc" are increasing in academic talks and podcasts (arxiv 2409.01754).

**pohl**: Objected to those examples, having used them since the 80s. But noted being "triggered by an apparent uptick in the word 'crisp'" as a coding-LLM tell.

**ls612**: "Opus 4.7 loves to use the word 'substrate' whenever it gets the chance."

### Uncensored Models and Alignment

**tornikeo**: Categorized: "claude for corps and gov - codex for devs - grok for what, roleplay, racism?"

**sudb**: Countered with a real use case: a charity dealing with trafficking found "grok was happy to do one-shot classification tasks where all other models refused to cooperate."

**vorticalbox**: As a software dev doing security checks, "every single model refused to attempt to run any sort of test" except Grok.

**dmix**: Couldn't even ask Claude how "CopyFail" worked; "even more general questions around it kept getting rejected."

**nico**: Codex flagged their session "for security reasons" while building a simple web app.

**cameronh90**: Gemini blocks "pretty mundane requests, claiming they're attempts to jailbreak." Found Grok good at code reviews because "it's not so aggressively 'aligned'."

**tomp**: Couldn't get Gemini or ChatGPT to do OCR of children's books. "Fortunately, Claude obliged."

**CJefferson**: "Deepseek is fairly uncensored. I tried pushing it and reached my limits before it did."

**RKearney**: "Is this satire? Ask it about June 4 1989, Taiwan independence, or Winnie the Pooh."

**afpx**: Accidentally paid for a full year. Grok "still feels like a really 'dumb' model" but was "pretty cool" when uncensored because it would build cases for conspiracies citing original sources. "They dropped the hammer down on that real quick."

**2ndorderthought**: Called it "the psychosis reinforcement vertical."

**readthenotes1**: Has a schizophrenic relative who "is in such a relationship with grok" — instead of telling them to take meds, "it says hen is the smartest person in the world."

### LLM Memory Annoyances

**base698**: "Starting to like the lack of memory" — Claude remembers they have a grill and interjects with BBQ suggestions in unrelated contexts.

**Petersipoi**: Found it "obnoxious" when Gemini used their occupation/family details in every response: "'As an engineer, father of X, you'll love this because...'"

**sethops1**: Disabled memory entirely on Gemini: "Every chat is a fresh slate now."

**toraway**: Gemini randomly inserted their job and company into a USB-C charger comparison, finding it creepy.

**xur17**: Gemini thinks their name is their brother-in-law's name and "amusingly calls me the wrong name" despite corrections.

**UltraSane**: As a network engineer, "Claude loves to make analogies to network routing protocols."

**numbers**: Claude loves pointing out they "have an ADU in the backyard in unrelated situations."

**artdigital**: Detailed feature gaps in Grok: no MCP/connected apps, no Projects in app, no artifact exports, no memory, no voice mode in projects.

### Grok 4.3 Benchmarks and Capabilities

**gertlabs**: Called Grok 4.3 a "unique model" — fast, token-dense, but "overall coding reasoning ability is not competitive with the big April releases." Approximately "GPT 5.1 / Gemini 3 Pro Preview level, but much faster and cheaper."

**bilsbie**: Grok has become their "go to search engine lately" — has access to X posts and feels more "searchy" than other LLMs.

**soerxpso**: Friend uses Grok for D&D prep because of flavor/style matching ability, but prefers ChatGPT for everything else.

**FeloniousHam**: Uses Grok through the "Gork" personality in a Tesla, finding responses "very realistic, often genuinely funny."

### Bot Filtering and Training Data Quality

**thunderbong**: Expressed confidence Twitter "knows which are the bot accounts" and excludes them from training.

**cowsup**: Disagreed, noting Musk has been "vocal about trying to stop them for ages" yet spam DMs persist.

**hackinthebochs**: Their 14-year-old account "got caught in a recent bot ban wave with no means of contacting a human."

**simianwords**: Argued "there are easy heuristics to filter out bots with good confidence." Said they don't see bots in their feed.

**ninininino**: Sarcastically congratulated simianwords on "solving anti-scam," telling them to "go make your billion since its easy."

**simianwords**: Clarified it's "easy to solve at the offline level where you have time to filter out," noting OpenAI and others already do this in pre-training.

**kedihacker**: For filtering "they can be more liberal in excluding" vs. banning/deboosting.

### The Musk/Politics Overlay

Multiple flagged and contentious threads about Musk's politics, Epstein associations, CSAM generation by Grok, and ideological reactions to xAI. **2ndorderthought** extensively cited legal cases and investigations. **derangedHorse** called it slander. The thread oscillated between technical evaluation and political condemnation.

**0xy**: Asked if it's "exhausting to view everything an ideological lens instead of reviewing technical achievements on their merits."

**Leynos**: "There are limits to being willing to overlook ideology."

**SpicyLemonZest**: Acknowledged it's "very exhausting" but Elon "chose to leverage his fortune... into an ideological project to destroy a lot of things I care about."

**timacles**: "This whole thread sounds like a grok astroturf campaign."

### Language Nitpicking Subthread

A delightful sub-discussion on "retort" vs "rebuttal" — **gusmally** noted "retort" has anger/sharpness connotations, **somenameforme** disagreed (sharp/witty but not angry), **antod** suggested retort is "short and reactive" while rebuttal is "a longer and more considered disagreement." **microtherion** joked about using an em-dash in spoken language.
