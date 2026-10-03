---
url: https://www.telerik.com/blogs/6-months-claude-cursor-part-2-teaching-ai-work-senior-engineer
date_fetched: 2026-10-03
---

Summarize with AI:

As you progress in your AI journey, you’ll need to make your agents work for YOU, the individual you, with your strengths and shortcomings.


Image generated with Gemini

In Part 1, I wrote about building skills so the AI understands your project. Part 2 is a harder problem: building a skill so the AI understands **you**.

After six months of using Claude daily for code review, architecture discussions and prompt writing for Cursor, I realized the most persistent problem was posture rather than technical capability. The AI treated me like a beginner when I had 30+ years of experience behind me. It suggested removing intentional code and offered generic options when I already knew what I wanted. When I showed that my code was right, it apologized for three paragraphs instead of moving on.

I wanted a technical peer, not an assistant.

Before writing the skill, I spent weeks going back over our conversations. I asked Claude itself to comb through our history and list the moments it got the tone wrong. Three patterns came out.

The first was underestimation. I’d share one of my Source Genesys templates with a Stopwatch measuring a hook’s execution time, and the AI would suggest removing it because it “didn’t seem necessary.” I’d explain that it was intentional, that I wanted performance metrics inside the hook itself. It corrected itself, then did the same thing in the next conversation, with no memory of the last one.

The second was premature diagnosis. The AI would flag an error in my metrics code when the real problem was in the neighboring Controller. I knew that because I’d looked at both before asking. The AI assumed I hadn’t.

The third was the multiple solution. “You can do A, B or C,” when I already knew it was B. The time spent explaining A and C was time lost for both of us.

So I wrote `jefferson-senior-dev`. The name was deliberate: “senior dev,” not “user” or “client,” because that’s how I wanted to be treated.

The skill opens with who I am: software engineer, developing since 1994, creator of Source Genesys. The resume takes five lines. Everything else in the file is rules of engagement.

The main rule: when I share code, assume it’s correct until you find clear evidence otherwise. Don’t question out of generic caution. Don’t suggest removing something because it “doesn’t seem necessary.” If you find nothing wrong, say it’s correct and move on. Don’t invent caveats to look useful.

There’s one exception: security. SQL injection, data exposure, hardcoded tokens and open permissions get flagged every time, even when the code works.

The hardest rule to write was one about myself. Sometimes I swap **true** for **false** in boolean conditions, and I’ve been repeating that mistake for years. I’ve watched someone outside IT do the same thing, which makes me suspect I’m not the only one. So the skill tells the AI to flag every inverted condition it finds in my code, because that’s where questioning me adds real value.

Declaring your own blind spots to an AI costs nothing, and few engineers do it. If you know where you fail and it knows too, you get a second check that never gets tired and has no embarrassment about pointing out the obvious.

A good skill teaches AI when to disagree with you.

The change showed up in the first week. Claude stopped offering options when I already had a path. When I shared code with a `// fixed` comment, it assumed the fix was right and focused on applying it to the template. The five minutes I used to spend convincing it that my code was intentional went to zero.

The effect I hadn’t planned for was the quality of what it caught instead. The one I keep coming back to was a discard assignment on an awaited call: the line waits for the operation, throws the result away and swallows the failure with it. Fire-and-forget wearing the costume of disciplined async code, which is exactly why it survives human review: Nobody reads past the await.

With the false suspicions out of the way, there was attention left for that kind of thing. Today, when it flags something, I take it seriously, because I know it isn’t a generic guess.

After the Claude skill, I built the equivalent for Cursor: `cursor-prompt-craft`. It came out of six months of real prompts, the ones that worked and the ones that failed.

The central tension is discovery versus prescription. I used to think Cursor should explore the code on its own. The results showed the opposite: the most effective prompts were the ones that gave the exact file paths, named the functions to change, and marked what must not be touched.

That became the rule. For Source Genesys, Claude writes prescriptive prompts: exact paths, named files, examples of the canonical pattern and an explicit **“do not”** section. Cursor became a tool and stopped offering unnecessary opinions. It works better when you take away its obligation to guess.

In January, I wasn’t running a self-hosted GitHub Actions runner; my CI/CD was semi-automated, and deploys were manual. Cursor ran without rules and produced unpredictable results, and Claude treated me like any other user.

