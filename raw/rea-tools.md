---
url: https://rea.tools/
date_fetched: 2026-10-10
---

Reverse engineering, with your coding agent

# Find out how

software works.

            REA gives your agent the tools to inspect a program and explain what it does.

Already know RE? Skip to analysis guides →

## Set up REA

Copy this into your coding agent:

Install REA and connect it to this coding agent using npx rea-agents@latest setup. Show me the setup plan for approval, then verify the installation.

`npx rea-agents@latest setup`
              
            ## What is reverse engineering?

              Finding out how software works by
              **examining the program itself.**
            

The goal: understand a feature well enough to explain it, change it or rebuild it.

            One question:
            **why does Calculator give 220 for 200 + 10%?**
          

            **Let's try two examples.**
            Change a game's speed, then return to Calculator and rebuild its %
            button.
          

Example 01 · Chrome’s dinosaur game

## Why does the dinosaur get faster?

- Goal
- Rebuild the game with adjustable speed.
- Why reverse engineer it?
- 
                To reproduce its acceleration, we need
                **the rule in the running game**: how fast it starts, how much it increases and when it stops.

REA returned

                  **The script actually running in the page.** The
                  agent reads its update function and speed settings, then uses
                  that rule in a new game.
                

Use REA to inspect this dinosaur game. Why does it get faster? Show the speed rule, then build a small version with an adjustable speed.

```
if (this.currentSpeed < this.config.MAX_SPEED) {
  this.currentSpeed += this.config.ACCELERATION;
}
```
                **Start at 6.** Add **0.001** each
                update without a collision, while speed is below
                **13**.
              

## See the script, checks and how to try the analysis

              REA inspected the HTTP browser edition through a local debugging
              connection and returned the loaded `index.js`,
              including its source and digest.
            

```
ACCELERATION: 0.001,
MAX_SPEED: 13,
SPEED: 6
```
We called the original game’s update function in a controlled browser check, with obstacles and automatic scheduling disabled. After 4,000 updates, speed was 10.0; after 10,000, it was 13.0, rounded to one decimal.

The new mini-game keeps that speed rule. Its drawing, jumping and collision code are a small teaching implementation. Open the lab to see the new code and run the same speed check.

To inspect the target yourself, follow the browser connection steps using the dinosaur page. Give your agent that page URL and your local debugging endpoint, then copy the prompt above.

Example 02 · Windows Calculator

## Rebuild Calculator’s % button.

              We saw why **200 + 10% gives 220.** Now let’s recover
              the rules for both + and ×, and try them.
            

- Goal
- Build a small calculator that handles both + and × correctly.
- Why reverse engineer it?
- 
                The same button uses
                **different rules after + and ×.**We inspect the installed app to find which number the percentage is applied to.

REA returned

                  **One branch divides by 100. The other also multiplies by the
                    first number.**
                  The agent uses both to recreate the % button.
                

                For this example, `previous = 200` and
                `current = 10`.
              

Use REA to inspect Windows Calculator’s % button. Recover the rules after + and ×, then build a small calculator that uses both.

```
if (operation == multiply || operation == divide) {
  percent = current / 100;
} else {
  percent = current * previous / 100;
}
```
- With +
- 
                    **Take 10% of the first number.**`10% of 200 = 20 → 200 + 20 = 220`
- With ×
- 
                    **Turn 10% into 0.1.**`200 × 0.1 = 20`

## See the real code and how the answer was checked

```
0x180124945: MOV EAX, dword ptr [R13 + 0x18]
0x180124949: CMP EAX, 0x5c
0x18012494c: JZ 0x180124aba
0x180124952: CMP EAX, 0x5b
0x180124955: JZ 0x180124aba
0x18012495b: MOV EDX, 0x64
```
```
#define IDC_MUL 92      // 0x5c
#define IDC_DIV 91      // 0x5b
#define IDC_PERCENT 118 // 0x76
0x64 = 100
```
Your first investigation

## Try REA on a small app.

Download our Notes example, trace its CSV export, and check one changed input.

## Want to go deeper?

              Tell your agent what you want to understand or build. With REA,
              you can work together on anything from
              **cloning this website** to
              **reconstructing a game from its executable.**
            

Use REA to inspect https://rea.tools/. Clone this website for me.

Use REA to reconstruct this game from its executable. Recover the gameplay logic in C and test it against the original.

### Native binaries

Inspect functions, strings, references and call relationships in executables and libraries.

Native analysis guide### JavaScript & Electron

Map modules, routes, IPC and native dependencies from an application folder or ASAR archive.

Application workflows### Browser & runtime activity

Capture selected browser or process activity, then compare the results across runs.

Browser observation guide## Any questions?

Read the FAQ, chat with us on Discord, or open a GitHub issue for bugs and feature requests.
