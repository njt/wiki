---
url: https://blog.plover.com/lang/bismillah.html
date_fetched: 2026-07-05
backfilled: true
---

| 
    
    Archive: 
 Subtopics: 
 Comments disabled | Tue, 23 Jun 2026 Making the rounds last week was this magnificent article on the complications of Arabic typesetting, An interactive introduction to the terrific experience of rendering Arabic typography and its technical debt. The author, Saleh, promises: 
 And then wow, does it deliver. Don't read my article, go read Saleh's instead, or at least read it first. Still here? Then a disclaimer: I do not know Arabic, not even all the letters, yet. I tried hard to get the details right in this article, but I expect there are misspellings, misstatements of fact, and so on, for which I apologize in advance. In one of my favorite parts of his article, Saleh discusses how, because Arabic script is always cursive, it is important how the letters are joined to one another. Modern Latin script has only a few ligatures, and omitting them is barely noticeable: 
 But in Arabic, the ligatures are important. The text looks grossly wrong without the correct ligatures. Early font engines couldn't render Arabic ligatures properly, and on-screen Arabic text always came out looking ridiculous, with the letters separate like Latin script letters, which is completely wrong for Arabic. Saleh gives this example, which says “hello, world, this is Arabic text”. It should look like this: مرحبا بالعالم، هذا نص عربي But early font renderers rendered it like this: مرحبا بالعالم، هذا نص عربي The crappy rendering was unfortunate, and only barely tolerable,  just
barely better than nothing.  Even if you don't read Arabic (I don't)
you can see the differences.  Notice, for example how the elegant and
symmetric cluster لعا is mangled to لعا.  Or look at just the first
(rightmost) letter.  It is the Arabic letter ‘m’, called  For the supremely important phrase بِسْمِ ٱللهِ ٱلرَّحْمَٰنِ ٱلرَّحِيْمِ the crappy rendering was  “Khalili Collection Islamic Art cal 0154", Ottoman Turkish, 19th century. Public domain, via Wikimedia Commons. It would be blasphemous to render this phrase, called the “basmala”, this crucial expression of honor for God, as a jumble of letters. Imagine if Exodus 20 had had God introducing the Ten Commandments by saying The incredible solution to this one problem was the inclusion in
Unicode of a special codepoint U+FDFD  In Firefox, with my fonts, the glyph renders like this, long and narrow: but on my Android phone there is a very different glyph. Here it is, highly magnified: What's going on here? It's fun to find out. The basmala actually has four words (“bismillah” is a contraction): 
 (At some point I should slip in that when the word “al-” (which means “the”) appears before an /r/ sound, its /l/ is assimilated, so that it is pronounced like /ar-/. This is analogous to what happens when the English prefix “in-” is attached to a word like “relevant”. “Inrelevant” is tricky to say. so the /n/ is assimilated and the word is spelled and pronounced “irrelevant”.) Here are the four words picked out in different colors. To a person literate in Arabic, I suppose this is obvious, but I found it a bit challenging. “Allah” الله is at the top. (I'm told this is traditional.) I've colored it green because green is said to have been Muhammad's favorite color. The two marks above it, the W-shaped mark and the vertical stroke above it, are diacritic marks (one called a shaddah and other other indicating the vowel). I'm not sure how optional they are, but in an earlier draft of this article I tried to explain them in detail and got bogged down in a multi-paragraph digression about the morphology of the word “Allah”, so I'm just going to move on without commenting on them further. Below “Allah”, in red, is “bismi”.  In Arabic this has three letters,
/b/ + /s/ + /m/, since the vowels are omitted.  At the right we have
 بس which is /b/ + /s/; the letters
are named  (Khaled Hosny, designer of the widely used Amiri font, told me that the design of Android basmala glyph is very bad. One of his criticisms was “the bizarre fusion of the letters” and I suppose the attachment of the م and the ن is one of the things he had in mind. He also objected to the insertion of “Allah” into the middle of “bismi”.) The third word, in blue, is al-raḥman الرحمن which as you can see starts with the same letters as al-raḥim الرحيم. You can also see the same first two of those letters at the start of “Allah” الله. As I mentioned before, “al-” means “the”, so you see it at the beginning of many Arabic words. It also survives in many English words that are descended from Arabic, such as alcohol, alcove, algebra, algorithms, and alchemy. (Not, however, “alligator”,
where the “al-” is the  The /r/ sound in al-raḥman and al-raḥim is made by the letter  Let's see, what else?  The only letter I haven't mentioned is the
fifth letter in al-raḥim الرحيم,
just before the  Arabic writing is very beautiful, isn't it?  Last time I tried to
learn the alphabet I got stuck because I was trying to learn the
sounds at the same time, and Arabic sounds are very different from
English sounds.  Arabic has three sounds that resemble English /h/.
One is very soft, one is very rough, and one is in between.   Wikimedia Commons has a gallery of basmalas, and web search produces thousands more. I am looking forward to understanding more of them. 
 |
