---
url: https://gyrovague.com/2026/02/01/archive-today-is-directing-a-ddos-attack-against-my-blog/
date_fetched: 2026-07-05
backfilled: true
---

Around January 11, 2026, archive.today (aka archive.is, archive.md, etc) started using its users as proxies to conduct a distributed denial of service (DDOS) attack against Gyrovague, my personal blog. All users encountering archive.today’s CAPTCHA page currently load and execute the following Javascript:

```
        setInterval(function() {
            fetch("https://gyrovague.com/?s=" + Math.random().toString(36).substring(2, 3 + Math.random() * 8), {
                referrerPolicy: "no-referrer",
                mode: "no-cors"
            });
        }, 300);
```
Every 300 milliseconds, as long as the CAPTCHA page is open, this makes a request to the search function of my blog using a random string, ensuring the response cannot be cached and thus consumes resources.

You can validate this yourself by checking the source code and network requests; if you’re not being redirected to the CAPTCHA page, here’s a screenshot. uBlock Origin also stops the requests from being executed, so you may need to turn that off. At time of writing, the code above is located at line 136 of the CAPTCHA page’s top level HTML file:

So how did we end up here?

## Background and timeline

On **August 5, 2023**, I published a blog post called archive.today: On the trail of the mysterious guerrilla archivist of the Internet. Using what cool kids these days call OSINT, meaning poking around with my favorite search engine, the post examines the history of the site, its tech stack and its funding. The post mentions three names/aliases linked to the site, but all of them had been dug up by previous sleuths and the blog post also concludes that they are all most likely aliases, so as far as “doxxing” goes, this wasn’t terribly effective.

My motives for publishing this have been questioned, sometimes in fanciful ways. The actual rationale is boringly straightforward: I found it curious that we know so little about this widely-used service, so I dug into it, in the same way that previous posts dug into a sketchy crypto coin offering, monetization dark patterns in a popular pay to win game, and the end of subway construction in Japan. That’s it, and it’s also the only post on my blog that references archive.today.

The post gathered some 10,000 views and a bit discussion on Hacker News, but didn’t exactly set the blogosphere on fire. And indeed, absolutely nothing happened for the next two years and a bit.

On **November 5, 2025**, Heise Online reported that the FBI was now on the trail of archive.today and had subpoenaed its domain registrar Tucows. Both this report and ArsTechnica also linked to my blog post.

On **November 13**, AdGuard DNS published an interesting blog post about a sketchy French organization called Web Abuse Association Defense (WAAD), which was trying to pressure them into blocking archive.today’s various domains. An update added on November 18 also suggests that WAAD is impersonating other people.

On **January 8**, **2026**, my blog host Automattic (dba WordPress.com) notified me that they had received a GDPR complaint from “Nora”, alleging that my blog post *“contains extensive personal data … presented in a narrative that is defamatory in tone and context”*. The complaint was entirely lacking in actionable detail, so I had Gemini compose a rebuttal citing journalistic exemption, public interest, failure to identify falsehoods, and host protection, and after a quick review Automattic sided with me and left the post up. Score one for AI.

On **January 10**, I received a politely worded email from archive.today’s webmaster asking me to take down the post for a few months. Unfortunately the email was classified as spam by Gmail and I only spotted it five days later. I responded on the 15th and followed up on the 20th, but did not hear back.

On **January 14**, a user called “rabinovich” posted Ask HN: Weird archive.today behavior? on Hacker News, asking about the DDOS-like behavior which they claimed had started three days ago. This is, as far as I can tell, the first public mention of this anywhere, and a kind HN user brought it to my attention.

On **January 21**, commit ^bbf70ec (warning: very large) added gyrovague.com to dns-blocklists, used by ad blocking services like uBlock Origin. This is actually beneficial, since if you have an ad blocker installed, the DDOS script’s network requests are now blocked. (It does not stop users from browsing to my blog directly.)

