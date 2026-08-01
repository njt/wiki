---
url: https://dl.acm.org/doi/epdf/10.1145/3800646.3800650
title: "Stop Blaming QA and Start Diagnosing the Hidden Inefficiencies Behind Delivery Delays: Five Drivers that Slow Releases And Erode Developer Productivity"
author: Ahmed El-Deeb
date_fetched: 2026-07-21
date_published: 2026-04
---

Ahmed El-Deeb argues that the common reflex — blaming QA for delivery delays — is a misdiagnosis driven by visibility bias: QA is a named stage with timestamps, so it's the easy target. In reality, deeper structural inefficiencies consume far more time and yield no value.

The article identifies **five primary drivers** of delivery inefficiency, each with measurable indicators and root causes:

1. **Instability-driven rework.** Unplanned hotfixes, rollbacks, and incident response. DORA reports unplanned work and rework consume ~20% of time. Root causes include large-batch releases, insufficient pre-deploy validation, and tightly coupled dependencies.

2. **Priority churn and requirement volatility.** Constant pivots and mid-delivery scope changes cause context switching, coordination overhead, and reduced throughput. A study of 22,771 requirements found more than half were modified at least once; 63.2% of "highly volatile" requirements accounted for 80% of total changes.

3. **Cross-team dependency overhead.** When teams cannot deploy or test independently, coordination costs multiply. Uber's experience — gating every change through 1,000+ services with thousands of E2E tests — illustrates the drag. Key root cause: Conway mismatch between architecture and organizational communication patterns.

4. **Technical debt and maintainability drag.** Meta reports over 14% of changes are explicitly code improvement rather than feature work. Time pressure creates delivery-only incentives that accumulate debt; one longitudinal study found ~23% of effort wasted due to technical debt.

5. **Review and approval queueing.** PR dwell time, reviewer scarcity, and approval gatekeeping create long-tail delays. Microsoft found that on "bad days" involving code reviews, PR dwell time was 48.84% higher. Stripe's survey found developers spend 17.3 hours/week on maintenance and 13.5 hours/week on technical debt.

Secondary signals — meetings, operational toil (Google: ~33% average toil, outliers up to 80%), and the developer productivity gap (actual coding ~11% of time vs. ideal ~20%) — are downstream symptoms of the top five, not root causes themselves.

The article also catalogs **gaps between theory and practice**: CI/CD theory assumes smaller changes are always safer, but ignores system-level coupling and organizational response patterns. Agile assumes low priority-switching cost, which empirical data contradicts. Microservices theory promises decoupled teams but runtime and data dependencies create hidden coupling.

Finally, El-Deeb proposes research directions: instability propagation models, cognitive reset cost modeling, requirement stability indices, quantitative socio-technical coupling metrics, reviewer load balancing algorithms, and incentive-sensitive debt models.
