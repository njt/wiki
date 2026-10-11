---
url: https://uipalooza.ai/experiments/browser-comparison/
date_fetched: 2026-10-11
---

### Rendering affects the result

Obscura loses door text in Sidewalk, changes image crops and wrapping, and drops labels in Elsewhere’s map. SleepyHollow keeps the overall composition, with native-control and typography differences.

A test of the tools that test taste

Chrome, Obscura, and SleepyHollow render the same 12 saved designs. Desktop and mobile. No new model runs.

Jump to the screenshots ↓RECOMMENDATION

SleepyHollow preserves the designs closely enough to merit a managed preview pilot. Obscura’s rendering and interaction differences would distort this benchmark. Neither should silently replace the current evaluator.

| Measured in this pilot | Chrome | Obscura | SleepyHollow | 
|---|

Capture success means a command returned successfully and produced a PNG, not that the image is correct. Timings include startup, loading, the common ≥1 s settling gate, and capture; background generation and some interaction tests were active. Pixel differences include font rasterization and are not design scores.

Obscura loses door text in Sidewalk, changes image crops and wrapping, and drops labels in Elsewhere’s map. SleepyHollow keeps the overall composition, with native-control and typography differences.

SleepyHollow’s selector click activates a covered button. Its coordinate click correctly hits the overlay. Obscura’s covered-button click returns success without activating the button; its Enter key also fails to activate our focused button.

The alternative captures save roughly a third of a second at the median here. That does not resolve a model spending 30 minutes debugging browsers. A controlled preview tool and reliable lifecycle matter more.

THE EVIDENCE

Click a screenshot to inspect it. Use “Overlay comparison” to slide between Chrome and another engine. Review notes are an assistant’s qualitative assessment of these captured viewports, not a blinded human study.

Four briefs × three saved designs × two viewports × three engines = 72 primary images. Two extra desktop captures for each Whacky design add 24 repeatability images. All 96 images, source hashes, measurements, and observations are saved locally.

Matched viewport dimensions, 1× screenshot scale, light theme, temporary browser state, and a common readiness probe. A shared proxy disables CSS animation and sends a same-origin resource policy. The corpus is fixed to original artifacts; the main evaluator and saved model results are unchanged.

These are viewport captures, not a full-page, touch-device, or exhaustive Web API test. Browser preferences, native controls, lazy-loading, and font rasterization can still differ. Memory was not ranked: WebKit’s system-managed helper processes and the active generation queue make a simple process-RSS comparison misleading.