On **January 25**, I emailed archive.today’s webmaster for the third time with a draft of this blog post, declining to take down the post but offering to “change some wording that you feel is being misrepresented”. “Nora” responded with an increasingly unhinged series of threats:

*And threatening me with Streisand… having such a noble and rare name, which in retaliation could be used for the name of a scam project or become a byword for a new category of AI porn… are you serious?*

*If you want to pretend this never happened – delete your old article and post the new one you have promised. And I will not write “an OSINT investigation” on your Nazi grandfather, will not vibecode a gyrovague.gay dating app, etc.*

At this point it was pretty clear the conversation had run its course, so here we are. And for the record, my long-dead grandfather served in an anti-aircraft unit of the Finnish Army during WW2, defending against the attacks of the Soviet Union. Perhaps this is enough to qualify as a “Nazi” in Russia these days.

## Speculation

The above are easily verifiable facts, although you’ll have to trust me on the email bits. (You can find a lightly redacted copy of the entire email thread here.) Everything that follows is more speculative and firmly in the domain of a hall of mirrors where nothing is quite what it seems.

The big question is, of course, **why**, and more specifically **why now**, 2.5 years after posting, when the cat is well and truly out of the bag. As multiple people have noted, there’s nothing the Internet loves more than an attempt to attempt to censor already published information, and doing so tends to cause *more *interest in that information, aka the Streisand effect.

To summarize our email thread, the archive.today webmaster claims they have no beef with my article itself, but they are concerned that it’s getting misquoted in other media, so it should be taken offline for a while. And in this Mastodon thread by @eb@social.coop, @iampytest@infosec.exchange quotes claimed correspondence with the webmaster, stating that the purpose of the DDOS was to “*attract attention and increase their hosting bill*“.

Call me naive, but I’m inclined to take that at face value: it’s a pretty misguided way of doing it, but they certainly caught my attention. Problem is, they also caught the attention of the broader Internet. They didn’t do so well on the hosting bill part either, since I have a flat fee plan, meaning this has cost me exactly zero dollars.

Perhaps more interesting yet are the various identities involved.

- “Nora”, who sent the GDRP takedown attempt and replied to my emails to archive.today, shows up in various places on the Internet including Hacker News, commenting on my original blog post back in 2023. Somebody by that name also has an account on Russian LiveJournal, where they posted correspondence between btdigg.com and an anti-piracy outfit called Ventegus. There’s also this rather batty exchange on KrebsonSecurity, where “Nora” says various scammers are actually Ukrainian, not Russian, and a “Dennis P” pops up to call her “fake” and a “scammer”. 
 - *Updated*- *20 Feb 2026*: It appears increasingly likely that the identity of “Nora” has been appropriated from an actual person, whose only connection to archive.today was a request to take down some content. As a courtesy, I have redacted their last name from this post.
 
- “rabinovich” on Hacker News submitted both the “Ask HN” about the DDOS attack, and an apparently competing archive site called Ghostarchive. As several HN readers noted, the name “Masha Rabinovich” is associated with archive.today.
- “Richard Président” from WAAD helpfully reached out and offered to assist me with a GDPR counter-complaint, rather transparently mentioning that this could be tied to “a request for identity verification”. (I have zero interest in pursuing this.)

## Conclusion

Well, I wish I had one, but at this stage I really don’t. The most charitable interpretation would be that the investigative heat is starting to get to the webmaster and they’re lashing out in misguided self-defense. Perhaps I’ll just quote a post by “Nora” on LiveJournal:

*And as the darkness closed in, Nora [redacted], once a seeker of truth, was swallowed by the very shadows she had sought to expose. Her name would be whispered in hushed tones by those who dared to tread the path of forbidden knowledge, a cautionary tale of a mind consumed by the cosmic horrors that lie just beyond our comprehension.*

Let’s see what the Internet hive mind comes up with.

