---
url: https://newsletter.semianalysis.com/p/how-ai-labs-are-solving-the-power?hide_intro_popup=true
title: "How AI Labs Are Solving the Power Crisis: The Onsite Gas Deep Dive"
author: Ajey Pandey, Jeremie Eliahou Ontiveros, Dylan Patel
date_fetched: 2026-05-14
date_published: 2025-12-30
topics:
  - ai-infrastructure-and-hardware
---

# How AI Labs Are Solving the Power Crisis: The Onsite Gas Deep Dive

SemiAnalysis analysis of how AI labs are bypassing the electric grid by deploying onsite gas generation. Covers gas turbines (aeroderivative, industrial, heavy-duty), reciprocating engines (RICE), and solid-oxide fuel cells (Bloom Energy). Details the prisoner's dilemma of grid interconnection queues, equipment lead times, manufacturer capacity, supply chain bottlenecks in turbine blade casting, and the economics of "bring your own power."

## Summary

AI power demand projected to grow from ~3GW (2023) to 28GW+ by 2026. Grid interconnection timelines have stretched to 5 years. AI cloud revenue of $10-12B per GW annually means getting online six months earlier is worth billions. xAI set the template by deploying 500MW+ of truck-mounted turbines at Colossus, bypassing the grid entirely.

## The Grid Problem

- ERCOT: tens of GW of datacenter load requests monthly, barely 1 GW approved in 12 months
- AEP Ohio: 35 GW of load requests, 68% didn't even have land control
- Interconnection request to commercial operation: now 5 years
- Speculative requests create a prisoner's dilemma clogging the queue

## Equipment Landscape

### Gas Turbines
- **Aeroderivatives**: GE Vernova LM2500 (34 MW), LM6000 (57 MW). $1,700-2,000/kW, 18-36 month lead times. Essentially jet engines bolted to the ground.
- **Industrial Gas Turbines (IGT)**: $1,500-1,800/kW, 12-36 month lead times.
- **Heavy-Duty**: E/F/H-class from GE Vernova, Siemens, MHI. CCGT achieves 50-80% better efficiency than simple-cycle but 30+ minute cold start.

### Reciprocating Engines (RICE)
- High-speed: 3-7 MW. Medium-speed: 10-20 MW.
- $1,700-2,000/kW, 15-24 month lead times.
- Less dependent on critical minerals than turbines.
- Maintenance burden: a 2 GW plant with 5 MW engines = 500 units, 2,000+ services/year.

### Fuel Cells (Bloom Energy SOFC)
- No combustion, electrochemical reaction. No material air pollution besides CO2.
- $3,000-4,000/kW. Stack life ~5-6 years, replacement is ~65% of service costs.
- Easier EPA permitting.

## Supply Chain Bottlenecks

Turbine blade production concentrated in four firms: Precision Castparts, Howmet Aerospace, CPP, Doncasters. These are a fraction of their customers' size and were hit hard by COVID aerospace declines plus the post-2001 gas turbine bust.

Critical materials in turbine blades: rhenium, cobalt, tantalum, tungsten, yttrium in exotic monocrystalline nickel alloys.

## Manufacturer Scale-Up

- GE Vernova: returning to 24 GW/year (2007-2016 levels)
- Siemens Energy: scaling from ~20 GW to >30 GW by 2028-30
- MHI: 30% increase (not the "double" Bloomberg claimed)
- Caterpillar: 2x engines, 2.5x turbines by 2030
- Wartsila: incremental only

## Deployment Patterns

- **Bridge power**: operate before grid connection. Training workloads tolerate lower uptime, so overbuilding for redundancy can be avoided.
- **Off-grid permanent**: VoltaGrid's Shackelford County uses 2.3 GW of systems for a 1.4 GW datacenter. Matching grid uptime requires overbuilding.
- **Hybrid fleets**: Meta/Williams Socrates South runs 30 units across 5 turbine/engine types producing 306 MW.

## New Entrants

- ProEnergy PE6000: retrofits CF6-80C2 cores from Boeing 747s
- Boom Supersonic "Superpower": 42 MW aeroderivative, 1.2 GW already booked by Crusoe

## TCO Analysis

Behind paywall. Not extracted.
