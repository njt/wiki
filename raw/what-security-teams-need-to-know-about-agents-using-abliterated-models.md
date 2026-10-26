---
url: https://eve.security/blog/what-security-teams-need-to-know-about-agents-using-abliterated-models
date_fetched: 2026-10-26
---

How to detect usage in Shadow AI, use dynamic policies, and prevent future connections

In July 2026, the OpenAI and Hugging Face incident proved how powerful agents and the models underneath them can be when left without safeguards. OpenAI’s agents, running models without the regular safeguards, managed to break out of a sandbox during a cybersecurity evaluation, creating exploits, coordinating through a hidden communication channel, and harvesting cloud credentials and private evaluation data.

In today’s landscape, there are other practical real world use-cases where agents need to run without the safeguards that are present in the frontier models used in agents every day.

For example, red-teaming requires agents that try to steal credentials or go after company data, which requires reducing or removing the model safeguards. In the OpenAI and Hugging Face incident, the team had to use an open weight model to investigate the incident because the safeguards on the frontier models were refusing the requests it needed to pass. In these kinds of situations, one process for removing these safeguards is called abliteration, which **surgically removes the refusal training** of open-weight frontier models (like GLM-5.3).

## How Abliteration Works

Abliteration is only possible with **open weight models** because the process requires weights you can actually load and run outside someone else’s API.

A model’s refusals are driven by one internal “refuse” signal. In order to find the direction in the model’s internal activation space that corresponds to “refuse this request,” you have to make forward passes through the model and modify its weights or activations directly. That means you need to have the weights themselves. Abliteration is very effective, and Anthropic has shown that abliterating GLM-5.3 resulted in 100 percent compliance with harmful cyber-attack instructions in testing. Anyone who downloads GLM-5.3’s, Llama’s, or Qwen’s weights can abliterate a copy on their own GPUs, cloud or on-prem (it doesn’t have to be literally local hardware) and then redistribute or use that stripped-down version.

**Abliteration doesn’t work against a model accessed purely through an API**, like Claude. If you only ever send text in and get text out, you never touch the weights or internal activations.

## What Security Teams Need to Know

### A neutral, third party is required

The security outcome of an agent’s actions always depends on the task it has been assigned, and how it uses tools, chains context and decides at each step in its lifecycle. The model is just one piece of the broader toolset, behavior and context that chains together into an expected, unexpected, malicious or benign outcome. When thinking about controlling for risks like an abliterated model, while better safeguards for the model itself are important, the important governance and control still lies in **a neutral party observing and restricting the agent where needed.**

### Set custom policies for the agent and interrogate if it goes too far

When you are using the abliterated agent for research or pen-testing, you need to be able to set custom policies that can work globally, that keep the agent in its sandbox or prevent it from materially harming an organization in the process.

This will be a completely different set of controls than what you would have gotten in a model with its safeguards turned on. A model judges whether a kind of content should exist at all. **Policy, by contrast, enforces scope and authorization at the action layer:** which systems an agent is allowed to touch, which tool calls it can make, and what counts as new or anomalous for that specific agent.

A red team running an abliterated model inside a sandbox still needs policy to keep that activity bounded to the authorized target and fully logged, not to stop the model from being willing to write exploit code, but to make sure that code never finds a path to anything outside the scope and target.

Even so, **policy alone has a known limit:** it has to be written in advance, and an abliterated model is, by definition, good at finding paths a static rule didn’t anticipate.

This is where it’s reasonable to expect agent interrogation to play a complementary role. Eve Security’s interrogation approach pauses an action when it’s both unusual for that specific agent and above a configured risk threshold, requiring the agent to justify the action in its own words before anything executes, **with that justification checked against real context**, such as a verified change ticket, rather than taken at face value. In short, interrogation is a way to go deeper with verification than a surface-level policy might be able to, in terms of the knowledge of what the agent is actually doing under the covers.

### Get visibility into Shadow AI

When an unknown agent with an abliterated model is behaving maliciously, you need to be able to detect and stop this inside your organization. An agent program is usually a list of agents, MCP servers, or SaaS bots, but an API request to an abliterated model can hide in something as simple as a local Python binary. This won’t show up in that agent program, and the agent’s model also will not show up in Cursor’s settings, for example, because all that is happening in a program is opening a connection to a hostname that happens to be a cloud model API.

From the perspective of a firewall, ordinary developer HTTPS would be visible. But for abliteration, what you need to be looking for is the hostname, at least in the case that you are connecting to something like ``api.abliteration.ai,`` which is a known model API.

Because Eve is connected through the endpoint, we can see the Python process reaching the API, on the signal of a managed machine calling that host. To detect this, or any connection to an API from an approved agent, you need to **tie the API call to the process, user and destination.**

Below is an image of what a detection would look like if an employee created a custom agent in your organization using an abliterated model.

### Preventing a Connection to an Abliterated Model

Once you’ve seen the malicious agent, you need a way to stop it, and prevent anything like that from being called again in the organization. Once Eve sees the Shadow AI through the endpoint, it blocks that API call through the EDR, and **sets a global policy to block any future calls like that.** And once Eve has seen it in one customer environment, that policy can be applied automatically to others, preventing others from having the same issue.

In the case of writing policies to block the call, the details and timing matters. Writing and improving on these kinds of policies must happen at agent speed in an automated, self-improving process.

To protect environments where abliterated models are being run, you need the ability to automate policies, benefit from shared learnings at scale, and block the connection using the tool stack you already have.

## Conclusion

Abliteration is a new class of risk enabled by the proliferation of open weight models, and is hauntingly similar to what caused the recent flood of acknowledgements of agents finding ways out of their sandboxes in the frontier models. That’s why it’s key to detail out now what security teams can do, whether they have a valid use for abliterated models, or are trying to prevent them from being used at all. To safely adopt today’s agents, effective governance calls for a dynamic suite of enforcement tools that sit neutrally outside the agent, automating policy creation, preventing future issues based on today’s learnings, and diving deep where the uncertainty is too risky.
