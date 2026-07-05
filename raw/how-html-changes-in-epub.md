---
url: https://www.htmhell.dev/adventcalendar/2025/11/
date_fetched: 2026-07-05
backfilled: true
---

# How HTML changes in ePub

by Robin Whittleton published on

ePub is the W3C standard for ebooks. It lets you take your knowledge of the web, and use it to produce little self-contained sets of documents that can be freely distributed as a single file ready for reading on extremely low-power devices, and they even reflow to fit any screen.

Yet while I said that you can use your knowledge of the web to build ePubs, the technology in use is twisted in unforeseen ways, and you might have to unlearn the things you think you knew. Prepare yourself…

## HTML, sort of

ePubs, at their core, use HTML, just like the websites we build every day. Except, well, there’s a big asterisk after that. Let’s dive into the differences.

A few decades ago XML emerged from the pit. XML – an extensible standard for expressing marked up data – could be used for documents, data transfer, and a bunch of other things, and people genuinely liked it (or, much like AI today, pretended to for job security). They liked it so much that a concerted effort was started to take HTML and rebuild it on top of XML. This project had a name you might have heard of: XHTML.

XHTML didn’t work out, for a number of reasons. The extensibility of XML turned out to not be useful when browsers didn’t support even common extensions. Then there was the problem of fragility: any syntax problems with your XHTML and your users would get a blank screen. If those two problems weren’t enough, XHTML was slower in practice because the browser needed to wait to download the entire document before doing anything else.

But there is one place where XHTML still rules the roost: ePub. ePub books are, at their heart, a collection of XHTML documents (now using the XHTML flavour of the HTML Living Standard). This means that:

- Valid, syntactically correct XML markup is needed. Without that, your e-reader will complain. This means self-closing tags, correct namespaces, XML attributes in the XML namespace (`xml:lang`), and so on.
- Other XML languages can be included directly into XHTML by adding namespaces.
- The `epub`namespace is unlocked, which adds additional functionality to your ePub in e-readers.

We’ll come back to that…

## CSS, sort of

So HTML is actually XHTML in ePub. Is CSS some sort of XCSS? Actually, no: CSS is broadly the same as you know it, but with a few quirks.

First up, e-readers and e-reader software are, compared to our normal evergreen browsers, typically really basic. They can run on underpowered hardware, people often keep their e-readers for over a decade, and the engines they use can be positively historic. To put it another way, I’m wary of using `:not()` in ePub CSS for a widely distributed title. While I might be overly cautious here, don’t expect nowadays normal pseudoclasses like `:is()` to have wide support. Luckily, layout tends to be simpler in a document-focused format, and progressive enhancement is possible with `@supports`.

Next, as our markup is now namespace aware, our CSS needs to follow. For example, if you wanted to style a piece of text in a different language that you’ve marked up like:

`<p>As Jean-Paul Sartre said, <q xml:lang="fr">L’enfer, c’est les autres</q>.</p>`Then you can’t simply use an attribute selector like `q[lang]`, you need to define your namespaces and reference them in your selectors using the `|` separator:

```
@namespace xml "http://www.w3.org/XML/1998/namespace";
q[xml|lang] { … }
```
## Strange extensions

As mentioned earlier, namespace support means that other XML-compatible markup languages can be incorporated directly into your XHTML document. These could be just additional semantic attributes, or even new elements. Obviously to get anything useful out of them you need e-reader support, but that’s present in a few cases. Let’s take a look at a couple.

You might actually be familiar with MathML, as it’s supported in HTML5. The way you use it in HTML is that there’s a broad agreement that the contents of the MathML spec will just work when used in HTML. Adding a basic MathML equation using standard MathML tags (`math`, `mi`, `mo`, `mn`, and so on) to your normal HTML document ends up looking something like .

But in XHTML (because it’s an XML language) there’s a standard integration process for any XML language you want to bring in. First, you define your MathML namespace against the root element:

`<html xmlns="http://www.w3.org/1999/xhtml" xmlns:m="http://www.w3.org/1998/Math/MathML">`Then the MathML elements are available to use inside the document under that namespace:

