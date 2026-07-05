---
url: https://github.com/taleshape-com/shaper
date_fetched: 2026-07-05
backfilled: true
---

**Open source, SQL-first data dashboards, reports and customer-facing analytics.**


Want us to run it for you?We offer managed hosting where your data stays in your infrastructure. See plans and pricing.

Build analytics dashboards simply by writing SQL:

```
SELECT 'Sessions per Week'::LABEL;
SELECT
  date_trunc('week', created_at)::XAXIS,
  category::CATEGORY,
  count()::BARCHART_STACKED,
FROM dataset
GROUP BY ALL ORDER BY ALL;
```
Learn more: https://taleshape.com/shaper/docs/

**Data Visualization**

- **Open Source**& Self-Hosted
- **SQL-First**and AI-Ready
- **Git-Based**Workflow
- Query across **Data Sources**

**Embedded Analytics**

- **White-Labeling**& custom styles
- **Row-level security**via JWT tokens
- Embed **Without IFrame**through JS & React SDKs

**Automated Reporting**

- Generate **PDF, PNG, CSV & Excel**
- Scheduled **Alerts & Reports**
- Sharable **Password-Protected Links**

The quickest way to try out Shaper without installing anything is to run it via Docker:

`docker run --rm -it -p5454:5454 taleshape/shaper`Then open http://localhost:5454/new in your browser.

For more, checkout the Getting Started Guide.

To run Shaper in production, see the Deployment Guide.

Shaper is 100% free and open source. Through **Taleshape**, we offer managed hosting and hands-on support for teams who need help getting analytics into production:

- **Managed Hosting**: We run Shaper for you—in our cloud or your infrastructure. We handle updates, security, and monitoring.
- **Hands-On Support**: Help with integrations, building dashboards, and meeting compliance requirements.

Feel free to open an issue or start a discussion if you have any questions or suggestions.

Also follow along on BlueSky or LinkedIn.

And subscribe to our newsletter to get updates about Shaper.

See CONTRIBUTING.md

See Github Releases

Shaper is licensed under the Mozilla Public License 2.0.

Copyright © 2024-2026 Taleshape OÜ
