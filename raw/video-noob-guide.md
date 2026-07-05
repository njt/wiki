---
url: https://gist.github.com/arch1t3cht/b5b9552633567fa7658deee5aec60453/
date_fetched: 2026-07-05
backfilled: true
---

Hanging out in subtitling and video re-editing communities, I see my fair share of novice video editors and video encoders, and see plenty of them make the classic beginner mistakes when it comes to working with videos. A man can only read "Use Handbrake to convert your mkv to an mp4 :)" so many times before losing it, so I am writing this article to channel the resulting psychic damage into something productive.

If you are new to working with videos (or, let's face it, even if you aren't), please read through this guide to avoid making mistakes that can cost you lots of time, computing power, storage space, or video quality.

This guide is quite long.
This is hard to avoid since videos are *really, really complicated*,
and there are lots of misconceptions to clear up.
I have tried to keep the different sections as independent as possible so you do not have to read the whole thing at once.

Let's start out with the most important thing: The mistake I see the most and that causes experienced users the most pain to see.

To efficiently work with video files, you need to know the (extreme) basics of how video files are stored:
When you download video files or copy them somewhere, you may come across various types of videos.
You'll probably see file extensions like `.mp4` or `.mkv` (or many others like `.webm`, `.mov`, `.avi`, `.m2ts`, and so on).
As a newcomer to video you might be tempted to think that this file extension is what determines the video format.
You might have found an `mkv` file somewhere and noticed that Vegas or Premiere cannot open it,
so you searched for ways to convert your `mkv` file to an `mp4` file.

While this is technically not wrong, it's far from the full story and can cause lots of misconceptions.
In reality, all these formats are so-called *container formats*.
The job of an `mkv` or `mp4` file is not to compress and encode the video,
but to take an already compressed video stream and package it in a way that makes it easier for video players to play them.
Container formats are responsible for tasks like storing multiple audio or subtitle tracks (or even multiple video tracks!) in the same file,
storing metadata like chapters or which tracks have which languages,
and various other technical things.
However, while they *store* the video (and audio), they're not the formats that actually *encode* it.

*Actual* video coding formats are formats like H.264 (also known as AVC) or H.265 (also known as HEVC).
Sometimes they're also (somewhat incorrectly1) called *codecs*, short for "encoder, decoder".
H.264 and H.265 are the most common coding formats, but you may also run into some others like VP9 and AV1 (e.g. in YouTube rips) or Apple ProRes.
These are the formats that handle the actual encoding of the video,
which is the much, much, *much* harder part.
A raw video file is *massive*, so these formats use lots of very clever and complicated tricks
to store the video as efficiently as possible while losing as little quality as possible.
In particular, this means that these formats are usually *lossy*,
i.e. that video encoding programs will cause slight changes in the video in order to be able to compress it more efficiently.
However, figuring out how to make a video as small as possible while sacrificing as little quality as possible
is *very hard*, which is why encoding a video takes a lot of time and computing power.
This is why rendering a video takes as long as it does.

Note that *H.264*  is different from *x264*, which you may also have heard of. *H.264* is the coding format itself,
while *x264* is a specific program that can encode to H.264.
The same is true for H.265 and x265.
You will see later on in this article why this distinction matters a lot.

So, to summarize:
A video file is actually comprised of a *container format* (like `mkv` or `mp4`), which itself contains an actual video stream.
Changing the container format is simple: You just rip out the video stream and stick it into another container.
(Well, it's a little more complicated than that. But the point is: The container format is not the one that encodes the actual video, so you can switch container formats without encoding the video from scratch.)
Changing the underlying coding format, however, or recompressing the video to change the file size,
is harder and will a) take time and computing power, and b) lose video quality.

The process of decoding a video stream and encoding it again using the same or a different coding format is called *reencoding*.
Changing the surrounding container format, on the other hand, is called *remuxing* (Deriving from "multiplexing", which refers to sticking multiple audio or video streams into the same file).

This is *extremely important* to know when working with videos!
If you try to convert your `mkv` file to an `mp4` to open it in Premiere by sticking it into a converter like Handbrake (or, worse, some online conversion tool) without knowing what you're doing,
you may end up reencoding your video instead, which will not only take much, much longer,
but also greatly hurt your video's quality.

Instead, chances are that you can just remux your video to an `mp4` instead, leaving the underlying encoded video stream untouched.
Now, granted, there are some subtleties here, in particular to do with frame rates (more on this later),
but the point is: lots of simple-looking "conversion" methods (like Handbrake, random converter websites, etc.) will actually reencode the video,
which you want to avoid as much as possible.
Knowing how a video file is structured, and what tools you can use to work with them (again, more on this later)
will help you avoid many of these mistakes.

Next, let's talk about the concept of "video quality", which I myself already invoked above.
I don't think there is any other concept in video with as many misconceptions about it as video quality,
and once again misunderstanding it can cause you to make many avoidable mistakes.
This is important for both encoding your own videos *and* for *selecting* which source footage you want to work with.

Here is a list of things that people commonly associate with a video's quality:

- Its resolution (1080p/720p/4k/etc.)
- Its frame rate (24fps / 60fps / 144fps / etc.)
- Its bit depth (8bit / 10bit / etc.)
- Its file size or its bitrate (i.e. file size divided by duration)
- Its file format (`.mkv`/`.mp4`/ etc.)
- Its video coding format (H.264 / H.265 / etc.)
- The program used to encode the video (x264 / x265 / NVENC / etc.)
- The settings used to encode the video
- The video's source (Blu-ray / Web Stream / etc.)
- The video's colors (brightness / contrast / saturation / etc.)
- The video's color space and range (i.e. whether it's in HDR)
- How sharp or blurry the video is

If you've paid attention in the previous section, you should know that at least some of these points, like the file format one, cannot be true (but it's still a misconception I sometimes see!). But, in fact, the truth is that none of these things are necessarily related to a video's quality! The program used to encode the video combined with the settings used in them gets the closest, but only in specific scenarios.

Why is this? Well, let's go through them one by one (but in a slightly different order to make things easier to present).

Like I said, these two combined are what gets closest to being directly related to the video's "quality".
Why they matter is probably obvious once one mentions them as a variable:
Of course different encoding programs can encode a video in different ways, and different settings will make them do it differently.
But the real lesson to learn here is that these are even parameters in the first place!
This is something that even semi-experienced users sometimes miss (for example, I did so when I was starting out!):
It's easy to think that `ffmpeg -i myvideo.mp4 myencodedvideo.mp4` is the only way to reencode a video (maybe sprinkle in `-preset slow` if you're feeling like an expert), without realizing that this will use a fixed (low) quality setting that could be adjusted with further settings.

