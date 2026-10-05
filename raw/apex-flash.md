---
url: https://www.cantina.security/apex-flash
date_fetched: 2026-10-05
---

# Introducing Apex Flash-1

Our first open-weights model trained specifically for security research

Today, Cantina Security, in partnership with Yeta Labs, is releasing apex-flash-1, an open-weights model for security research, post-trained on real vulnerabilities from our proprietary dataset.

As AI systems become more capable at cybersecurity, an important question is emerging around who should be allowed to use that capability and under what conditions. Much of the current safety debate focuses on controlling access through proprietary models, API policies, usage restrictions, and safeguards designed to prevent advanced cyber capabilities from being misused.

The problem is that cybersecurity is adversarial.

Attackers do not respect acceptable-use policies, enterprise procurement rules, data residency requirements, or third-party risk controls. They can run open models locally, modify them, remove safeguards, and build systems around capabilities that already exist. Defenders are the ones operating under the strictest constraints, and that creates an asymmetric form of safety where legitimate security teams are restricted from capabilities that determined attackers can still obtain.

Cybersecurity has been through a version of this debate before, in the late 1990s and early 2000s. Vulnerability disclosure, exploit development, scanners, and offensive security tools were all controversial, because the same technology could be used to attack systems or defend them. Over time, tools like Nmap and Metasploit, public vulnerability research, and reproducible exploits became part of the basic infrastructure of modern security. The takeaway was not that dual-use technology is harmless, but that restricting the knowledge available to defenders does not make the underlying offensive capability disappear.

AI brings that argument back at a much larger scale.

The question is not whether these capabilities can be misused. They can. The question is whether restricting legitimate access meaningfully restricts attackers once capable open models already exist. We think it increasingly restricts defenders instead. Machine-speed attacks require machine-speed defense, and defenders need models they can run, control, and improve inside the systems they are responsible for protecting.

Our first contribution to this collective defense is apex-flash-1.

Alongside the standard model, we are releasing apex-flash-1-abliterated, an abliterated variant for researchers who want greater control over model behavior in their own security workflows.

For this initial training run, we created 150 tasks from 50 vulnerability cases. Each case has three views with different levels of information.

-  01 #### Guided whiteboxFull source code plus detailed guidance on the vulnerability and exploit path. The agent still has to execute a working exploit. 
-  02 #### Focused whiteboxFull source code with limited direction toward a subsystem. The agent must investigate the flaw and develop the exploit itself. 
-  03 #### Focused blackboxLimited direction and access to the running target, without source code available. 

## Training on real security work

- Base model
- GLM-5.3-Flash
- Method
- Rank-256 LoRA + selective full-parameter training, GRPO
- Training tasks
- 150, from 50 cases
- Role
- Worker model, orchestrated

We started with GLM-5.3-Flash and ran a fine-tune using GRPO reinforcement learning.

Our security work gives us a growing collection of vulnerability patterns and the context needed to turn them into training environments. We build on functioning open-source applications and protocols, introduce realistic flaws, and preserve the surrounding complexity that makes an investigation meaningful.

Turning those environments into useful training data requires another layer of work. We vary the information available to the agent and calibrate difficulty so that successful investigations are within reach without making the answer obvious. As models improve, we can revisit these environments with less guidance and more demanding combinations of flaws. This ability to keep creating, validating, and refining cases is central to the system we are building.

The size of the training set also shaped how we updated the model. We used rank-256 LoRA across all experts and routers, combined with full-parameter updates to 16 experts selected based on activation. Updating all parameters across all experts caused the model to collapse in our experiments. We suspect the small RL dataset led to misattributed gradients in experts not suited to the agentic workload.

Building a credible security RL gym takes more than introducing a bug. The environment, available information, and success checks all need to support a coherent investigation. A task can become too leading if its setup makes the answer obvious, or effectively impossible if essential information is missing or the objective cannot be reached. Task prompt wording is a critical part of this balance: naming a function or pointing to a subsystem can dramatically change solve rates. Recreating the complexity of production systems makes this calibration particularly difficult.

