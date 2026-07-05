---
url: https://gist.github.com/485e7e4c84f54b2ff34f060f19593ae2
date_fetched: 2026-07-05
backfilled: true
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

Custom instructions are the missing manual for Copilot. They define coding standards, preferred technologies, and project rules—exactly what you’d tell a new teammate—and are sent with every chat request so Copilot doesn’t need to re-scan your codebase each time.

Instructions live in your repo and can be scoped. A .github/copilot-instructions.md file provides global guidance; smaller .instruction files (e.g., for C#, Razor) apply only to specific languages or areas.

VS Code can now auto-generate these instructions from your codebase. A one-click “auto update instructions” feature in the chat gear menu scans your entire workspace, identifies architecture, packages, naming conventions, error-handling patterns, and even what’s missing (tests, telemetry), then produces a structured markdown file.

Auto-generated instructions make Copilot ask clarifying questions and produce style-matched code. In the demo, after generating instructions, Copilot asked whether to implement full CRUD and in-memory storage before writing a DTO with JsonPropertyName attributes and endpoints that matched the project’s existing patterns.

Instructions are living documents. As the project evolves, you can re-run the auto-update to merge new context into the existing file, keeping guidance current without manual rewriting.

Pithy and provocative quotes

“Think of it as like the same exact rules that you would tell another team member about how we like to structure our code. You need to give Copilot this information that is sent with every chat request so it knows how to respond back.”

“So that way it doesn’t need to scan your code base every single time.”

“It is the very first thing that you should do.”

“Before I might just have guessed of how I wanted it to. But now because I’ve given it information with the copilot instructions, it’s now going to return back code that is more in line with my style.”

“So go off add some custom instructions today.”

Tools, practices, and methodologies

Custom instructions (GitHub Copilot)
Markdown files (.github/copilot-instructions.md or .instruction files) that define coding guidelines, preferred libraries, naming conventions, error handling, and project-specific rules. They are attached to every chat request so Copilot internalizes your standards without repeated codebase scans.

Auto-update instructions (VS Code Insiders)
A button in the Copilot chat gear menu that triggers an agent to analyze the entire workspace and generate or merge a copilot-instructions.md file. It captures core commands, high-level architecture, major packages, formatting, typing, naming, error-handling style, and even notes what’s absent (e.g., no tests, no telemetry). Use it to bootstrap instructions and then refine them manually.

awesome-copilot (GitHub repo)
A community collection of pre-made custom instructions, prompts, and chat modes for stacks like Blazor, React, TypeScript, etc. Provides a starting point that you can tailor to your project.

Agent mode with instruction references
When instructions are present, Copilot’s agent mode explicitly references them in chat, uses them to ask clarifying questions (e.g., “Should I implement full CRUD? In-memory or database?”), and generates code that respects the defined conventions.

Scoped instruction files
Break instructions into smaller files (e.g., csharp.instruction, razor.instruction) so that only relevant rules are sent for a given file type, keeping context focused and token-efficient.

Unanswered questions and omissions

How reliable is the auto-generated output? The demo shows a clean result, but the talk doesn’t address cases where the codebase has inconsistent styles, legacy patterns, or anti-patterns that might be baked into the instructions.

Maintenance and team dynamics. No guidance on version-controlling instructions, resolving conflicts when multiple developers auto-update, or ensuring the instructions don’t drift from actual team standards over time.

Token and performance costs. Sending a large instructions file with every request could hit token limits or slow responses, yet the talk doesn’t mention any trade-offs or size recommendations.

Beyond VS Code. The feature is shown only in VS Code Insiders. What about Visual Studio, JetBrains IDEs, or GitHub Copilot Chat on the web? The talk is silent on cross-IDE support.

Security and privacy. Scanning the entire codebase and sending architectural summaries to an AI model raises questions about sensitive data exposure, but this isn’t discussed.

What if the instructions are wrong? The talk assumes the auto-generated file is a good starting point, but doesn’t explore how to validate or correct it, or what happens when instructions conflict with Copilot’s own training.

Dynamic vs. static context. The speaker says instructions mean Copilot “doesn’t need to scan your code base every single time,” but the auto-update feature itself scans the whole codebase. It’s unclear whether the instructions remain static until manually refreshed, potentially missing recent changes.

One of the things that folks struggle with when they get started using AI tools is getting the AI to respond. In coding sort of standards and practices like your coding, you want the AI to code like you and give you formatted code that follows your standards. Now, when you provide it context, so when you're giving it code files, it will automatically take a look at that and try to sort of respond back with things that are similar. So if you use Pascal or Camel Casing or Underscores, it'll try to do that. But we can guide GitHub Copilot and AI tools by giving it more context.

And to do that with GitHub Copilot and VS or VS code or wherever else you're using GitHub Copilot, you can use custom instructions. It defines guidelines or rules for generating code, performing code reviews, or even generating commit messages. So you can give it coding practices, preferred technologies, project requirements, and think of it as like the same exact rules that you would tell another team member about how we like to structure our code. You need to give Copilot this information that is sent with every chat request so it knows how to respond back. So that way it doesn't need to scan your code base every single time.

So what would that look like? Well, custom instructions can be added into a GitHub folder and that's going to basically describe the code generation instructions, and those are sent with every request. You can also break them down into smaller files like dot instruction files for things like just apply these to C or apply this just to Razor, for example. So what would that look like? Well, here inside the VS code docs, I love it because it breaks it down like here's some general sort of coding standards like use Pascal for these versus use Camel for this, use prefix for this, use all caps for this.

Here's how I like to do error handling for TypeScript or React. Here's my TypeScript guidelines and my REACT guidelines, and then each of these instructions will be sent based on the code that it is analyzing and modifying. Now, there are tons of great GitHub repos out there that you can go and find instructions in, but there's a great resource on awesome copilot on GitHub that will also give you custom instructions, reasonable prompts, and chat modes. For example, if you're getting started with, I don't know, Blazor, you could tap on that. And here for example, is a markdown file that you could add or get started with at least and customize to your liking when working with A blazor.

So different naming conventions, Blazor and. NET guidelines, error handling and different API and performance optimizations when using Blazor Server or webassembly. Now of course you know about your project, so if it's webassembly you give it webassembly things. And if it's Blazor Server you could give it Blazer Server things for example, which are cool. But it's actually even easier than ever to generate and get started or even update your existing copilot instructions in VS code.

Here I am inside of the latest Visual studio code insiders build 102 and we can see this is my tiny shop application. So I have a blazor front end and ASP NET core backend and I don't have any files inside this GitHub folder. So I could go and I could add a new file copilot instructions or I can go into chat and I can see this little gear here to customize chat. This is where it's going to enable me to add prompt files, instructions, tool sets, custom chat modes, but also auto update instructions. So, so when I tap on that, it's going to send a prompt off to the agent and it's going to ask it to analyze the workspace or create or even update the existing one and do it based on different requirements for core commands for building give it high level architecture including major packages and services, repo specific styles like formatting, typing, naming, error handling.

If you have existing rules it will take a look at those and then it will also then generate this existing copilot instructions over here. It's going to also patch or merge it if it already exists, which is really nice. So it's going to use markdown heading bullets and then generate for you. So what it just did really, really quickly here in this project is it went off and it scanned the entire code base, looks at all the different looks at all the different code inside of it and it generated a new copilot instructions here. So let's take a look.

So what it's done is it said here's some core commands that you can use on a build run. Here's some Azure developer CLI things. There's no test projects found so I could add that in there. Here's the backend so it has the products which is minimal, APIs uses in memory or SQLite database has a API here, random failures for error demos, which is great. So that's nice to know.

Here's the blazor Front end data models and then different coding standards here. This is super nice. Right? So here's my, here's my C sort of guidelines, here's my JSON property, names for DTOs, Blazer and general. Right.

So it even tells it, you know, use it concise code, it's demo focus. There's no health checks, telemetry, advanced resiliency, things like that, which is nice. And then agent mode rules so it tells it stuff not to do, for example, and things to prefer. So this is super duper nice. So if I come back over into agent mode over here, let me open up the chat, I'm going to create a new thread and let's go ahead and keep and continue.

I'm going to say, all right, let's add a new API endpoint for users into the backend. And the first thing that we'll see is inside of the chat is that it is using a reference of the copilot instructions here. So we can see that it is adding this into it. So it's going to ask me and clarify here what should it create? And I'm going to say yes, yes, let's do all crud and those all look good and let's do in memory.

So it's actually looked and it's taken a look at those copilot instructions and asked me questions about how I want to implement it, which is nice. And before I might just have guessed of how I wanted it to. But now because I've given it information with the copilot instructions, it's now going to return back code that is more in line with my style. That's here. So here is my dto, my user with the JSON property names, which I like.

So I'm going to update my map user endpoints and hopefully what we'll see over here is that we have our user endpoints just like this. And what's great is that it now knows how to exactly run my terminal commands and I can give it more specific information on it as well. So now it's going to go ahead and build this up just like that. Now as I start to work in my project, I can always come back, click on this and auto update instructions again and then we'll update it and give it more context as my project grows. Anyways, that is how easy it is to get started adding those copilot instructions into your project.

So here it is just right here. You can add one or you can go ahead and add multiple that, break it down into smaller chunks. But it is the very first thing that you should do and build big shout out to the VS code team for adding this one click button to make it super simple to add some beautiful co pilot instructions that you can then massage exactly to your liking. So go off add some custom instructions today. Sam.
