---
url: https://arxiv.org/html/2607.20300v1
title: "Don't Trust the Label: License Laundering in AI Supply Chains"
author: James Jewitt, Hao Li, Gopi Krishnan Rajbahadur, Bram Adams, Ahmed E. Hassan
date_fetched: 2026-07-25
date_published: 2026-07-22
---

# Don't Trust the Label: License Laundering in AI Supply Chains

James Jewitt, Hao Li, Gopi Krishnan Rajbahadur, Bram Adams, Ahmed E. Hassan — all affiliated with Queen's University

arXiv preprint, arXiv:2607.20300v1 [cs.SE], 22 Jul 2026

## Abstract

The study traces 232,270 dataset→model→application chains across Hugging Face and GitHub to measure two forms of "license laundering": (1) Unknown laundering—artifacts with no declared license acquiring labels downstream, and (2) Category laundering—one declared license category replacing another during redistribution. Key headline: 62.3% of chains pass through at least one artifact with no declared license, and every obligation-bearing license category falls below 7% end-to-end survival while Permissive reaches 95.1%.

## Key Concepts & Definitions

**License Laundering:** Described as "the stripping or replacement of the legal rights and obligations attached to an asset as it moves through an AI supply chain."

- **Unknown laundering:** Artifacts with no declared rights acquire definitive licenses downstream, implying legal certainty where none was established.
- **Category laundering:** One declared license category is replaced with another during redistribution (e.g., Sharealike obligations vanishing under an Apache-2.0 label).

**Legal Risk:** An upstream rights holder can assert copyright at any time, forcing takedown, mandatory relicensing, or financial liability.

### Case Study Cited: Books3

The dataset packaged 196,640 books from Bibliotik, circulated under an MIT label despite no downstream rights being granted by rights holders. Models trained on it (Claude, LLaMA, BloombergGPT) shipped under their own license terms. The Danish Rights Alliance forced Books3's withdrawal in 2023. The Bartz v. Anthropic case resulted in a $1.5B settlement (approved July 2026)—the court held that while training was fair use, acquiring and retaining pirated copies was not.

## Methodology

### Supply Chain Construction

- **Data sources:** Hugging Face (datasets & models) + GitHub (applications)
- **Linking:** Datasets→models via Hugging Face metadata; models→applications via code search and AST analysis on GitHub
- **License extraction:** ScanCode used for GitHub repositories
- **Initial collection:** 3,198 datasets, 6,218 models, 26,302 applications → 294,012 candidate chains
- **After filtering:** 232,270 chains spanning 3,120 datasets, 5,556 models, 24,076 applications

**Two filters applied:**
1. Removed models serving only as base models with no direct application invocation
2. Excluded chains with license strings not mapping to the classification framework

### License Categorization

765 unique license strings observed → grouped into 7 categories:

| Category | Examples |
|---|---|
| Permissive | MIT, Apache-2.0, BSD-3-Clause |
| Copyleft | GPL, AGPL |
| Sharealike | CC BY-SA, LGPL, MPL |
| ML License | OpenRAIL, Llama Community License |
| CC-Restrictive | NonCommercial, NC-SA, NC-ND, NoDerivatives |
| Public Domain | CC0, Unlicense |
| Unknown | Empty fields, "Unknown," "Other" |

270 strings were classified against FSF, CC guides, and Stalnaker et al. frameworks. 495 strings remained unidentified (custom/ambiguous). Artifacts can carry multiple categories simultaneously.

## Key Findings

### Finding 1: Unknown Laundering (62.3% of chains)

- 144,631 of 232,270 chains (62.3%) pass through at least one artifact with an Unknown license.
- 88.8% of chains with an Unknown dataset reach a Known model.
- 80.3% of chains with an Unknown model reach a Known application.
- Inverse pattern: 24.9% of chains with a Known-licensed model end at an application with no declared license.
- 56,683 chains (39.2%) involving an Unknown artifact end at an Unknown application.

**Concentration:** The top 10% of Unknown datasets account for 89.5% of dataset→model laundering transitions. High-fan-out examples include ImageNet-1K (Unknown license, spreading to 243 models and 1,709 applications), The Pile (2,139 applications), and BookCorpus (1,695 applications).

### Finding 2: Category Laundering (37.5% of fully-Known chains)

- Of 87,639 chains where all artifacts carry Known licenses, 32,901 (37.5%) drop at least one category.
- Most laundering occurs at dataset→model (65.4% retention) vs. model→application (86.4% retention).
- 23.9% drop categories only at dataset→model; 5.5% only at model→application; 8.1% at both hops.

**Survival rates by category (end-to-end):**

| Category | End-to-End Survival |
|---|---|
| Permissive | 95.1% |
| Copyleft | Below 7% |
| Sharealike | 4.7% (739 of 15,678 chains) |
| ML License | ~4.2% model→application (moderate dataset→model at 55.8%) |
| CC-Restrictive | Below 7% |

**Collapse toward Permissive:** When obligation-bearing categories are lost, "the downstream license almost always becomes Permissive rather than another category." For dataset→model, 93.1% of Copyleft transitions land on Permissive. For model→application, 77.9% of ML License transitions land on Permissive.

**Notable asymmetry:**
- ML License survives dataset→model (55.8%) but not model→application (4.2%).
- Copyleft does the opposite (0.9% dataset→model, 39.0% model→application).
- The authors note that Copyleft benefits from "decades of tooling support" in the software ecosystem, whereas ML License terms lack operational enforcement mechanisms.

## Threats to Validity

**Internal:** The study tracks license labels (metadata), not full legal text. If labels don't reflect publisher intent, analysis captures "the label as practitioners encounter it." Some vague strings ("other") classified as Unknown may overstate the Unknown count.

**External:** Only 7.1% of Hugging Face models declare training data via metadata. The sample captures popular artifacts with widest downstream reach. Chains with unclassifiable license strings (495 of 765 unique) were excluded but represented only 12.2% of pre-filter chains, and the authors state "Every standard obligation-bearing license" falls within the 270 recognized strings.

## Implications & Recommendations

### For Practitioners

Verify upstream before integrating—a model's license alone is unreliable. Check declared training datasets on Hugging Face, classify upstream licenses, and treat conflicts or absences as integration risk.

### For Model Publishers

Declare upstream dataset licenses, not just the model's own. The datasets field in Hugging Face model cards already exists; populating it "requires no new infrastructure, only a change in documentation practice."

### For Rights Holders

A restrictive license alone is not enforcement. Given that obligation-bearing categories survive below 7%, rights holders "should pair the license with enforcement mechanisms outside metadata: contractual terms or gated access."

### For Platform Owners & Tool Builders

Build cross-platform license tracing tools. Existing SCA tools (ScanCode, FOSSology, Black Duck) operate on single-platform, package-manager dependencies. AI chains span Hugging Face and GitHub, mix three legal paradigms, and declare dependencies via metadata tags. SPDX 3.0 and CycloneDX offer BOM profiles for AI but "nothing populates them automatically." The paper calls for tools that auto-generate AI BOMs from existing metadata.

## Conclusion

The paper provides the first empirical measurement of license category changes across the full AI supply chain. The core finding is that license laundering is not rare or isolated—it is routine and systemic. Unknown laundering concentrates in foundational datasets, and Category laundering systematically strips obligation-bearing licenses, collapsing them into Permissive labels. Until automated cross-platform tracing infrastructure exists, the authors argue that manual upstream verification remains "the only safeguard."
