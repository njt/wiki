---
url: https://gyrovague.com/2023/08/05/archive-today-on-the-trail-of-the-mysterious-guerrilla-archivist-of-the-internet/
date_fetched: 2026-07-05
backfilled: true
---

Do you like reading articles in publications like Bloomberg, the Wall Street Journal or the Economist, but can’t afford to pay what can be hundreds of dollars a year in subscriptions? If so, odds are you’ve already stumbled on archive.today, which provides easy access to these and much more: just paste in the article link, and you’ll get back a snapshot of the page, full content included.

For a long time, I assumed that this was some kind of third-party skin on top of the venerable Internet Archive, whose Wayback Machine provides a very similar service at the very similar address of archive.org.  However, the Wayback Machine is slow, clunky, frequently errors out, and most importantly, it’s very easy for websites to opt out, retroactively erasing all their content forever.  In contrast, archive.today has *no *opt-outs or erase buttons: like it or not, they store everything and it’s not going anywhere, with some limited exceptions for law enforcement, child porn, etc.

The Internet Archive is a legitimate 501(c)(3) non-profit with a budget of $37 million and 169 full-time employees in 2019. archive.today, by contrast, is an opaque mystery. So who runs this and where did they come from?

## The origins and owners of archive.today

The first historical record we have of the site dates from May 16, 2012, when a “Denis Petrov” from Prague, Czech Republic registered the domain archive.is, the original name of the site.  archive.today followed in 2014, and the site has since registered countless variations: archive.li, archive.ec, archive.vn, archive.ph, archive.fo, etc.  Denis Petrov is a common Russian name, with pages and pages of matches on LinkedIn, but it may well be an alias: informer.com notes that the same contact information was used to register a series of very sketchy domains, ranging from “carding forum” *verified.lu* to piracy sites *btdlg.com* and *moviesave.us* (all long since gone), many seeded with German keywords (*spiel, gewinnt, online*).

Domains aside, “Denis Petrov” has little presence on the web, and three seemingly connected domains proved dead ends. The obvious *denispetrov.com* was an entertaining rabbit hole, with the author an accomplished programmer with an interest in Web automation, but it’s clearly the work of a New Yorker, they’re blogging at the tail end of a 25-year career and the blog dries up entirely in 2011, so it doesn’t match the place or time. denis.biz (2001) and petrov.net (1998!) contain nothing.  The one intriguing bit of evidence we have is this series of screenshots (archive) where Brave’s tech support addresses *webmaster@archive.is* as “Denis”, but odds are that’s just from the same DNS record.

We can glean a few more clues from *archive.today*‘s web presence.  The FAQ, unchanged since 2013 (!), states that they are located in Europe and asks for PayPal donations in euros.  Looking through the voluminous Tumblr blog, featuring tons of questions but very terse answers, the author’s English is excellent but not *quite *native, with occasional Noun Capitalization also hinting at a German background.  Yet they answer questions in Russian, and the site uses a Russian analytics engine.

The most interesting detective work to date comes from Stack Exchange, where Ciro Santilli managed to link the profile picture of an account *archive.today* once used to archive LinkedIn content to a “Masha Rabinovich” in Berlin.  Even more intriguingly, in a 2012 F-Secure forum post, a “masharabinovich” complains about *“my website http://archive.is/”* being blacklisted.  They pop up on Wikipedia as well getting told off for adding too many links to archive.is, including a mention that they’re using the Czech ISP fiber.cz, and their early edit history includes many updates to the pages “Russian passport” and “Belarusian passport”.  “Masha” (Маша) is a common Russian diminutive of Maria, although it can also be a Hebrew form of Moses (מַשה), and Rabinovich is an Ashkenazi Jewish surname.

 Early Github captures on *archive.today *are linked to a now completely disappeared account called “volth” (copy archived by archive.today itself), who was a fluent speaker of Russian, contributed extensively to NixOS (which archive.today uses) and has a profile picture not dissimilar to Masha’s.  The linked volth.com domain is now only an empty husk, but it dates back to 2004, with early versions first doing some kind of sketchy search engine network marketing thing (2005), promising “Total Success in Internet” (2008) and eventually being put up for sale (2010), making it likely that its original owners the Espinosas are unrelated to whoever owns the domain today.

