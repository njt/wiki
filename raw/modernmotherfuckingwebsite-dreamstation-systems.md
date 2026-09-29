---
url: https://modernmotherfuckingwebsite.dreamstation.systems/
date_fetched: 2026-09-29
---

## Seriously, what the fuck happened?

The original motherfucking website told you in 2013 that your site was an over‐engineered piece of shit. You nodded, laughed, shared it, and then went right back to work and made it **so much fucking worse**.

Now you need Node, a package manager, 1,100 dependencies, a bundler, a transpiler, a meta‐framework, a hydration strategy, and a CI pipeline to put a paragraph on a screen. Your “Hello World” has a lockfile longer than this page. Your blog has a loading spinner. *A blog.* It’s text, you maniac.

And your excuse is always the same: “But we need *modern features*. SEO. Social cards. Performance. Responsive images.”

Cool. Here’s the thing, shithead: **all of that is just HTML.** It’s tags. You type them. With your fingers. Into a file.

## Semantic elements: accessibility is important, dumbass

You wrote `<div class="nav-wrapper-container-inner">` and then installed an accessibility library to explain to screen readers that it’s a nav. There is an element for that. It’s called `<nav>`. It shipped *in Internet Explorer 9*.

```
<header>…</header>
<nav>…</nav>
<main>
  <article>
    <h1>A heading. One of them.</h1>
    <section>…</section>
    <aside>…</aside>
    <time datetime="2026-09-24">today</time>
  </article>
</main>
<footer>…</footer>
```
That’s landmarks, a document outline, keyboard navigation, and it makes reader mode work. Screen readers understand it. Search engines understand it. No ugly motherfucking JavaScript in sight.

You also don’t need a component library for an accordion. Behold, `<details>`:

## A normal fucking disclosure widget

No npm install, no `useEffect`. It works with a keyboard, it works with a screen reader, it’s searchable with find‐in‐page, and it’ll still work in twenty years without having to reconstruct an entire build environment.

Same goes for `<dialog>`, `popover`, `<input type="date">`, and native form validation. There are three major libraries that already worked out the kinks in all of this, they’re called Gecko, Blink, and WebKit.

## JSON-LD: it’s fucking JSON

You installed an “SEO plugin” with its own settings page and a premium tier to generate this:

```
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "This is a modern motherfucking website.",
  "datePublished": "2026-09-24",
  "author": { "@type": "Person", "name": "Robin Reel" }
}
</script>
```
That’s it. That’s structured data. You tell the search engine what the page is using: the one data format every developer on Earth already knows. Pick a type from schema.org (`Article`, `Product`, `Recipe`, `Event`, `FAQPage`), fill in the blanks, paste it in the `<head>`. It doesn’t execute. It doesn’t screw with rendering. It doesn’t need a fucking SDK.

This page has one. Go look.

## Open Graph: it’s seven meta tags

You know that nice card that shows up when someone pastes your link in a chat? You do not need a “social sharing integration” for that. You need this:

```
<meta property="og:type"  content="article">
<meta property="og:title" content="This is a modern motherfucking website.">
<meta property="og:description" content="Zero of the bullshit.">
<meta property="og:url"   content="https://modernmotherfuckingwebsite.dreamstation.systems/">
<meta property="og:image" content="https://modernmotherfuckingwebsite.dreamstation.systems/www.webp">
<meta property="og:image:alt" content="Robert Cailliau’s W3 logo">
<meta name="twitter:card" content="summary_large_image">
```
Done. Slack, iMessage, Discord, LinkedIn, Mastodon, Bluesky, and X will all unfurl it beautifully.

And here’s the part that should really sting: because these tags are **in the actual HTML**, crawlers see them instantly. Your client‐rendered SPA injects them with JavaScript after the crawler has already fucked off, which is why you then had to bolt on server‐side rendering to get back what **just fucking writing HTML** already gave you.

## Resource hints: performance tuning in one line

You hired a consultant and adopted a “performance framework”. Meanwhile the browser has been begging you to just *tell it* what’s coming:

```
<!-- Open the connection early: DNS + TCP + TLS, done before you need it -->
<link rel="preconnect" href="https://cdn.example.com" crossorigin>
<!-- Cheap fallback for the "maybe" origins -->
<link rel="dns-prefetch" href="https://analytics.example.com">
<!-- Fetch the thing the parser can't discover on its own -->
<link rel="preload" href="/fonts/body.woff2" as="font" type="font/woff2" crossorigin>
<!-- Tell the browser which image actually matters -->
<img src="hero.avif" fetchpriority="high" width="1200" height="600" alt="…">
<!-- And which ones don't -->
<img src="footer-junk.avif" loading="lazy" decoding="async" width="400" height="300" alt="…">
```
Want instant page navigations like a “real app”? You don’t need a client‐side router. You need speculation rules, which is (say it with me) *a script tag with some JSON in it*:

```
<script type="speculationrules">
{ "prerender": [{ "where": { "href_matches": "/*" }, "eagerness": "moderate" }] }
</script>
```
Browsers that support it prerender the next page on hover. Browsers that don’t support it ignore it. Nobody gets a broken back button. Add `@view-transition { navigation: auto; }` to your CSS and your plain old multi‐page site cross‐fades between pages like the SPA you spent nine months building.

A word of warning, because you *will* overdo it: if you preload everything, you’ve prioritized nothing. Hint your actual fucking content and not a fucking PDF behind two user clicks.

## Modern image formats: the browser picks, you chill

Your hero image is a 4 MB PNG. I know it is. Don’t lie to me.

AVIF and WebP are routinely half the size of a JPEG at the same quality. “But users’ browsers might not support them!” Bullshit, AVIF shipped in Chrome six years ago and WebP did OVER A DECADE AGO. And you don’t need WordPress’s “image optimization service” or a magic framework `<Image>` component to serve them, because HTML has had format selection built into a tag for years:

```
<picture>
  <source type="image/avif"
          srcset="hero-800.avif 800w, hero-1600.avif 1600w"
          sizes="(max-width: 40rem) 100vw, 40rem">
  <source type="image/webp"
          srcset="hero-800.webp 800w, hero-1600.webp 1600w"
          sizes="(max-width: 40rem) 100vw, 40rem">
  <img src="hero-800.jpg" width="1600" height="800"
       alt="Something worth the bytes">
</picture>
```
You don’t need to sniff the user agent for that. The browser takes the first format it understands, at the size that fits the screen. The `width` and `height` attributes reserve the space so your layout doesn’t jump around like a caffeinated toddler, which is your Cumulative Layout Shift score fixed with two attributes.

“But I need a build step to make the AVIFs!” No, you need to run one command, one time, when you add the image:

```
avifenc hero.png hero.avif
cwebp -q 80 hero.png -o hero.webp
```
That’s not a build system. That’s writing a fucking file that you then serve.

## You don’t need a build system to write tags

Let’s review what you thought required a toolchain:

- **Accessibility landmarks:**element names.
- **Structured data:**a script tag with JSON in it.
- **Social cards:**meta tags.
- **Performance hints:**link tags and two attributes.
- **Next‐gen responsive images:**a picture tag.
- **Dark mode:**one media query. This page has it. Go flip your OS setting, I’ll wait.
- **Responsive layout:**- `max-width`. One property. Your phone is already fine.

Every one of these is a *platform feature*. It’s in the browser. It was already downloaded, by the user, before they ever heard of you. The framework doesn’t give you these things. At best it gives you a slower way to type them, and at worst it hides them behind a plugin ecosystem so you forget they were free.

Other than the Open Graph image and the WCAG badge, this page is a single HTML file of about 23 KB. It has no dependencies, no cookie banner, no hydration errors and no security problems. The deploy process is “copy the file to the served directory”.

## “Yeah, but…”

## “But I have 500 pages and a shared header.”

Fine. That’s a real problem, and a static site generator or server‐side includes are proportionate answers. Notice how that is *not* the same thing as shipping a 2 MB runtime to every visitor so their phone can assemble your header for you. Generate HTML, serve HTML.

## “But I’m building an actual application.”

Then use an actual application framework, you beautiful bastard. If you’re building a spreadsheet, a video editor, or a design tool, go nuts. But be honest: you’re building a marketing site, a blog, a docs page, or a restaurant menu. It’s a document. The web was *invented* for documents. It’s extremely fucking good at them.

## “But developer experience!”

Your developer experience is waiting 40 seconds for a dev server to boot so you can change a word. Mine is pressing save and hitting refresh. Also, nobody visiting your site cares about your developer experience.

## “But it looks plain.”

So write some CSS. Modern CSS has nesting, variables, grid, container queries, `:has()`, `clamp()`, and color functions built in. You don’t need a preprocessor for those anymore either. You don’t need JavaScript for them either. The point isn’t “never style anything”. The point is that every byte should be there because *you chose it*, not because it fell out of `node_modules`.

## This is a website. Look at it.

View source. Seriously, right‐click, do it. Everything I just described is sitting right there in plain text, readable top to bottom by a human being, the way the web was meant to be. No source maps required, because the source *is* the map.

Start with HTML. Add what you need. Stop when you’re done. That’s the whole fucking methodology.