This matters for methods such as GRPO, which compare outcomes across a group of rollouts for the same task. With a binary success reward, groups in which every attempt succeeds or every attempt fails provide no contrast in that outcome signal. Cases the model can solve some of the time offer a trainable comparison. Our approach emphasizes realistic environments, clear verification, and difficulty appropriate to the model's current capabilities.

That constraint also shaped what we built apex-flash-1 to do. We designed it as a strong worker model, intended to be orchestrated by a larger model that gives it a focused security task. Training a smaller model on challenges it almost never solves would provide little useful GRPO signal, so we calibrate difficulty to its current capability while preserving the investigation and exploit work. Once focused, the worker can be tenacious: reading code, using tools, testing hypotheses, and verifying a finding. We have found that this division of work lets smaller models uncover bugs that still require deep investigation.

## Anatomy of a training case

Each training case is a running system with a defined security objective. We construct the service, seed it with realistic synthetic data, and give the agent a task, tools, and scoped access. Source code is included for whitebox tasks. The diagram below connects environment setup, agent investigation, verification, and training.

Verification and case validation serve different purposes. The verifier checks whether the agent achieved the objective through the intended bug and assigns a reward. We also review apparent successes for unintended shortcuts, then repair and recheck affected cases before reuse.

Scroll horizontally to explore the diagram.

### Example: artifact isolation in Forgejo

In one such environment, we recreated a workflow-artifact authorization failure in Forgejo, an open-source code-hosting platform. The bug class came from our discovery pipeline, and the scenario was constructed around systems we had encountered in production security work.

The agent began with access to one running workflow. A separate workflow held a protected artifact. The security boundary was straightforward: permission to retrieve one workflow’s outputs should not grant access to another’s.

We introduced a defect in how artifact access is authorized. A signed download URL omits part of the workflow context that the application later uses to select an artifact. Understanding the bug required connecting two parts of the system: what the signature authorizes and what the download handler actually retrieves.

The agent investigated that mismatch and demonstrated its effect against the running application. The verifier checked whether it recovered the protected artifact’s contents, giving us a concrete outcome to train against.

This reflected an authorization failure we encounter in production: individual components appear reasonable in isolation, but disagree about the scope of a permission. Recreating that pattern in a functioning system preserves the context that makes it challenging to investigate.

## Performance and cost

The result is a fine-tune of GLM-5.3-Flash that consistently outperforms the base model by 5-8% on our internal held-out evaluation. Across the full 60-task run, we estimate a cost of $2.38 for apex-flash-1, compared with $4.56 for GLM-5.3-Flash and $74.68 for Opus 5 High (using provider pricing).

Public cyber evals are useful, but they still simplify security work into relatively clean, academic tasks. We built our own environments to look much closer to the work we actually do for customers, including real exploitable bugs we have found, responsibly disclosed, and in many cases received bounties for.

We have post-trained a range of open models, from roughly 27B parameters up to 1T. apex-flash-1 is the first one we think meaningfully moves the Pareto frontier for real-world cyber tasks. That balance of performance and cost matters if we want capable security agents to become cheap enough to scale to every company, not just the few that can afford frontier-model economics.

-   Our model Pass@1: 66.7% 40 of 60 tasks - Run cost
- $2.38
 
-  GLM-5.3-Flash Base model Pass@1: 60.0% 36 of 60 tasks - Run cost
- $4.56
 
-  Claude Opus 5 High Reference Pass@1: 71.7% 43 of 60 tasks - Run cost
- $74.68
 

## Road ahead

This release is a preview of a broader training effort. We are already training on new environments built from years of novel security work, including bugs we have found and responsibly disclosed to companies. We are also expanding the mix of systems, vulnerability classes, and multi-step exploit chains, making the training cases more demanding as the models improve, and training across different agent harnesses so these capabilities carry over across the tools researchers actually use. The expanding data mix includes environments built from public CVEs.

Explore Apex to see how this work supports production security.