Also, for the record, I am gyrovague-com on Hacker News and Gyrovagueblog on Wikipedia.

all archives of this domain on archive.is are being censored with the default nginx page lol

I wonder if it had ever occurred to you that many people, myself included, use archive.today services as perhaps the only way to keep informed. Whether you like it or not, your investigation is part of an attack on an archive, and on archives in general, and on the utility of these things for many people who, unlike you perhaps, cannot access things in the usual way. I find your rationale to be entirely self centered and low IQ and maybe you should look in the mirror and try to understand the damage you do to others, in probably many ways, not only with respect to archive.today, but in many ways yet to be uncovered. As for the DDOS, good. You deserve it. You don’t get to harm people and not be harmed. You don’t get to punch someone in the face for “boringly straightforward” reasons and not get punched back. Of course, it permits you yet another round of self serving attention seeking. Keep in mind that you’re pissing off many more people than just archive.today. When archive.today shuts down, in part because of your complicity, I’ll lose access to ~40% of news sites. When I think about who to blame, your name will be near the top of the list. Think about that.

the service may be good. This article is about its people, who’ve demonstrated poor decision-making ability. The coverage is justified and entertaining.

i find this description more fitting of yours, having had the misfortune of reading it 😞 as it were, i still don’t have nearly your level of self-pity 😊

What “doxx”? Are you on drugs? Not only was that info not secret and not initially revealed or discovered by this person (just reported on), but some names (which are probably not even real) alone doesn’t constitute doxxing.

Loser. Go back to living in your broke down dacha

I found this post on HN. I very much agree with user “It doesn’t matter”. I am a very curious person by nature, and I understand your attempt to find the owner of archive.today was innocent, but if this person is indeed Russian, and in time of a war, OP you almost doxxed them. I am neither Russian nor Finnish nor in the US but in Middle East or China, for instance, if a neighbor accuses you of commiting a crime even if you didn’t, or being a member of a political part even if you aren’t, they can raid your house and… let’s not discuss this further. You NEED to hide this blog post just like this person asked you to https://gyrovague.com/2023/08/05/archive-today-on-the-trail-of-the-mysterious-guerrilla-archivist-of-the-internet/ Yes, a lot of time has passed since they asked you to hide in the email, and they might not be a native English speaker so you offended them without meaning to, but you still need to understand how bad someone in a country like that may have it, and you NEED to comply to their request for their safety.

This is clearly the same person replying. At least try to change your writing style.

You can think whatever you want about the author’s morality (and be wrong), but the fact of the matter is that archive.today is undeniably in the wrong for revenge DDoS-ing a personal blog, both morally and legally.

When the owner of the archive has shown themselves to be malicious, and now is censoring archive content (this blog), I don’t think they should be considered reliable.

You are either “Nora” or very silly. Probably both.

Your made up scenario of “losing access to ~40% of news sites” is not convincing. Theres TOR, VPNs, cgi-proxies you can self-host, Starlink.

Asking the question of who is operating one of the most widely known services on the internet is very valid. The archive.today admins DDOS response to asking this question confirms that we should ask ourselves that question, because obviously they’re vengeful, apparently not trustworthy and shady.

The whole Nazi take is so obviously an uneducated russian talking point, that I’d consider the whole argument to be a smoke screen and the auther not really being russian.

i don’t know anything about this blog, but now that someone is going to try and take down the info, I sense injustice somewhere. I will read this and save a copy in case it goes down.

.

“I found it curious”. clearly you could have omitted personal info if the only reason you did the doxx was because you were curious. seems more like you want to discredit / shut down this tool thousands use to inform themselves about the world

Did you read the original post? All of it was incredibly public.

Lol what a dick move! The dude even made some fake puppet accounts to comment on this thread. Trying to take down your content goes entirely against the principle of archiving…

A Reddit user says that photos of his minor daughters have been on archive.today for years, and that the site’s operator has refused to remove them. He also claims that the operator lives in New York.