By June, Source Genesys was generating complete platforms with automated deploy, built-in observability, test coverage and post-deploy self-correction. Cursor worked inside limits defined by skills, and Claude knew when to trust my code and where my blind spots were. The semester also produced concrete deliverables: the new template for Source Genesys using Progress **Telerik UI for WinForms**, a generic dark mode CSS for every component, a frontend metrics system and automatically generated help screens.

None of that came from an AI alone, and none of it came from me alone. It came from a partnership that took six months to settle. Concrete results aren’t a matter of one week or one month. You must hone the tool until it delivers what you want.

July was the first month both skills ran without me adjusting them, and most of it went to infrastructure. A Virtual Private Server migration had left me with no metrics, no centralized logs and no traces, and the gap surfaced in the worst possible way: a production flow stopped working, and I had nowhere to look.

Debugging in the dark, in 2026, is embarrassing. The rest of that story is Part 3, but the conclusion fits in one line. If Source Genesys generates the infrastructure, it must generate observability along with it.

The part that touches this article came at the end of the month. I fixed the flow on one platform, and Cursor documented what it had done in a skill. The next question was inevitable: why am I applying this manually on the other six?

It became a CLI parameter. I passed the skill name and the list of platforms, and the pipeline fired the agent with the same instruction on each one. It ran on six of the seven platforms. The seventh didn’t have the module involved and was correctly skipped. The build passed on all six.

That changes what a skill is. In Part I, it was context. In this article, it was posture calibration. Now it’s a unit of work: a file that describes a fix and that the pipeline knows how to execute across N repositories.

With one caveat I insist on putting in writing: a green build is not a validated fix. The agent applied, compiled and reported. End-to-end testing on each platform is still human work, and that part I haven’t automated.

Calibration made me faster, and speed made my own mistakes more expensive. Three of them are worth writing down.

I told Cursor to clean up corrupted Latin characters across every repository at once, and the instruction was broad enough that it also replaced `??` with a dash. `??` is an operator in C# and in TypeScript. It broke working code and swept through dozens of documentation files on the way out. Git and a targeted pass got it back. The scope was mine to set, and I set it wrong. So it became a rule that forbids operator substitution and names the ones that are never touched.

I ran two pipelines at the same time, more than once. The sessions competed for the same processes, and the report came back with failures that weren’t failures. I spent an afternoon investigating noise. A session lock in the runner is in the queue.

Against those two, the best small decision of the month was writing per-session and per-platform statistics to CSV. It showed that a single command accounts for almost all the pipeline time, that one execution of it took 33 minutes while the retry took 2, and that a failure I had been treating as instability was deterministic: Cursor had compiled the project as x86 instead of AnyCPU. Without measuring, I’d still be guessing.

Calibrated AI accelerates whatever you told it to do, including the wrong thing. The skill keeps the AI from getting the tone wrong. It doesn’t keep me from getting the order wrong.

The `jefferson-senior-dev` skill that Claude wrote about our partnership ends with the line I consider the most important of all:

“Jefferson treats Claude as a high-level technical colleague, not an assistant. When he shares code, he expects peer analysis, not a tutorial. When he says ‘this is wrong,’ he is

almostalways right. Claude’s job is to add real value: find what Jefferson missed, not question what he already knows.”

That “almost always” I added on purpose. Nobody is right 100% of the time, and it’s in the cases where the AI catches what I missed that the partnership pays for itself.

Generative AI isn’t going to replace senior engineers, but it will deliver more to the people who know how to train it instead of treating it like a search engine. And training here isn’t fine-tuning or RAG. It’s a Markdown file with the right rules, loaded in the right context. A skill is actionable documentation: it records what you know and where you accept being corrected.

If I could go back to January 2026 and give myself one piece of advice, it would be this: *before you write your first prompt, write your first skill.*

In the next post, Part 3, I’ll show what happened when the infrastructure had to catch up with the partnership.


Automate with AI. Just don’t expect it to get to know you on its own. That part is your job.

Jefferson S. Motta is a senior software developer, IT consultant and system analyst from Brazil, developing in the .NET platform since 2011. Creator of www.Advocati.NET, since 1997, a CRM for Brazilian Law Firms. He enjoys being with family and petting his cats in his free time. You can follow him on LinkedIn and GitHub.
