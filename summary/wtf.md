---
url: https://www.andrewt.net/dithered-qr-codes/wtf/
title: "Dithered QR codes"
author: Andrew T
date_fetched: 2026-08-14
date_published: unknown
---

# Dithered QR codes

Andrew T (andrewt.net) explains how to embed a photograph into a QR code so it still scans, using dithering and error diffusion.

A QR code splits into function patterns (the bold finder squares a scanner locks onto) and data modules (where the payload and its headers live). Because a scanner reads data modules only after the finder patterns fix the grid geometry, those modules tolerate a surprising amount of vandalism — which is how branded and logo QR codes work. The trick here is to shrink each data module to the centre cell of a 3×3 grid and give the surrounding eight cells to a photo. The result is a 147×147 one-bit image, plus salt-and-pepper noise wherever a data module's forced colour clashes with the picture.

The rest of the post is about taming that noise. Naive thresholding is replaced with dithering (Bayer, then Floyd–Steinberg error diffusion), which spreads each pixel's quantization error into its unprocessed neighbours so the image's average brightness stays right and the irregular speckle hides the forced data bits. The key move is a second error-diffusion pass that runs first: it sets the known data-module colours in advance and diffuses the resulting error (sometimes 95% of a pixel) outward, so the required bits vanish into the picture instead of sitting on top of it.

Two honest caveats close the post. First, aesthetics and scannability are a genuine trade-off — a pretty code on a laptop screen may not survive a crumpled paper flyer on a stranger's "potato phone," because you have spent the redundancy that error correction would otherwise use. Second, the generator emits tiny, margin-less images; a real code needs a quiet-zone margin (the opposite colour to the finder patterns) and CSS to stop browsers blurring the upscale.