https://www.reddit.com/r/DataHoarder/comments/12trawt/has_anyone_ever_actually_spoken_to_denis_petrov/

According to this WaybackMachine, in 2014 the donation button on archive.today pointed to a small kitten charity in New York.

https://web.archive.org/web/20140723052310/http:/archive.today/

Unrelated to the topic but looking into that reddit users account they seem to be pretty well known for being severely mentally ill so I’d take that source with a large grain of salt

Mr. Archive.today? Denis Petrov? Or whatever name you’re using, seriously, stop creating fake accounts to spam this thread. You’re just embarrassing yourself at this point…

No the redditor lmfao look into it theyre a known lolcow I agree with the sentiment that the archive owners are probably not that good but I really wouldnt use someone who supposedly has kids named “Star Queen Genevieve” and “Lady Margaret” as evidence just saying

I feel like the criticisms along the lines of “you’re exposing the archive.today operator to retaliation by government agencies that don’t like what they’re doing” miss the fact that your blog post is a collection of publicly accessible information that those agencies are more than capable of finding and figuring out themselves.

If only the archive.today operator wasn’t such a overly sensitive deranged weirdo. I really love his service, but he gets so offended over small things and does this weird stuff like when he blocked Brave browsers and CloudFlare’s 1.1.1.1 DNS. Why is it that so many people that run great things have to be these insane schizophrenics?

Random blogger here who stumbled across your blog post via Heise. I’m a user of archive.is (and luckily uBlock Origin), but after reading your post, I might rethink using the archive service.

Thank you for your work on this.

You know what I think is hilarious? Archive.today is notorious for refusing to remove information no matter what, even when it’s highly sensitive, yet they’re obsessively censoring the trail of breadcrumbs they left themselves as much as possible (deleting accounts you found, requesting Archive.org exclusions, etc) suggesting you found something that made them very uncomfortable.

Ironically, they deleted the lj.rossia.org account, but as of the time of writing this comment, they have failed to remove it from their own website (as they have some other pages related to the breadcrumbs about their identity, such as your initial blog post).

https://megalodon.jp/2026-0212-0110-00/https://archive.ph:443/ElZxO

There is a significant trust issue when we can’t know the provider of the site, what their policies are and how they are vetting what is posted here. Misinformation/disinformation can be spread by as simple methods as choosing what to include or exclude, a tactic that was used extensively by WikiLeaks to promote certain narratives. Obviously the archives themselves can be tampered with, and as a great volume of them are paywalled, such tampering is not easily identified.

I do think it is relevant to ask who is behind the site, what their policies are, and how they are being held accountable. I understand the legal risks for them, however there are jurisdictions that can provide a shield and that is what many other efforts do to avoid consequences.

I personally do not see how anyone can trust the contents of these sites given the murky nature of their administration, ownership and the actions they have taken to anyone asking even the most mild of questions.

Out of curiosity, why don’t you want to take down the previous article (even temporarily)?

If you are genuinely wondering what part of the article they have issue with, they clearly don’t want their personal details discussed in the article, so why don’t you just remove their dox from the article?

A

Wikipedia Request for Comment (RFC)has been opened in response to the recent DDoS activity linked to archive.today, including the attack described on this blog.The discussion is about which options should be adopted regarding archive.today:

An editor has requested comments from other editors for this discussion.

This is not a majority vote, but a discussion among Wikipedia contributors. Consensus is based on the strength of arguments, not simply counting votes.

If you’ve been affected by or are concerned about this DDoS incident, you’re invited to participate. Your perspective is welcome.

https://en.wikipedia.org/wiki/Wikipedia:Requests_for_comment/Archive.is_RFC_5

I’ve noticed that the landing page of that site includes a reference to the Russia Today website, which is a known Russian government propaganda outlet.

Seems like they have activated their famous bot farms to write stupid comments on your posts.
