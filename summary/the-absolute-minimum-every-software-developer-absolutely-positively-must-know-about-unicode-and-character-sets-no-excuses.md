---
url: https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/
title: The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!)
author: Joel Spolsky
published: 2003-10-08
site: Joel on Software
topics:
  - software-engineering-craft
---

Joel Spolsky's canonical 2003 introduction to character encodings and Unicode, written as an exasperated corrective to widespread programmer ignorance. It remains the single most-referenced plain-English explanation of why "plain text" is a fiction, what a code point actually is, and how UTF-8 saved the internet.

The article walks chronologically through the history: ASCII's 7-bit English-only world, the OEM code page free-for-all where every region stuffed its own characters into 128–255, the ANSI codification of that chaos into named code pages, and the Asian DBCS mess where some characters took one byte and others two. This history sets up Unicode as the necessary fix — a single character set assigning every letter in every writing system a unique *code point* (e.g. U+0041 for A, U+0639 for Arabic Ain).

Spolsky's key conceptual move is separating the platonic letter (the code point) from its on-disk representation (the encoding). This lets him explain why there are multiple Unicode encodings: UCS-2/UTF-16 (two bytes per character, with the byte-order-mark endianness problem), UTF-8 (1–6 bytes, backward-compatible with ASCII for English text), UTF-7 (7-bit safe for draconian email systems), and UCS-4 (four bytes per character, memory-profligate). He also notes that old encodings like Latin-1 can represent *some* Unicode code points but will silently corrupt others into question marks.

The single most important fact: **It does not make sense to have a string without knowing what encoding it uses.** There is no such thing as plain text. Every "my website looks like gibberish" bug traces back to this. Spolsky covers the practical mechanisms — HTTP `Content-Type` headers, HTML `<meta charset>` tags, Internet Explorer's statistical language-guessing — and explains why Postel's Law fails here: being liberal in what you accept means guessing wrong occasionally.

The article closes with practical guidance: use UCS-2 internally (as Windows does), publish as UTF-8. Joel on Software's 29 language editions have never had an encoding complaint.
