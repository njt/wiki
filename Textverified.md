# Textverified

Temporary US phone numbers for SMS and voice verification — carrier SIMs, not VoIP, so they pass anti-VoIP blocks. Pay per verification, rent short-term, or lease indefinitely. API available. Crypto payments accepted. Operating since 2019.

---

Textverified is a SaaS service that rents out real US mobile numbers backed by physical SIM cards. You use them to receive the verification codes (SMS or voice calls) that online services require for account creation. Because the numbers are real carrier lines rather than VoIP (Google Voice, etc.), they work with services that block virtual numbers.

## Key Tiers

- **One-time verification** — $0.25+ per use. Get a number, receive one code, done.
- **Non-renewable rental (1–14 days)** — $1.50+. One number, unlimited verifications during the rental window.
- **Renewable rental** — $5/month+. Keep a number indefinitely. "Own your number forever," per their marketing.

The pricing is transparent and the credit system means you only pay for what you use. For heavy users there's an API and bulk discount pricing.

## What Makes It Different

> "Unlike Google Voice, we only use non-VoIP numbers for a simple reason: so many sites and app registration forms are able to detect and block VoIP numbers."

This is the core value proposition. Most free temporary number services are caught by anti-spam filters. Textverified's physical SIM approach means they're not.

## API & Automation

A documented REST API at `/docs/api/v2` lets you integrate verification into automated workflows. Combined with crypto payments, this enables fully anonymous, programmatic account creation — which is both the feature and the ethical challenge.

## Critical Analysis

This is a straightforward utility service, and that's fine. The product does one thing well: let you get a verification code without giving out your real number. The pricing is reasonable, the tiers make sense, and the physical SIM approach is genuinely different from the sea of free VoIP-based alternatives.

The ethical valence depends entirely on use case. Protecting your privacy when signing up for a throwaway account? Reasonable. Bypassing bans, astroturfing, or scaling abuse? The same tool enables both. Textverified's marketing emphasizes privacy protection, but the API + crypto payment combo is optimized for anonymity at scale — that's a power tool, and power tools cut both ways.

The free tier at `/free` is a nice gesture but practically useless: public numbers get hammered, and services that enforce one-number-per-account will already have those numbers burned.

For a wiki reader: if you need a one-off verification code and don't want to give out your real number, Textverified is the pragmatic choice. The $0.25 minimum is cheaper than the privacy cost of doxxing your phone number to every service that demands one.

## Related

- No existing wiki pages on phone verification or anonymity infrastructure — this is a first entry in what could become a privacy-tools cluster
- Related conceptually to [[Security and Sandboxing]] (isolation patterns), [[Agentcookie]] (session identity management)

---
*Source: [[raw/textverified]]*
*Last updated: 2026-06-15*
