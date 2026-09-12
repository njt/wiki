# Dithered QR Codes

A walkthrough of how to embed a photograph into a QR code — shrink the data modules, dither the image into the freed space, then run a second error-diffusion pass to hide the forced data bits — and an honest account of what the trick costs in scannability.

---

## Key Quotes

> "one in nine of the pixels are effectively random colours"

The source of the noise, named precisely. Each data module's colour is fixed by the encoded payload, so when you overlay a photo, one cell in every 3×3 block is whatever the QR spec demands rather than what the picture wants. The author says "random" but means "forced by data you don't control" — and locating the noise is what makes it solvable.

> "But we can solve that using *more error diffusion*."

The post's real insight, hidden in a throwaway line. The first pass of error diffusion *masks* the required bits rather than drawing them: set the data modules to their known colours first, then diffuse the resulting error outward so the forced pixels melt into the surrounding image. The same algorithm run twice, doing two opposite jobs.

> "That sounds bad, but that's exactly why it's so important to diffuse this error instead of just accepting it."

Why the technique exists. A forced pixel can be 95% wrong for its neighbourhood — far beyond the ≤50% error you would normally tolerate — and only spreading that error into the free pixels keeps the image from looking like a hole punch.

> "Ultimately it's a trade-off between aesthetics and scannability — and remember that just because a code scans on *your* phone, from a laptop screen, doesn't mean it will scan on a random stranger's potato phone from a printout in bad lighting."

The thesis, and the sentence that makes the piece more than a party trick. Every redundancy you spend on the picture is redundancy you can't spend on surviving a crumpled flyer.

## Key Themes

#concept #tool #image-processing #quantization

### Redundancy is a budget

A QR code's error correction is slack you can spend. Logo-in-the-middle codes spend a little; this spends nearly all of it, shrinking each data module to one cell in nine and giving the other eight to the photo. The post is really about knowing what you're spending and what it costs, not about the specific pixels.

### Error diffusion as concealment

The same Floyd–Steinberg loop does two opposite jobs. Run normally, it makes a one-bit image *look* right. Run first, over pixels whose colours are already decided, it *hides* those pixels by pushing their error into the neighbours. Reusing one mechanism for two purposes is the kind of move that shows up only when someone understands the algorithm rather than just calls it.

### The honesty clause

The generator is a toy, and the author says so: tiny, margin-less, blurry-when-upscaled, and only reliable on a big clean screen. Listing the failure conditions — crumpled paper, potato phones, the quiet-zone margin, CSS `image-rendering` — is what separates a neat hack from a tool you would ship.

## Critical Analysis

This is a model of explaining a hack well: start from "what is a QR code," teach just enough dithering to follow along, then land on the non-obvious move. The article's real subject isn't QR codes — it's the recurring idea that you can spend redundancy, and spend quantization error, *deliberately*.

The two-pass error diffusion is the genuinely reusable pattern. Whenever you have pixels whose values are forced — a watermark, a hidden bit plane, a logo — don't just set them and eat the discontinuity; diffuse their error into the free pixels so the forced ones disappear. That generalises far beyond QR codes.

The contrast with [[QR Generator (delphi.tools)]] is instructive: delphi.tools surfaces error-correction levels so the user can *preserve* scannability while embedding a logo; Andrew T *spends* that same budget to make the whole code a picture. Same mechanism, opposite priorities.

What the post skips is quantification: how many modules can you actually flip before it stops scanning? It says "a few" and moves on. That's arguably the right level for a blog post — the exact Reed–Solomon math would bury the insight — but it leaves "how far is too far" to trial and error, which is exactly where the potato-phone warning bites.

The deeper rhyme is with [[Binary Vector Embeddings]]: both discover that crushing a continuous signal to one bit destroys far less than intuition predicts, provided you're clever about where the error goes. Dithering is to a photograph what binary quantization is to an embedding — the "95% error" this post frets about is the mirror image of the "95% retention" that post celebrates.

[[Draw Your Font]] sits at the opposite end of the same spectrum: its adaptive threshold must preserve every ink pixel faithfully so potrace can trace the letterforms, whereas dithering *deliberately* distorts pixels to fake midtones. Faithful binarization versus deceptive binarization — same one-bit destination, opposite ethics.

## Cross-Links

- [[QR Generator (delphi.tools)]] — error correction exposed as a knob to preserve scannability, versus spent as a budget for aesthetics
- [[Draw Your Font]] — the faithful end of one-bit image processing (adaptive threshold), against this page's distorting end
- [[Binary Vector Embeddings]] — the "one bit is enough" quantization rhyme: how much survives the crush, and where the error goes

---
*Sources: [[raw/wtf]], [[summary/wtf]]*
*Last updated: 2026-08-14*