In reality, a video coding format like H.264 *only* specifies how an encoded H.264 stream should be *decoded* to pixel data again.
The H.264 specification does not specify in *any* way how a video should be "encoded" - only the format that the resulting file should use.
What makes formats like H.264 so efficient is that they provide very many different options to specify different kinds of redundant data with very few bits,
like "Fill this entire 16x16 block with white" or "copy this 16x16 block from the previous frame and adjust it a little."
But finding these redundancies, and figuring out how to specify some frame with as few bits as possible in this format,
while losing as little quality as possible, is up to the encoding program.

So, I really cannot stress enough that the encoding *settings* (including the tool used) matter the most when it comes to a video's quality.
This mainly manifests itself in two ways:

- 
The tool used. When it comes to encoding H.264 or H.265, the best encoders without any competition 2are x264 and x265. When you are in any situation where you can afford it, you should be using one of these encoders. Most video editing programs allow you to select them (and programs like ffmpeg or Handbrake (though ideally you shouldn't use the latter) use them internally).Most importantly, hardware encoders like NVENC aren't useful when targeting quality and efficiency. They aren't as sophisticated as x264/5 and are geared more towards low latency and high throughput. Again, this is very important to realize, and it's the main reason why I am stressing this so much. Hardware encoding certainly has its place in scenarios like streaming where latency matters much more than efficiency or quality, but when your goal is to output a high-quality encode, you shouldn't ever use it. 
- 
The quality setting. In x264/x265, the main knob to fiddle with to control quality is the setting called CRF (short for Constant Rate Factor). Lower CRF means higher quality (i.e. less quality loss when encoding) at the cost of higher file size. 

My main point here is not really how to use the CRF setting, but mainly that it exists in the first place,
and that it above everything else controls the output quality of your video.3

There are lots of other settings in x264/x265 that experts can use to precisely tweak their encodes,
but if you don't know what you're doing I'd recommend not touching them at all.
Once again, my main point here is really just that **encoding settings affect output quality**.

Now, I said above that these parameters are what gets closest to the video quality, but only in specific scenarios. Why is this? Well, what I mean by this is all that the encoding settings can affect is how closely the encoded video resembles the input video, i.e. how much quality is lost at the encoding step. If your input video is already bad, then reencoding it with perfect settings will not fix it. This may seem obvious, but it highlights how video quality has multiple different facets. Say you are choosing what footage to use as a base for your encode or edit, and have the choice between two sources, where one has a much higher bitrate than the other. Usually, you would choose the source with the higher bitrate, but this only makes sense if the two sources were encoded from the same underlying source (or at least similar ones)! It's very possible that the higher-bitrate source had some other destructive processing applied to it (say, sharpening, a bad upscale, lowpassing, etc. - more on these later). In cases like these, you may want to choose the lower-bitrate source instead, if it's at least encoded from a clean base.

So, as a summary, the *quality loss* of an encode is controlled by the encoding tool and settings,
but the quality of an *existing video* is affected by every single step
that happened between it being first recorded or rendered and it arriving on your hard drive.

I've now spent a long time talking about what quality *isn't*,
as well as what quality is *affected* by, so it might be time to try to formulate an actual definition of quality.

We already got pretty close with our discussion of encoding some video from a given source,
with the goal of getting an output that differs from the input as little as possible.
*That* is what quality is:
The quality of an encoded and processed video is a measure of how closely it resembles the source it was created from.

Again, this sounds extremely obvious once you spell it out, but it has huge consequences that may not be clear to everyone!
Most importantly, quality can only ever be measured relative to some *reference*,
some kind of ground truth.
Without a ground truth, everything becomes subjective.

Secondly, this now says something about the "quality" of videos you may come across in the wild (i.e. ones that weren't encoded by you): When you have two or more possible sources for the same footage (say, a movie or a show) available, and want to evaluate their quality, what matters is which of them is closer to the original footage they were both created from. In the case of a movie, this would be the original master. Once again, this may sound obvious, but we will see soon how many misconceptions are formed from not understanding this principle.

Finally, I need to talk about the word "closer" in this new definition, which is actually doing a *lot* of heavy lifting.
What "closer" really means here is very complicated, which is why I left it somewhat vague on purpose.
There are lots of ways to compare how close two videos are
(and that's assuming that they have the same resolution, frame rate, colors, etc),
but none of them are perfect.
In particular, there are many automated "objective" metrics (you may have heard of PSNR, SSIM, VMAF, etc.).
These are very important for encoding programs to function at all,
but it's important to realize that no automated metric is perfect,
and they all have their own strengths and weaknesses.

Because of this, video "quality" will always entail some degree of subjectivity. Still, there are some thing that are almost certainly wrong, and you'll see some of them in the following sections.

You should now have a decent idea of what quality actually means, and what it's determined by. Still, I want to spell out explicitly why various other parameters do not directly correlate to quality, and clear up associated misconceptions. So, let's go through them one by one.

This should hopefully be clear from the section on encoding settings.
Yes, more bits usually means better encode quality *if* everything else stays the same,
but ultimately the full package of encoding settings (which bitrate can be *one* of) is what matters.
Different encoders or settings will result in different efficiency levels,
so you can have two encodes of the same quality and different file sizes or vice-versa.
For example, NVENC allows very fast encoding at the expense of larger file sizes,
so an x264 encode (with decent settings) will get you much smaller files of the same quality
(but will of course take much longer).

Again, hopefully this should mostly be clear now:
What matters is the *tool* used to encode the video (and its settings), not the format it encodes to.
A more advanced format will allow for more techniques to efficiently encode a video,
but that only matters if the encoding program properly makes use of them.

In particular, there is an often quoted factoid that "HEVC is 50% more efficient than AVC", which in reality is just plain wrong. H.265 (that is, current standard H.265 encoders) does usually provide an efficiency gain over H.264 (that is, current standard H.264 encoders), but if it does it's by far less than 50%. And, as always, the format is just one facet of the full "Encoder and settings used" package. On pirate sites I sometimes see comments like "I want to download this, but it's AVC. Is there an HEVC version somewhere?", and I hope that I don't have to explain anything further about why that makes no sense.

Another important point is that the strengths and weaknesses of encoding tooling can greatly differ based on the level of quality you're targeting.
AV1 is the current new and fancy coding format,
and modern AV1 encoders (when used correctly) can yield incredible efficiency gains over x264/5 *on low-fidelity encodes*.
However, for high-quality encodes (i.e. targeting visual transparency),
x264 and x265 are still far ahead.
It's for reasons like these that it's very hard to make blanket statements on the efficiencies of different encoders.

One final thing to mention here is that the coding format *will* affect how difficult it is to *decode* your video.
Older or smaller devices may struggle to decode more advanced formats like AV1 or even H.265 (or specific profiles of formats like 10bit H.264).
This doesn't directly affect quality, but it may be important to mention for people that plan on making their own encodes:
If you're targeting high player compatibility, you may need to keep this in mind and (for example) release an 8bit H.264 version alongside your main release.

Hopefully I don't have to say anything more here.
Read the first section again if this is not yet clear to you.
But I *have* seen "This is an mp4 file, can someone upload an mkv file instead?" more than once,
which is why I need to spell this out here.
If this was you, look through the later sections to see how to fix these things for yourself.

This may be the biggest misconception of them all:
Many people effectively think that resolution is the *only* thing that controls a video's quality.
Maybe it's because of how YouTube and many other streaming platforms expose resolution as the only setting to change "quality".
Either way, this is not the case.
We've already seen why this is true in general, but let's go over some specific cases:

- 
Often, people downscale videos to some lower resolution in order to save file size. For example, if they have some 1080p video that, when run through their encoder, results in a 1GB file while they'd like their file to only be 500MB, they'd try to render it to a 720p video instead. But, as we've seen by now, this is usually not the right way to go about it. If your main goal is to bring down the file size, this can be done much better by adjusting the encoding settings instead: Different parts and scenes of a video will be easier or harder to compress. Scenes without a lot of motion or flat scenes without lots of details or grain will be easier to encode than scenes with lots of moving elements. Encoders know this and are able to intelligently allocate bits where they are most needed, focusing on *visual*quality rather than a uniformly fixed level of precision.By leaving the quality reduction to the encoder instead of downscaling before encoding, the encoder can decide where to save bits, rather than being forced to lose detail *everywhere*. This will often result in a much better-looking result at the same file size.Additionally, encoding downscaled video isn't actually as efficient as one might think, at least not with modern encoding formats: Since all the elements in the video get squished down, there'll be more small details in the same region of space, which makes them harder to encode. Now, if you're targeting *extremely*small file sizes, so that achieving these at (say) 1080p with very low bitrates is impossible without extremely visible artifacts, then you could consider reducing the resolution to make the artifacts more uniform. But resolution definitely shouldn't be the*first*knob you reach for to adjust file size: That should be the CRF or bitrate.
- 
Sometimes, some geniuses decide to use that new and fancy AI upscaling software they saw marketed somewhere to "improve" some video and upscale it to some higher resolution like 4k, and probably add a bunch of sharpening and whatnot in the process. I could write an entire article about AI upscaling alone (in fact, I have), but to keep things short: We've established that *quality*measures how close some video is to the source it originally came from. Applying*any*kind of post-processing4to the video can only ever take it further away from the source, not closer, and upscaling (AI or not) is no exception. Any kind of "detail" you may see the upscale add can only be invented, not added: The upscale fundamentally only has its input to work with, any extra data has to be pulled out of thin air. And no, I don't want to hear that*your*AI upscaling model is actually really good and better than the other ones, so it's actually okay to upscale with it. This is not a question of how good the upscaling process is, it's the process of upscaling itself that's already inherently lossy.There are some extra nuances here (read the linked post for some of them) and AI is not *inherently*bad, but please just trust me as a more experienced person when I tell you that**you should not upscale videos just for the sake of upscaling them**.

- 
These lessons about resolution also matter when it comes to *choosing*sources. Once again,*quality*refers to how close a video is to its original source, and it's very much possible for that original source to itself have a low resolution.This is especially relevant for digital anime, which is often produced at some resolution below 1080p. Even in 2025, production resolutions like 720p or 1500x844 are still very common, with the 1080p release being upscaled (usually using conventional methods, not AI) from that. Usually this is not too important to the end user, but it does mean that if you see a new fancy 4k release of some digitally animated show being advertised, the chances are extremely high that this is not *truly*4k, and instead just upscaled from whatever 1080p master they had before. Note, though, that anime movies or shows that were originally animated on film can be a different story,Similarly, this is very relevant for digitally animated shows that were originally released on DVD. For a good portion of shows from that era, there exist 1080p Blu-Rays that are extremely badly upscaled, so that the DVD will be a much better source. (However, DVDs bring a *ton*of other complications with them, so in the end you should pray that someone else has already done the work of making a proper encode for you.) There are also plenty of shows where this isn't the case, especially if the Blu-ray is a better rescan of a film or if the show has a LaserDisc release, but the general takeaway is that "higher resolution does not automatically mean better" also extends to official releases.

So, as a summary, keep in mind that resolution is not the same as quality. A higher resolution may not mean better quality, and lowering the resolution may not be the best way to save file size.

This is fairly similar to the resolution story, so there's not much more to say here.
Just like AI upscaling just for the sake of upscaling, frame interpolation is bad.
There's not even any nuance here this time, just don't do it. (Do I need to spell out what "quality" means again?)
Movies and TV shows are usually 24fps (well, often they're actually 23.976fps5, but you get the idea),
so if you find a source somewhere that has some different frame rate, double-check if that is the correct one.

This is a tricky one, and I am mainly mentioning it to talk about a very specific technique in encoding.

Bit depth is a slightly more niche concept, so I'll explain it just in case:
Bit depth refers to how many color values are possible for each pixel.
Almost all images and videos you'll come across are 8bit.
For RGB colors, this would mean 256 red/green/blue color values per pixel,
which results in `256 * 256 * 256 = 16777216` total possible RGB color values.
In reality, video colors are not *actually* stored in RGB,
and usually do not exhaust their full available range of values,
but for getting a basic intuition this is not too important.

However, it's also possible for videos to have a higher bit depth like 10bit or 12bit. Apart from masters, this is common for HDR video.

In principle, the same rules as for resolution and frame rate apply:
Don't change any aspects of your video without a good reason,
so don't change the bit depth either if you can avoid it.
That said, it *is* common in video encoding to actually encode footage at a bit depth higher than the source's.
This is due to intricacies of video encoding that are too complicated to explain here,
but the upshot is that encoding at a higher bit depth can actually result in an *increase* in efficiency.
This is why you may see 10bit encodes of 8bit footage:
These do not mean that there was a 10bit source somewhere,
they're just encoded in this way because it was more efficient.

This doesn't contradict our philosophy of not changing anything without good reason,
it just means that there is a "good reason" in this case.
In particular, this is feasible here because, unlike with resolution or frame rate, increasing bit depth is not a destructive process (when done correctly)6.

(If you're interested in *why* encoding at a higher bit depth is more efficient, here's an attempt at a basic explanation:
Intuitively, you might be confused about this, since adding more bits ought to correspond to more bits to store,
which results in more required file size.
But the important thing to realize is that the "bit depth" in modern video coding formats is not actually
what controls the level of precision with which pixel values (or, in reality, DCT coefficients) are stored.
That level of precision is controlled by the quantization level, which is a different parameter.
(And that is in fact the main knob that encoders turn to regulate bit rate and quality.)
Instead, the actual bit depth controls the level of precision at which all mathematical operations
(like motion prediction and DCTs) are performed when decoding the video, as well as the allowable scale for the quantization level.
Encoding at a higher bit depth means that operations are performed with more precision,
which makes certain encoding techniques more precise and hence more efficient, which in turn saves space.
However, raising the bit depth also means that slightly more bits need to be spent to encode the actual quantization factor (and other elements),
so at some point you do get diminishing returns.
Empirically it turns out that encoding at 10bit works pretty well for 8bit content, but that encoding at 12bit is not worth it.)

This is another slightly tricky one.
*Usually*, a Blu-ray release of some footage will be better than a web version from the same source,
on account of having a much higher bit rate.
However, this doesn't *always* need to be the case:
The fact that various post-processing operations can affect the quality of the video also applies to the authoring stage
(that is, the process of taking a show or movie's master, and putting it onto a Blu-ray,
performing all the necessary conversion and compression that this entails),
and it is very much possible for a Blu-ray release to have some destructive filtering applied to it that the web releases do not (or for the Blu-ray release to just have terrible encoding settings).
Different web streams from different sites, or different Blu-rays from different authoring companies can be different too.

Again, this is especially relevant in anime, where some Blu-ray authoring companies apply a blur to the video before encoding it, which hurts quality7.

If you're just starting out in working with video, it may be hard to judge for yourself which source is better, but the main thing I want to convey here is that "Blu-ray" does not automatically have to mean "better quality". Always try to manually evaluate sources using your eyes, or ask someone more experienced for advice on which source to pick (see below for some resources on this).

HDR (High Dynamic Range) is another complicated topic.
What I mainly want to convey here is that, once again, HDR does not *automatically* mean "better than SDR".
If there are HDR and SDR sources of some footage available,
it all depends on how they were created, and from what kind of common source (if there is one).
It's possible for the SDR version to be a direct tonemap of the HDR one (in which case the HDR version is the objectively better source) or for the HDR version to have been inverse tonemapped from the SDR one (in which case it's the other way around), or for them to have both been created from some base source (in which case it depends on *how*).
For example, it is not uncommon for official HDR releases of some footage to never actually reach a brightness above 100 nits,
and hence be no better than the SDR version.

In particular, you should be very suspicious of any HDR (or Dolby Vision) source you may find for a video that wasn't officially released in HDR anywhere. It's very much possible that this "HDR" version was created artificially from the SDR version by whoever released it, in which case (just like an AI upscale) there's no reason to use it over the base SDR version.

Again, HDR is a very complex topic and these things can be very hard to evaluate as a newcomer, but the important thing is to know that this subtlety exists in the first place. If the SDR version looks decent, you may just want to save yourself (and your viewers, if there are any) the trouble of dealing with HDR and work with the SDR version.

As I have already repeated ad nauseam,
the goal of video encoding is to change the source as little as possible.
Just like you shouldn't change the resolution or frame rate without a good reason,
the same applies to colors.
I sometimes see releases where people "improved the colors :)",
and it turns out that what they really did was fiddle with the brightness and saturation sliders until it looked "better"
(read: brighter and more vibrant).8
But doing this is the opposite of staying true to the source.
Color grading is very important for editing photos or raw footage,
but when you're working with footage that was *already* edited and mastered by the artists,
any further "color corrections" go against the artistic intent.

In short, remember that "brighter and more saturated" does not mean "better".

Finally, while we're on the topic of colors:
When you run an encode, especially from some kind of video editing software,
make sure to make a direct comparison of some output frames to the corresponding input frames
*using good viewing software* (i.e. mpv or vs-preview, see below).
If you see a noticeable color mismatch, this may be due to some misconfiguration in your editing software or project
(like the color matrix or color range)
that you will need to look into.
Read the section on Color Space Parameters for more information on this.

Last but definitely not least, we have another one of the bigger misconceptions. Many people think that "sharp" means "higher quality" and, in particular, that "blurry" means "lower quality". While it's true that a lower quality encode can manifest itself in more noise around lines, and that reducing the resolution (which we've already established you probably shouldn't do) will automatically mean that lines can no longer be as sharp, this is far from a one-to-one correspondence.

In reality, the exact same thing as for resolutions, frame rates, or colors applies.
You want to stay as close to your original video as possible.
If some elements of the original video are comparatively blurry, chances are that they're *meant* to be blurry.
(Or, at the very least, any kind of sharpening process will not be able to distinguish
between elements that are meant to be blurry and ones that aren't.)

Hence, just like you shouldn't fiddle with color sliders just to "improve the colors",
you shouldn't slap a sharpening filter on top of your video just to "make it sharper :)".
This will only take your video further away from the source, not closer.9

It's true that to the layman viewer's eye, sharper content will look more appealing.
But once you know what to look for, you will see that sharpening creates a lot of ugly artifacts like
line warping or haloing.
Like with upscaling, please just take my word for it when I tell you that **prioritizing sharpness above all else is not a good idea**.

Now, that was a lot of text, but unfortunately it was needed.
Video is very, *very* complicated, and this was just the tip of the tip of the iceberg.
In case that was too much information to dump on you all at once, let me summarize the most important takeaways:

- You cannot judge a video's quality just by looking at its resolution and file size.
- If in any way possible, use x264 or x265 to encode your video. Use the CRF setting to adjust quality vs. file size instead of jumping directly to downscaling.
- You should not change any aspect of your video unless you know exactly what you're doing (and the target audience of this post does not). This affects resolution, frame rate, colors, sharpness, and any other postprocessing filters you might think of applying.

As a novice video encoder, it may be hard to see quality loss in the beginning. You may come across images or comparisons where some experienced encoder says "Oh my god this looks terrible!!" while you're thinking "Are those the same picture?".

But don't worry, this is normal. You have to know what to look for in an image, and you have to train your eyes to look for it. (But know that this is cursed knowledge. Once you learn how to spot artifacts, you can never look at video the same again.) A full guide on how to spot video artifacts would take up an entire second article with many example images, but as a short summary, here is a list of areas you should focus on most:

- Dark areas, especially dark gradients
- Strong colors, in particular black edges on deep and dark reds
- Areas with lots of (static or dynamic) grain or texture
- The spaces *around*sharp lines and edges. Don't look at the edges themselves, instead look for noise*next*to them. In particular, look for bright "halos" around edges (also called ringing)
- In particular, look for noise next to sharp full-resolution elements like on-screen text
- Image borders

Keep in mind that what constitutes acceptable quality loss is always in the eye of the beholder,
and that that is a two-way street.
If you are creating encodes mainly for yourself, and you yourself cannot see any quality loss, then there's no reason to worry about it even if someone else tells you it's visible.
However, on the other hand, you also shouldn't criticize anyone for releasing high file size encodes to prevent quality loss just because *you* can't see the artifacts they would prevent.

This section is a bit more advanced than the other sections. You can skip it on your first time reading, if you prefer, or only read the summary at the end of the section.

There are some things that can go wrong when modifying a video that you should be aware of. They shouldn't happen with the workflows described below, but if you use a different workflow and/or add extra steps, you may run into these.

I have mentioned above how videos are (usually) not actually stored as RGB, and this is where this becomes relevant. The colors of video frames are usually stored in a different color space called YCbCr. Here, the "Y" part is called "luma" and more or less represents a pixel's "brightness", while the "Cb" and "Cr" parts are called "chroma" and represent the color tone. This goes all the way back to how analog color television had to be built in a way that is backwards-compatible with black-and-white television, but we're stuck with it now.

Moreover, the chroma part of the video is often stored at half the resolution (in both directions) of the luma part.
This is called "chroma subsampling".
For example, a typical 1920x1080 video would be stored as `1920*1080` luma values and two sets of `960*540` chroma values.
When playing the decoded video, your media player first needs to upscale the two 960x540 chroma "planes" to 1920x1080.
Once again, this mostly has historical reasons nowadays.

Check this comparison for an example of how an RGB image splits into Y/Cb/Cr parts.

Now, why do you need to know this?
Well, the problem is that there is *more than one way* to convert an RGB10 image to YCbCr and back.
The most relevant of these two are called BT.601 and BT.709.
Both of these specify a "matrix" (if you don't know linear algebra, just think of it as a "formula") that can be used to compute a YCbCr value from an RGB value and vice-versa.
However, converting a given YCbCr value to RGB via BT.601 will give a different RGB color than BT.709!

Out of these two color matrices, BT.601 is the older one.
It was used for old SD-era content like DVDs.
BT.709, on the other hand, is the newer matrix.
The vast majority of videos you will run into (say: movies and TV shows that were produced in 720p or higher, screen captures, most YouTube videos11, etc.) should be BT.709.
One notable exception here is HDR video, but dealing with that properly is an entirely different beast of its own and is outside the scope of this post.

However, some old software (including ffmpeg!) may still default to using BT.601 in certain cases even when BT.709 should be used instead. This is why it is important to know about this distinction.

Now, a media player playing back a video file needs to *know* which color matrix (BT.601 or BT.709) a video uses.
For a "well-behaved" video, this is specified in the video's metadata, but unfortunately many videos *aren't* this well-behaved.
(And many video editing or encoding programs do not always store this information in the metadata of the videos they output.)

If the media player does not know what color matrix a video file uses, it has to guess based on the information it has, like e.g. the video's resolution. For example, some video players may guess that an untagged (i.e. with no specified matrix) video uses BT.601 if its height is less than, say, 600 (suggesting that this is an SD era video).

This can already cause some suprising behavior! Let's say you have an untagged 1080p video, which you then downscale to 480p (which I have explained above may be a bad idea, but that's beside the point here). If you open the 1080p video and the 480p video side by side in such a media player, the two videos will be displayed with different colors!

If you were seeing this without knowing what's going on, you might be led to conclude that the video encoder is somehow changing the colors, but the reality is that the change in resolution is causing the media player to guess a different color matrix. Hence, you can solve this by tagging both your input and your output video as BT.709. (If you tag the input video as BT.709, most encoders should should also copy this to your output video.)

For reasons like these, this topic can cause great headaches, but the upshot is:
**If you see your reencode somehow changing colors, check your input and output video's color matrices.**
If your source video is untagged, it can be hard to figure out what color matrix it *should* have,
but when in doubt you can at least make sure that your output has the same matrix as your input
(or, if your input is untagged, the same matrix as what a player like mpv would guess).
Or, of course, you can ask someone more experienced for advice.

You can check a file's color matrix in MediaInfo (see the section below for the tools mentioned here),
and you can edit it in MKVToolNix.
To see what color matrix mpv assumes on your video, open your video in mpv, press `i`, and check the `Colormatrix: ` field under the "Video" section (*not* the "Display" section).

Unfortunately, the story does not end with the color matrix. The matrix is just one of multiple parameters that specify how to interpret a video's colors. The full list of parameters is:

- The color matrix (BT.709 / BT.601 / BT.2020 / etc.). Note that BT.601 is sometimes also called SMPTE170M or BT.470BG.
- The color range (Limited or Full)
- The transfer characteristics, or gamma (BT.709 / BT.601 / BT.1886 / sRGB / PQ / HLG / etc.)12
- The primaries (BT.709 / BT.601 / BT.2020 / etc.)
- The chroma location (Left or Center Left / Center / Top Left / etc.)

Yes, codes like "BT.709" can refer to *multiple* of these parameters.
But this doesn't cause much confusion in my experience, since most programs make a clear distinction between matrix, transfer, and primaries.
If you hear someone talk about "a BT.709 video" (without specifying whether they mean matrix, transfer, or primaries),
they probably mean the matrix.
(A video with a BT.709 matrix does not need to automatically have BT.709 transfer and primaries (it could have sRGB instead, for example), but if you see a video tagged with a mix of BT.709 and BT.601 that usually means that something went wrong *somewhere*.)

All of these five parameters are values that are necessary to properly interpret a video's colors,
and that *should* be provided as metadata but often aren't.
Luckily, if you're editing a video, you generally only need to worry about the matrix, range, and chroma location - the other two (transfer and primaries) are less likely to break.

We have already talked about the matrix, so let's continue with the range.
The vast majority of videos you're dealing with will have a "Limited" range.
The color range should also not break when reencoding video with, say, ffmpeg, but I *have* seen issues with color range when working with video editors like Vegas.
When you see your output colors being much darker or much brighter than your input, check the color range settings of your editing program.

Finally, there is the chroma location. This is the most subtle of all of these, since it's harder to notice issues with it, but it may also be the one with the highest risk of breaking - especially when working with video editing programs.

When applying chroma subsampling (which, as you hopefully remember, is the process of scaling down the chroma by a factor of two in each direction), there are multiple ways to perform the downscale process. Ultimately, what you need to do is turn each 2x2 group of four chroma values into a single value. One way to do this would be to simply take the top left pixel of each 2x2 group. Alternatively, one could average all four pixel values together and obtain a new value that could be interpreted as the interpolated "middle" value of the 2x2 square. These two methods result in different alignments of the chroma sample grid relative to the video's pixel grid: A chroma value could either "come from" the top left of a 2x2 square of luma values, or from the "middle" (or from some other location). This is called the "chroma location", and it is yet another parameter that media players need to know to play a file back properly: When scaling the chroma back to the luma's full resolution, the scaling process needs to factor in the chroma location in order to not introduce a shift relative to the original chroma.

For reasons that I have not yet been able to uncover, the standard chroma location for most videos is actually "center left", i.e. in the middle between the left two samples of the four luma samples. Hence, video players (or at least good players like mpv) will default to "center left" chroma when a video's chroma location is not tagged. However, a lot of video processing software does not handle chroma location correctly, e.g. often using a chroma location of "center" instead. Because of this, depending on your workflow, editing a video may sometimes introduce a "chroma shift" where the chroma ends up shifted by some small amount relative to the luma. This can be hard to spot to a novice video encoder, but it manifests in colored lines consistently having a slightly different color on one side of the line than on the other side. As before, chroma location issues can usually be fixed by tagging your videos and/or configuring your tools correctly (and by asking a more experienced person if necessary).

**In summary**: Depending on your workflow, processing your video may have a risk of breaking your video's color matrix, color range, or chroma location. (And, moreover, it is possible for your sources to have broken or mistagged matrices, ranges, or chroma locations. But if you are starting out it may be better not to worry about that part, and to just ensure that you aren't introducing any *additional* problems involving these parameters.) These issues can manifest as follows:

- Wrong (usually mistagged or untagged) color matrices: Generally manifests in colors slightly changing in hue and intensity. See this comparison for an example. This can usually be fixed by tagging your input correctly.
- Wrong color range: This will manifest in colors becoming much less saturated or more saturated throughout. See this comparison for an example. Color range issues should be fairly rare for normal reencodes, but can happen when using a misconfigured video editor.
- Chroma shift:
See this comparison for an example.
This one can be hard to spot for beginners.
The easiest way to spot a shift with a given reference is to (zoom in a lot and) look at areas with very strong colors.
Spotting a chroma shift without a reference is even harder,
but in this comparison you can see that diagonal lines on the "chroma shift" image have a slight reddish glow on the right side and a blue-ish glow on the left side.
When this glow is consistent throughout the entire video, that may be indicative of a chroma shift.
(Make sure to not confuse an intentionally added chromatic abberation effect for a chroma shift, though.13) How to fix a chroma shift can depend a lot based on your workflow (so figuring out where it is introduced should be your first step), but tagging your input video and checking your editing software's configuration will definitely help. If you cannot figure out how to configure your software to avoid a chroma shift, you can try retagging your output.

This section was far more technical than the rest, but the good news is that you do not need to understand all of it right away. The important parts to remember are that processing a video can cause some issues related to color spaces. When trying out a new workflow, it may be a good idea to compare an output frame to the corresponding input frame and look for any color mismatches that look like the ones in the examples above. If you don't see any, you can forget everything you read in this section. If you do find something, this section may help you to fix it.

When you're working on an anime or some other media that is not in your target audience's language, you will need to add subtitles, in which case there are a couple of things you should know.

The most powerful format for subtitles is Advanced SubStation Alpha, or ASS for short14.
ASS subtitles not only allow showing subtitles for spoken dialogue
but also creating translations for on-screen text that blend in seamlessly with the original video.
Even if you do not plan to make subtitles like these themselves,
you probably want to ship subtitles you downloaded from somewhere, which will probably be in the ASS format.

One important thing to know is that the only container format that really supports ASS subtitles is mkv. If, for some reason (probably because you're targeting some kind of streaming), you do not want to release an mkv file in the end, you will need to hardsub. See below for the best way to do this.

Secondly, if your goal is to edit your video, you will have to think about how to match your subtitles to your edit. There is no good automated solution here. Your options are basically:

- Manually retime the subtitles in a program like Aegisub, or
- Hardsub the subtitles and edit the hardsubbed video.

In general, you should avoid hardsubbing when possible, since it

- involves reencoding, and hence introduces quality loss,
- takes time (which may not be a problem when you are only editing your video once, but becomes increasingly annoying if you want to make incremental fixes later on),
- makes it much harder for anyone, including yourself, to change some aspect of the subtitles later on.

However, retiming all subtitles yourself for a quick edit is also a lot of effort. In the end, the choice is yours. If you do end up hardsubbing, make sure you do it correctly. Read the later sections for how.

I've now talked a lot about what you *shouldn't* do, so what should you do instead?
This section contains some useful tools, as well as workflows to do certain things the right way.

- 
MediaInfo: If you install one tool from this list, you should install this one, which is why it's listed at the top. Step 0 in anything to do with video is finding out what exactly you are working with, and MediaInfo will tell you exactly that. Open a file in MediaInfo and switch the view to "Text" at the top to see all important data. If you ever need help from someone more experienced with video, sending them a proper MediaInfo dump of your file is a great way to get them into a good mood. 
- 
mpv is the single best media player out there, and ideally you should use it. MPC-HC (if you get the actual latest version) is alright too, but mpv is the definite best. In particular, VLC is not recommended. Apart from simply watching the video, mpv can also make screenshots and encode videos for you. The latter in particular is very helpful for hardsubbing. 
- 
ffmpeg is your Swiss army knife for everything to do with video, from inspecting to encoding and remuxing. (Though you should know that it's usually not actually ffmpeg itself that's doing the encoding. When you encode to H.264/H.265 with ffmpeg, ffmpeg is actually calling x264/x265 internally. I'm mainly bringing this up because it bothers me how everyone praises ffmpeg for being good at encoding when it's really x264/5, but this also means that you should check the x264/5 documentation if you need help with encoding, not ffmpeg's.) Note that ffmpeg is kind of a jack of all trades, master of none. It can do a lot of things fairly well, but for specific tasks there are often specialized tools available that do it even better. FFmpeg is a *command-line tool*. If you have never used a command-line tool before, read this page for a quick primer.Before you start complaining about how complicated ffmpeg is and how arcane its syntax is, do yourself a favor and read the start of its documentation. It turns out that reading the (f.) manual actually helps a lot! 
- 
Use SlowPics if you want to share image comparisons. There are also ways to automatically upload comparisons to SlowPics using vs-preview, but those are a bit more involved. When looking at a SlowPics comparison, uncheck "Show border" and "Smooth scaling" at the bottom and use the clicker comparison rather than a slider. Use the number keys (1/2/3/etc) to switch between images. 
- 
MKVToolNix for muxing mkv files. You can do this with ffmpeg too, but MKVToolNix has a GUI if you need one (and is better in certain ways). 
- 
MKVExtractGUI or MKVcleaver to extract tracks from mkv files (or learn how to do it with ffmpeg). 
- 
Aegisub to edit subtitles. Note that you can also simply open `.srt`or`.ass`subtitles in a text editor like Notepad if you need to quickly check something, but if you want to do actual editing or timing you should use a proper tool like Aegisub.
- 
MkvToMp4 for remuxing an `.mkv`file to an`.mp4`with a proper constant frame rate (see below).

- Avoid using Handbrake if possible. Handbrake has a lot of footguns like suddenly changing the frame rate or adding interlacing. I would recommend you to just learn basic ffmpeg usage instead. If you must use a GUI, try Staxrip.
- Don't use any file conversion websites. Those all just use ffmpeg under the hood anyway, so you'd be better off just spending the 10 minutes to learn how to use ffmpeg directly. Hopefully I don't need to tell you that you shouldn't fall for any 30$ x264/ffmpeg wrappers either.
- Don't use Topaz AI, Anime4k, RealESRGAN, RIFE, etc. Trust me, just don't.
- Don't use imgsli for image comparisons, it (lossily) converts its images to JPEG which invalidates any comparisons. Use SlowPics (linked above), and try not to use the slider feature there either. You can spot a lot more differences by switching the full images back and forth than with a slider.

Finally, let me explain a few things you *should* do.

If you've read the previous sections, you'll know that reencoding a video will hurt its quality (and reencoding it over and over will hurt its quality even more, since later encodes will spend bits to reproduce the artifacts introduced in the previous encodes). Hence, you should make sure that you only reencode when absolutely necessary, and do all other necessary conversions through remuxing. Ideally, that would mean (lossily) reencoding only once, at the very end of your workflow. If your editing software does not allow encoding using x264/x265, you can export a lossless render from it and then encode that lossless render with ffmpeg.

Sometimes, you cannot easily avoid reencoding an additional time at some other step in your workflow.
If this happens, at least make sure that your intermediary encodes are either lossless, or as close to lossless as possible.
Unfortunately, I have not yet found a reliable way to encode a lossless file that common editing programs can open
(if you know one, let me know!),
but at the very least you can make an x264 encode with `-crf 1`.

To make an *actually* lossless encode with `x264` you can add `-qp 0` instead of a `-crf` argument,
but be aware that not all programs will be able to open such a file.

The simplest way to encode a video is using ffmpeg. More advanced users will encode using x264 or x265 directly, but ffmpeg is fine for beginners.

A basic template command to reencode a video is simply15

```
ffmpeg -i yourinput.mkv -c copy -c:v x264 -preset slower -crf 20 youroutput.mkv
```
As explained in the first section, adjust the CRF to control the quality at the expense of file size.
If you're encoding anime or animation, you may want to bump up the bframes by adding `-x264-params bframes=8`
(which will save a bit of file size but take longer to encode).
Other than that, **do not touch any other settings you do not understand**.
In particular, do not use `-tune animation` for anime; that tune is targeted towards extremely flat animation,
so it will be counterproductive on anime, which usually has a fair amount of grain and texture.

A good way to think of video encoding is as a three-way tradeoff between file size, quality, and encoding speed.
You can decrease the file size, but only at the expense of quality or encoding speed,
and similarly for the other two factors.
The `crf` setting is used to regulate between quality and (decrease in) file size.
The `preset` setting controls the encoding speed, and hence the efficiency.
A faster preset will mean a faster encode, but also a larger and lower-quality one.

Be aware that the visual quality of a given CRF value will depend on the resolution you're encoding at. CRF 18 at 1080p behaves differently from CRF 18 at 480p. The best way to pick a CRF for your encode is just to run a few sample encodes and compare the results.

Hopefully you can figure this out with the tools linked above (MKVToolNix being the easiest way). All I really want to say here is that if you are muxing in ASS subtitles, you need to add all the fonts used in the subtitles as attachments. Aegisub has a font collector that can collect all the fonts used in a file. If you don't want to install the fonts, you can use a font manager like FontBase (add the folder with all fonts as a "watched folder") to temporarily activate them without installing them.

This is more tricky and the main reason why this section exists.
In principle, muxing an mp4 file is easy: Just run `ffmpeg -i yourinput.mkv -c copy youroutput.mp4`.
However, chances are that the reason you are remuxing to an mp4 file is so that you can import your video into your favorite video editing program.
In that case, remuxing using ffmpeg can cause some problems with the frame rate.

Most videos you'll come across have a constant fractional framerate of 24000/1001 (which is approximately 23.976) frames per second.
But this is actually a bit of a lie: A lot of times the frame rates aren't *truly* constant
(and, in fact, in mkv files they often *cannot* be).
For certain technical reasons, frame timestamps often need to be rounded,
which causes ever-so-slight deviations from the constant 24000/1001 frames per second.
Video players handle this completely fine, so that you'd never even notice it as a normal (or even experienced) user.
However, some video editing programs can be extremely picky about these frame rates,
and introduce stuttering when the frame rate is not *truly* constant.

Since mkv files fundamentally cannot have a true constant frame rate of 24000/1001,
remuxing to mp4 using ffmpeg will also result in a frame rate that is not truly constant.
You can see this in MediaInfo in the `Frame rate mode` entry.16

There exist a couple of ways to fix this:

- 
MkvToMp4 is a GUI application that can remux an mkv to an mp4 file and force a constant frame rate if applicable. While I haven't audited it in detail myself, I know video editors who have used it for a long time and haven't had issues with it. Note that it only supports H.264, not H.265, though. 
- 
With the right incantation, you can also force a constant frame rate in ffmpeg. The best one I could come up with needs two invocations, though: `ffmpeg -i yourinput.mkv -c copy -video_track_timescale 24000 intermediary.mp4 ffmpeg -i intermediary.mp4 -c copy -bsf:v "setts=dts=1001*round(DTS/1001):pts=1001*round(PTS/1001)" out.mp4`If your source video is, say, 30000/1001 fps instead of 24000/1001, replace the 24000 in the first call with the appropriate numerator. There also exists a tool called mp4fpsmod that can force mp4 frame rates, but I found the ffmpeg call to be more reliable when the first frame does not start at timestamp 0. 

Furthermore, you may also want to add `-movflags +faststart` to your command when muxing an mp4 file.
This will make the muxer put all the file's metadata at the start of the file rather than at the end,
so that even partial downloads of the file can be played back.
With the above two commands, this would look as follows:

```
ffmpeg -i yourinput.mkv -c copy -video_track_timescale 24000 intermediary.mp4
ffmpeg -i intermediary.mp4 -c copy -bsf:v "setts=dts=1001*round(DTS/1001):pts=1001*round(PTS/1001)" -movflags +faststart out.mp4
```
Use mpv to hardsub:

```
mpv --no-config yourinput.mkv -o youroutput.mkv --audio=no --ovc=libx264 --ovcopts=preset=slower,crf=20,bframes=8
```
Adjust the encoding settings accordingly, of course.

This will hardsub the track marked as the default, add e.g. `--sid=0` or `--slang=eng` to select a different track.
Hardsubbing is an extra encoding step, and like explained above you want to reencode as few times as possible.
Hence, either make sure that hardsubbing happens at the end of the workflow from a lossless source,
or output a (near) lossless encode when hardsubbing (e.g. by setting the CRF to 1).

This is a bonus section meant to prevent some slightly more advanced misconceptions. If you don't know what the term "interlacing" means, you can safely skip this section.

If you *do* know what interlacing means,
the main thing I want to get across here is that not all interlacing is the same,
and in particular that the answer to seeing footage that looks "interlaced" is not always to run a deinterlacer.

When working with movies and TV shows,
it is actually much more likely for interlaced-looking footage to really be *telecined*.17.
What this means exactly is outside the scope of this article,
but you can read fieldbased.media or the Wobbly guide for more information.
The important takeaway is that telecining can (almost) be losslessly reversed (though it may need manual processing),
and that running a deinterlacer on telecined footage will throw away half the vertical resolution while still keeping the frame rate stutters.
When you see footage that shows combing, please consult some more experienced person before blindly running a deinterlacer on it.

The above should cover everything you need to know as a beginner. If you like to suffer and are interested in learning more about multimedia and encoding, the JET Guide can be a good place to start. In particular, it contains a big list of resources that link other good guides. My blog also has some more posts about video stuff.

## Footnotes

- 
*Technically*the term*codec*refers to a specific*program*that can encode and decode a certain format, not the format itself, but almost nobody makes that distinction in practice. (That is, many people use "codec" to refer to the format itself, too. The distinction between a format and an encoding program*is*extremely important, as we'll see later.) ↩
- 
When targeting quality ↩ 
- 
You may be wondering why I am not mentioning bitrate, which is also a setting in x264/x265. This is because setting x264/x265 to some bitrate will make them *force*the video to that bitrate (when possible), even if it may not be necessary. This will make it waste bits on simple scenes that could be spent on more complex scenes instead. When you are not encoding for live streaming, CRF is the better setting to use, since it will automatically allocate the bits where they're needed most. ↩
- 
Unless you're *extremely*surgical with it and know*exactly*what you're doing, which the target audience to this post definitely doesn't ↩
- 
And *that*is actually 24000/1001 fps and*also*constant frame rates are usually a lie anyway but you get the idea. ↩
- 
Scaling or changing the frame rate can *also*be nondestructive when done correctly, but they're much easier to get wrong than the bit depth. ↩
- 
*Why*this happens is complicated (and we don't even fully know ourselves). The technical term is*lowpassing*, with the idea being to remove high frequencies in advance in order to improve compressibility, but in practice this is just counterproductive. We suspect that certain proprietary authoring software suites have this lowpassing enabled by default, and that authoring studios aren't aware of it or its negative consequences. ↩
- 
There *are*some actual types of errors in encoding that affect colors and can be objectively fixed, like double range compression or mistagged color matrices, but those are not the same thing as fiddling with some sliders, and they once again require you to know exactly what you're doing. ↩
- 
Once again, some caveats apply here in specific cases. For example, if you absolutely cannot avoid upscaling your video, you might as well find a "good" way (whatever that means) to upscale it, and try to add as little blurring as possible. But sharpening just for the sake of sharpening is not a good idea. ↩ 
- 
In case that you have heard about how there are also multiple "RGB" color spaces before (if you have not, just ignore this footnote), I should clarify that I do not mean any specific RGB color space here (so in particular not sRGB). For the purposes of this section it's enough to interpret the term "RGB colors" as just "a set of three color values that each specify *some*sort of R/G/B component", without worrying about, say, primaries or transfer characteristics. ↩
- 
You can actually see the color matrix a YouTube video uses by right-clicking the video and clicking "Stats for Nerds". ↩ 
- 
There is a distinction between OETF and EOTF to be made here (as well as subtleties like an OETF of BT.709 often being interpreted as an EOTF of BT.1886, as well as sRGB sometimes being displayed as 2.2 gamma or BT.1886 instead or vice-versa, and so on... God, gamma is a mess), but all of that is out of scope of this article. I'm only listing all of these so people can recognize the abbreviations when they see them in the wild. ↩ 
- 
If you're wondering how to distinguish between these (apart from asking someone more experienced): A digital chromatic abberation effect is usually created by shifting the RGB planes relative to each other, while a chroma shift is a shift of, well, the chroma plane relative to the luma plane. Chromatic abberation will result in differently colored glow around edges and probably cannot be fixed by retagging the chroma location. ↩ 
- 
Yeah, the jokes never get old. ↩ 
- 
This is a starting point for the target audience of this article. Experienced encoders targeting transparency will use very different settings. ↩ 
- 
Though I'm not fully sure if MediaInfo is completely reliable here. The best way to know for sure is to use MP4 Inspector and check the `moov > trak > mdia > minf > stbl > stts`box. If it is truly CFR, there should only be a single entry. ↩
- 
This is a fairly established term in the encoding community, but it's actually somewhat incorrect. Outside of the encoding community, you'll usually see this being referred to as *3:2 pulldown*instead. ↩

Okay, here's the "no-brainer" TL;DR for someone who just wants to avoid major screw-ups without reading all the details:

TL;DR for No-Brainers:MKV vs. MP4 are just "boxes."Don't use random online converters or simple "convert MKV to MP4" tools. They'll likely re-make (re-encode) your video and make it look worse. If your software needs MP4, try to "remux" (change the box without re-making the video).RE-ENCODING IS BAD (99% of the time for you).It loses quality and takes forever. Avoid it like the plague.DON'T "IMPROVE" VIDEOS:Don't upscale(e.g., 720p to 1080p/4K with AI). It's fake detail and looks bad.Don't change frame rates(e.g., 24fps to 60fps). It looks weird.Don't "fix" colors or sharpness.You'll probably make it worse. The original is usually how it's meant to be.BAD TOOLS = BAD RESULTS:verycareful with Handbrake; it can easily re-encode when you don't want it to.WHAT TO DO INSTEAD (if youmusttouch it):MediaInfo(free tool) to see what your video fileactuallyis.MkvToMp4or specific`ffmpeg`commands).haveto re-encode, learn basic`ffmpeg`commands to use`x264`or`x265`with a`CRF`setting (lower CRF = better quality, bigger file).Basically: If you don't knowexactlywhat you're doing, try not to change the video itself. Changing the "box" (container) is sometimes okay. Changing the video inside is usually bad.[Gemini 2.5 Pro]
