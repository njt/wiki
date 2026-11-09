---
url: https://www.bobbytables.io/p/alphamaxxing
date_fetched: 2026-11-09
---

I used to sit down and plan the entire stack for a project. I’d scribble down the programming language, which database would store data, and where it would be hosted. I’d jot it down in a notebook while sipping a martini at Dante in NYC. A few days later, I’d run `git init` in a new directory and start writing my implementation. By hand. Music turned up to 11.

It was glorious.

Starting a project isn’t simple, and AI makes the adventure even more complicated. AI can write code lightning fast, but can’t (or maybe “shouldn’t”) choose your project’s trailhead. There is one truth, though, in this newfangled world of AI assisted software development: AI is not only building our software, **it’s also becoming the predominant user of our software.**

## The API is now the product

Muse, Grokbot, Dots, and whatever 38 products that launch in-between these keystrokes can all perform magic. But that magic can’t happen without an API. Large Language Models, for all intents and purposes, are brains in a jar. Artificial Intelligence needs Artificial Muscles and Bones attached for it to be useful.

Browser-based interfaces for your users are still a must (for now), but if your *programmatic* interface is lackluster or down-right terrible, then I have bad news: your TAM (total addressable market) is smaller. Not because there are less humans that would benefit from your product, but because their agents, assistants, bots, etc… can’t use your product.

The API is now your product. And I’m going to tell you how I approached this new reality.

## Getting Started

I’ve started hundreds of projects, but with my project, Cadenya, I took a different approach than my notepad and martini past. Instead of enthusiastically writing code in an empty directory, I started defining the API surface first. I chose Protocol Buffers and gRPC as the interface definition language (IDL).

My hypothesis was simple: If I define the API endpoints with documentation and validation rules, then the AI could “take the wheel” and implement it.

I spent a few days tweaking my API contract until I was happy. I cracked open a new Claude Code session, gave it some manufactured encouragement with “You’re a principal Go engineer!”, and watched it crank the wheel with my service definition in hand.

It’s important to pick a language you can call bullshit on with Claude. Go and Ruby are my expertise. So how did Claude do?

## First Impressions

The first result of the API implementation was bad at best and **atrocious** at worst. But it didn’t matter. I could play with it using tools like grpcurl and feel the API design in action. I could be my end-user in an afternoon, but more importantly:

I could see how *Claude*, who is a persona in my addressable market, would try to use it in a new session. Claude had its stumbles, but it was clear that it wasn’t *struggling*. I’d prompt it with tasks such as “Create a new tool set via the API” and see how it handled the task.

The initial “meh” API design went from hypothesis to working implementation in an afternoon. Using my intuition and taste, I could tell which parts were wrong and what to fix.

## Sacrificial Code

Some engineers fall in love with their code. I used to be that engineer. But in the world of AI, you can’t love the code. Farmers don’t name their livestock. You should not treat any of your code as long-for-this-world, either.

I wanted another perspective on the implementation.

With this new enlightenment, I deleted the folder with my Go implementation. I kept my eyes wide open with a sinister grin as I typed `rm -rf ./caddyshack/`

gRPC and Protocol Buffers are implemented in several languages. I wanted to take another crack at the API after feeling out the alphamaxxing implementation. I’m also lucky enough to have been a classically trained Rubyist for the past 16 years.

## Same API. Different Backend.

I wanted to see how my API choices would influence my database model. Ruby on Rails has the best ORM in the world. I can model data, migrate a database, and add validations in less than 100 lines of production-quality code.

After tweaking my API contract, I played the same song with my ruby-encrusted instrument. “You are a principal level Ruby engineer. Implement the gRPC server for the `./api` directory in a new Ruby on Rails application”

Several Spindrift seltzers later, I was able to play with the new API again, and see how the database was influenced. This experiment was invaluable because it influenced several decisions. It’s why I switched from UUIDv7 to prefixed ULIDs. It’s why I split out a Workspace model from the Account table. And it’s how I modeled machine-friendly user identities.

## Ruthless Abandonment

One API contract. Two prototypes. Two `rm -rf`’s, hundreds of validation points. Time to value: *A few days.*

However my sloppy-joe prototypes never hit the spot. They tasted good, but, like a sloppy joe, I felt icky on the inside once they were done. And if you’ve never had a sloppy-joe, find one. It will change your life. And then wreck it.

## Stick-shift Coding

I learned how to drive in a car with a stick shift. I loved it. I felt in ultimate control of my 1989 Honda. A perfect gear change as my foot let the clutch fit the gears together was a brief but blissful experience each time.

That’s how writing code by hand feels to me.

So I made the decision to **write Cadenya’s initial API implementation by hand**. I created a new git repository, opened up Zed (my new favorite editor), and started cranking. Music up to 11.

Just like driving my manual 1989 Honda.

## Hand-rolled Code

I chose Go for Cadenya’s backend. And writing the codebase by hand meant I had the reigns of the application’s structure. I chose the packages and folder structure, wrote the database migrations, and even the Envoy configuration, all by hand. It rocked.

And because I knew my API contract made (enough) sense, I could focus on the quality of the code, and nothing else. I leaned into my engineering taste and experience which would become the bedrock for the most important thing in the world of AI Assisted Coding: **Context.**

I wasn’t building features so much as I was writing context. Pay attention to any live Claude or Codex session and you’ll see it call the `read_file` tool dozens of times. I wanted to influence Claude and Codex and bias it towards my preferences. I went all in on this approach.

That decision to hand-write features for three (yes, *three*) months is why I can ship features in a matter of hours - with a code quality bar that would not have been met if I let Claude make all of the decisions in a bare folder.

## Shifting to Automatic

Wiring up Temporal, gRPC, Envoy, Postgres, etc. might *seem* like a waste of time in a world where AI can do these tasks. But there’s an important distinction between doing grunt work and setting up a project for success.

“While we teach, we learn” - Seneca

I’m not comfortable letting AI know everything about my codebase and leaving me with nothing. You shouldn’t be either. The foundations of an application (often) are the future constraints of progress.

Hand-rolling the initial implementation of the API resulted in multiple gifts:

- I created my syllabus for coding assistants and the patterns this application uses and why 
- Simultaneously, I was locked in how the backend worked from day 0, because I built it 
- I felt - *good*about it and I wasn’t worried about sloppy-joe code hiding in my foundations

And maybe the most important thing: **My prompts for new features were rooted in examples, not half-baked “now make it do this.”**

I could reference database tables, workflow files, and so much more, while steering an LLM to an outcome. I became Google Maps for coding agents in my codebase.

## Set the tone and tempo

Go to any concert and the lead singer will inevitably scream “put em up!” during a song and start clapping their hands, encouraging the audience to join in. Within a few beats, everyone is locked in. Pleased, the singer goes back to their microphone and continues the song alongside the correctly tempo-ed applause. Everyone else keeps clapping to the beat without assistance.

That’s what coding with AI is to me now. I’m the lead singer, and my job is to set the tempo of the rest of the band. If I say “start clapping” to anyone, **they’ll pick the tempo and volume**. I can’t get upset if it’s not what I wanted.

Defining an API contract, discovering multiple routes of implementation, and then building the initial pieces with my own two hands has led to a stable codebase. And Claude and Codex are now my trusty bandmates.

And now *we* can crank it up to 11 again.

## Shameless Plug

If you’re looking for an agent runtime that is the Switzerland of AI platforms, then Cadenya.com might be right up your alley.