While we may not have a face and a name, at this point we have a pretty good idea of how the site is run: it’s a one-person labor of love, operated by a Russian of considerable talent and access to Europe. Let’s move on to the nitty gritty.

## Infrastructure

There are two components to any archival site: the scraper that copies the pages, and the storage system where the pages are kept and retrieved on demand. Helpfully, the FAQ shares some details of what the storage side at least used to look like:

The archive runs Apache Hadoop and Apache Accumulo. All data is stored on HDFS, textual content is duplicated 3 times among servers in 2 datacenters and images are duplicated 2 times. Both datacenters are in Europe, with OVH hosting at least one of them.

In 2012, the site already had 10 TB of archives and cost ~300 euros/mo to run, escalating to 2000 euros by 2014 and $4000 by 2016. As of 2021, they have archived on the order of 500 million pages, and with the average size of a webpage clocking in at well over 2 MB these days, that’s a cool 1,000 TB to deal with. (For comparison, the Internet Archive is around 40,000 TB.)

The less discussed but more controversial half of the site is scraping, the process of vacuuming up live webpages. Since 2021, this uses a modified version of the Chrome browser, and the blog readily admits that the availability of computing power to run these automated browsers is now the main bottleneck to expanding the site. To avoid detection, archive.today runs via a botnet that cycles through countless IP addresses, making it quite difficult for grumpy webmasters to stop their sites getting scraped. Access to paywalled sites is through logins secured via unclear means, which need to be replenished constantly: here’s the creator asking for Instagram credentials.

Finally, the serving of the website is also subject to a perpetual game of cat and mouse: *“I can only predict that there will be approximately one trouble with domains per year and each fifth trouble will result in domain loss.”*  As of today, archive.today still works, but users are redirected to archive.md.

## Funding

The other major source of permanent uncertainty is the site’s funding model. We’ve established that its costs are considerable, but according to the creator, as of 2021 ads and donations covered less than 20% of expenses, with donations on the order of 6000 euros. PayPal donations, previously accepted, were switched off around 2022 since the creator could no longer top up the account, implying they’re in Russia, and they complain about the difficulty of doing cross-border payments “across the Iron Curtain”. Donations these days are via Liberapay, an obscure French non-profit organization, and YC-backed startup BuyMeACoffee. Surprisingly, the creator has a healthy skepticism of crypto, so this remains unsupported.

The other source of income is ads.  The FAQ, far out of date, has a “*promise it will have no ads at least till the end of 2014*“, but there have long been Yahoo network ads injected on top of pages when you use mobile (but, oddly, not on desktop).  Revenue is even more of a question mark, but apparently on good days they “almost cover expenses” (a remark that doesn’t quite square with the other comment about ads and donations together covering less than 20%), while on bad days they’re getting kicked out from serving ads because an archive of the Internet will inevitably archive advertiser-unfriendly NSFW content too.

## Archive.today, not tomorrow?

So there we have it: the site is a one-man battle against entropy, constantly battling domain registrars, anti-scraping systems, copyright enforcement, easily spooked advertisers, and global financial system payment rails designed to obstruct Russian citizens. By staying anonymous and keeping a low profile, they’ve (likely?) managed to avoid the kind of legal tussles that have embroiled Alexandra Elbakyan of Sci-Hub fame, but they’ve still funded it to the tune of tens of thousands of euros during that time. They clearly have a second source of considerable income that’s likely somewhat sketchy as well, so if that ever goes away, archive.today is likely to go away with it.

The creator is fully aware that the site is a mere “weak tool” that is “doomed to die“, but the bus factor of one combined with its semi-legal nature means there can be no real continuity: there will never be a legally incorporated Archive.Today Foundation to carry on his work. It’s a testament to their persistence that they’re managed to keep this up for over 10 years, and I for one will be buying Denis/Masha/whoever a well deserved cup of coffee.

*All images in this post feature the Bibliotheca Alexandrina at Alexandria, Egypt.*

> ranging from “carding forum” verified.lu to piracy sites btdlg.com and moviesave.us (all long since gone), many seeded with German keywords (spiel, gewinnt, online).

btdig.com still exists, and is also run by the same guy
