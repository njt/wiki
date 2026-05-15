# Estimating Pi with a Coin

Toss a coin until the first time the observed proportion of heads exceeds one half (cumulative heads exceeds cumulative tails) and record that fraction. Start afresh, repeat. The average of the fractions approaches pi/4. Beautiful mathematics connecting coin tosses to Catalan numbers to pi.

---

## Key Quotes

> The underlying "Catalan-number series identities appear implicitly in the probability theory literature" but this specific interpretation "appears to be novel."

## Key Themes

#mathematics #probability #monte-carlo #pi

The method: toss a fair coin repeatedly until heads are ahead of tails for the first time. Record the fraction of heads at that moment. For example, if the sequence is tails, heads, tails, heads, heads, you record 3/5 (three heads out of five tosses). Start over. Repeat many times. The average of all recorded fractions converges to pi/4.

This is a Monte Carlo method -- estimating a mathematical constant through random sampling. What makes it delightful is how few moving parts there are. You need a coin and patience. The connection to Catalan numbers (which count the number of ways to arrange balanced parentheses, among many other things) is the mathematical depth beneath the simple procedure.

## Critical Analysis

Jim Propp's contribution is the interpretation, not the underlying mathematics. The Catalan-number series identities were already known. But reframing them as "toss a coin, get pi" is the kind of mathematical storytelling that makes abstract results tangible.

This is a 3-4 page paper. Its value is elegance, not utility -- you wouldn't actually estimate pi this way (the convergence is slow). But it joins the pantheon of "surprising ways to find pi" alongside Buffon's needle and the Leibniz series. The surprise is the point.

---
*Sources: [[raw/estimating-pi-with-a-coin]]*
*Last updated: 2026-05-14*