```
<m:math alttext="n + 1">
  <m:mi>n</m:mi>
  <m:mo>+</m:mo>
  <m:mn>1</m:mn>
</m:math>
```
Same goes for SVG: in HTML it’s just allowed to be included with standard SVG tags; using `svg`, `title` and `path` can get you something like . But in XHTML you’ll need to declare the SVG namespace and use that with your images:

`<html xmlns="http://www.w3.org/1999/xhtml" xmlns:svg="http://www.w3.org/2000/svg">````
<svg:svg viewBox="0 0 452 452">
  <svg:title>the HTMHell logo</svg:title>
  <svg:path d="M198.2.94c-28.6 3.3-61.4…"/>
</svg:svg>
```
Of course, we’re talking about ePub here. There’s a W3 ePub specification (currently at version 3.3) that defines the structure and metadata of an ePub document, and gives us the `epub:type` attribute. This attribute (in conjunction with the Structural Semantics Vocabulary specification) can be used in a few ways to improve your collection’s usability within an e-reader. Let’s look at an example.

One thing books often have is endnotes, but there’s no easy way of expressing that semantic state in HTML. In ePub’s vocabulary though we find the `noteref` attribute value. If we use this on a link then readers know to pull in a fragment from another place and typically display then this inside a modal that can be dismissed to return to the existing place. This looks something like:

`<p>ePub is hellish.<a href="endnotes.xhtml#note-1" id="noteref-1" epub:type="noteref">1</a></p>`And in your collection of endnotes:

```
<li id="note-1" epub:type="endnote">
  <p>…unless you read HTMHell. <a href="page.xhtml#noteref-1" epub:type="backlink">↩</a></p>
</li>
```
**Note:** `epub:type` is gradually being deprecated in favour of the `role` values defined in the Digital Publishing WAI-ARIA spec, but currently that functionality is either unimplemented or only available in the latest systems. For example, the above endnotes functionality doesn’t yet work in Apple Books (generally a good e-reader) if you use `role="doc-noteref"` and `role="doc-endnotes"`.

As well as these ePub semantic vocabulary, we also have things like the Z39.98-2012 Structural Semantics Vocabulary. This extends the base ePub spec, and takes the form of `z3998`-namespaced attribute *values*, which definitely looks odd if you’re not used to it. You can use these to express even more fine-grained semantic values, for example Roman numerals (`<span epub:type="z3998:roman">DCLXVI</span>`) or parts of letters. Broad support for extracting any functionality from these is extremely lacking, but when did that ever stop us front-end developers from going over the top?

## Want to try this all out?

Writing an ePub isn’t exactly hard, but there’s some scaffolding necessary to get to a workable structure. You need a `container.xml` file in a `META-INF` directory that points at a package file. The package file contains a bunch of metadata about your ePub, including a manifest of the XHTML files in your book (for example, different chapters), and a spine describing the order they should be shown to the reader. You then add your XHTML files, reference them in the manifest and spine, and finally zip the whole directory up into a single archive and rename it to a `.epub`.

To get you started, I’ve prepared a copy of this blog post as an ePub ready for your favourite e-reader (assuming it’s modern: I haven’t included any compatibility hacks). To inspect the contents we just need to unzip it; try renaming the file to have a `.zip` at the end and opening it with your favourite unarchiver. The markup can be found in `src/epub/text`.

If you want to create your own ePubs I’d personally recommend starting with the Standard Ebooks toolset: it can create unbranded ePubs and has a bunch of compatibility tooling built in. Once installed, you can create a directory with the requisite scaffolding with `se create-draft --white-label`, and then build that into an ePub with `se build`.

I hope you have fun putting your own ebooks together! It’s a useful new skill that reuses a lot of your existing knowledge.

## About Robin Whittleton

Still calling myself a front-end developer, though these days more focused on spreading the good word of accessibility.

Site: robinwhittleton.com

Mastodon: front-end.social/@robinwhittleton

## Comments

There are no comments yet.

## Leave a comment
