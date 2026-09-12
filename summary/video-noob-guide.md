---
url: https://gist.github.com/arch1t3cht/b5b9552633567fa7658deee5aec60453/
title: "What you NEED to know before touching a video file"
author: arch1t3cht
date_fetched: 2026-05-15
date_published: unknown (gist, ~303 stars, 22 forks, 54 revisions)
topics:
  - software-engineering-craft
---

# What you NEED to know before touching a video file

An in-depth educational guide for novice video editors and encoders, written after the author observed repeated beginner mistakes in subtitling and video re-editing communities.

## 1. The Anatomy of a Video File and Remuxing vs. Reencoding

.mp4 and .mkv are container formats, not encoding formats. The actual video coding formats (H.264/AVC, H.265/HEVC, VP9, AV1, ProRes) do the real compression work. Container formats package already-compressed streams, storing multiple audio/subtitle tracks, chapters, and metadata.

**Reencoding** = decoding and re-encoding a video stream (lossy, time-consuming).
**Remuxing** = changing the container without touching the encoded stream (fast, lossless).

Tools like Handbrake or online converters often reencode when users just want a container change. Distinguishes H.264 (the format) from x264 (a specific encoding program).

## 2. Video Quality

Lists common misconceptions about what determines quality (resolution, frame rate, bit depth, file size, file format, codec, encoding program/settings, source, colors, sharpness) and explains why none of these alone determines quality.

**The Encoding Program and its Settings:** H.264 only specifies decoding, not encoding methodology. x264 and x265 are the best encoders for quality. Hardware encoders like NVENC prioritize speed over quality. CRF (Constant Rate Factor) is the main quality control — lower CRF = higher quality, larger file.

**Interlude: So Then, What is Quality Actually?** Quality is "how closely it resembles the source it was created from." Quality is relative to some reference/ground truth. Automated metrics (PSNR, SSIM, VMAF) exist but none are perfect.

### Mythbusting

- **File Size/Bitrate:** More bits doesn't automatically mean better quality — efficiency varies by encoder.
- **Video Coding Format:** The tool and settings matter more than the format itself. "HEVC is 50% more efficient than AVC" is "just plain wrong." AV1 excels at low-fidelity encodes; x264/5 are still ahead for high-quality/transparency encodes.
- **File Format:** Container format doesn't determine quality.
- **Resolution:** Higher resolution may not mean better quality, and lowering resolution may not be the best way to save file size. Adjusting CRF/bitrate is better than downscaling. AI upscaling "invents" detail and moves video further from its source. Digital anime is often produced below 1080p even in 2025.
- **Frame Rate:** "frame interpolation is bad. There's not even any nuance here this time, just don't do it."
- **Bit Depth:** Encoding at 10bit from 8bit source can actually increase efficiency due to how video coding formats work internally. Unlike resolution scaling, increasing bit depth "is not a destructive process (when done correctly)."
- **Video Source (Blu-ray vs Web):** Blu-ray doesn't automatically mean better. Some Blu-ray authoring companies apply destructive blur/lowpass filtering. "Always try to manually evaluate sources using your eyes."
- **HDR vs SDR:** HDR is not automatically better. Warns about artificially created HDR from SDR sources.
- **Colors:** "'brighter and more saturated' does not mean 'better'." Warns against "improving" colors in already-mastered content.
- **Sharpness:** "prioritizing sharpness above all else is not a good idea." Sharpening creates artifacts like haloing/ringing.

## 3. Summary of Key Takeaways

- You cannot judge quality by resolution and file size alone
- Use x264 or x265 with CRF rather than downscaling
- Don't change resolution, frame rate, colors, sharpness, or apply postprocessing without knowing exactly what you're doing

## 4. Learning to Spot Quality Loss

Areas to focus on: dark areas/gradients, strong colors (especially black edges on deep reds), grainy/textured areas, spaces around sharp lines (look for halos/ringing), text edges, image borders. Acceptability is subjective.

## 5. Color Space Parameters

Explains YCbCr color space and chroma subsampling (typically 4:2:0 where chroma is stored at half resolution in both directions).

**Key parameters:**
- **Color matrix:** BT.601 (older, SD-era) vs BT.709 (HD video). Wrong tagging causes color shifts.
- **Color range:** Limited vs Full (vast majority of videos use Limited range)
- **Transfer characteristics/gamma:** BT.709, BT.601, BT.1886, sRGB, PQ, HLG
- **Primaries:** BT.709, BT.601, BT.2020
- **Chroma location:** Default is "center left" for most videos. Incorrect handling causes chroma shift visible as colored glows on edges.

"If you see your reencode somehow changing colors, check your input and output video's color matrices." Tools: MediaInfo to check, MKVToolNix to edit.

## 6. Subtitles

ASS (Advanced SubStation Alpha) is the most powerful subtitle format. Only mkv fully supports ASS subtitles. Avoid hardsubbing when possible since it "involves reencoding, and hence introduces quality loss."

## 7. Recommended Tools

**Recommended:** MediaInfo, mpv, ffmpeg, SlowPics, MKVToolNix, MKVExtractGUI/MKVcleaver, Aegisub, MkvToMp4

**Tools to avoid:** Handbrake ("has a lot of footguns"), any file conversion websites ("just use ffmpeg directly"), Topaz AI, Anime4k, RealESRGAN, RIFE, imgsli for comparisons

## 8. Workflows

**General principle:** Reencode only once, at the very end. Use lossless intermediates where possible. Export lossless from editing software, then encode with ffmpeg.

**Encoding template:**
```
ffmpeg -i yourinput.mkv -c copy -c:v x264 -preset slower -crf 20 youroutput.mkv
```

Three-way tradeoff between file size, quality, and encoding speed (CRF regulates quality vs size; preset regulates speed vs efficiency).

## 9. Bonus: Interlacing

Interlaced-looking footage is often actually telecined (3:2 pulldown). Readers should "consult some more experienced person before blindly running a deinterlacer" on telecined footage.

## 10. The Rabbit Hole

Points to the JET Guide and the author's blog for further learning.

## 11. Footnotes (17)

Covering: codec terminology, x264 superiority, CRF vs bitrate, post-processing caveats, fractional frame rates, scaling/destructive operations, lowpassing in Blu-ray authoring, color "fixes," sharpening caveats, RGB color spaces, YouTube's color matrix, gamma complexity, chromatic aberration vs chroma shift, ASS jokes, transparency-targeting settings, MediaInfo reliability for CFR, and the term "telecine."

## Comments

The author states the guide's target audience is "users that make some form of edit of some video that they want to distribute or archive and who, at least to some extent, care about video quality." A commenter posted a TL;DR generated by Gemini 2.5 Pro; another criticized it as stripping "all of the learning, meaning and knowledge shared." The author explained their intentional lack of quantitative recommendations and acknowledged legitimate reencoding use cases.
