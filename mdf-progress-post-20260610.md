---
title: "MDF Two Weeks On: Real Lightning Payments, a WordPress Plugin, and What the Logs Told Us"
date: 2026-06-10
description: "When we published the MDF proposal, both payment rails were stubs. Now real sats are moving, the demo is listed in payment indexes, and our own WordPress logs make the case better than we could."
tags: ["ai", "open-source", "web-standards", "self-hosted", "bitcoin", "lightning"]
authors: ["Gary Walker", "Graham Hall"]
---

When we [published the MDF proposal](https://blog.bitcryptic.com/blog/mdf-markdown-first) a couple of weeks ago, we were upfront that the payment layer of the reference implementation was stubbed — structural validation only, no real money moving. We also said the open questions section was a genuine invitation, not a rhetorical one.

Both of those things have changed in ways worth reporting. Real payments are flowing, and the community pushed back in exactly the ways we hoped.

Also, since then, Cloudflare published a number that reframes the whole conversation: **57.4% of web traffic is now automated**. The web crossed the human/agent threshold ahead of most predictions. The question is no longer whether sites should think about agent consumption — it's whether they want any say in how it happens.

## Lightning Payments Are Live

The headline: **L402 payment verification is production-complete**. The demo server at [mdf-demo.bitcryptic.com](https://mdf-demo.bitcryptic.com) now creates real Lightning invoices, issues HMAC-bound macaroon credentials, and verifies payment preimages against settled invoices. We've tested it end-to-end with real sats over a live Lightning channel.

Which means this works, right now, for anyone:

```bash
# Returns 402 with a real Lightning invoice in the challenge
curl -H "Accept: text/markdown" https://mdf-demo.bitcryptic.com/micropayment/intro
```

Pay the one-satoshi invoice with any Lightning wallet, present the preimage, get the content. The full MDF loop — discover, get challenged, pay, access — is no longer a diagram. It costs less than a hundredth of a cent and settles in under a second, which is exactly the property that makes "price as access policy" viable at micropayment scale.

The stack behind it is deliberately boring: [Alby Hub](https://albyhub.com), self-hosted in Docker, talking to the MDF server over a REST API. That's a design point, not an accident. The MDF spec mandates that payment occurred — it doesn't mandate *how you verify it*. Alby Hub is one answer for Lightning; run whatever node implementation you like.

The demo's three paid tiers are also now listed on [402index.io](https://402index.io), an emerging directory of L402-enabled endpoints — domain-verified and health-checked. Agents looking for payable content can discover ours without ever visiting this blog.

x402 — the EVM/stablecoin rail — remains stubbed, and the reason why turned out to be one of the more interesting problems in the whole architecture. More on that below.

## The Community Did Its Job

We opened issues across the ecosystem — the x402 community repo, the llms.txt project, 402index — asking the questions our open questions section flagged. The responses materially improved the spec's direction.

The thorniest issue is x402 receipt verification: when an agent presents proof of an on-chain payment, *who checks the chain, and why do you trust them?* A server verifying against a public RPC endpoint has just reintroduced a trusted third party into a system that was supposed to remove one.

Three threads came out of raising this publicly. One contributor proposed naming "off-chain-settled x402" as a distinct profile — a pattern where settlement happens before the token exchange, eliminating the RPC trust problem entirely for that class of deployment. Separately, the team behind an agent-receipt format filed an IETF Internet-Draft (`draft-krausz-verification-state-00`) covering offline-verifiable signed verdict artifacts — prior art on the same binding problem from a different angle. And for native on-chain verification, we're building toward a TEE-attested verification oracle so that MDF operators won't need to run blockchain infrastructure at all — the same relationship Alby Hub has to L402. That's in active development; it gets its own write-up when it ships.

None of this was in the original proposal. All of it came from publishing the open questions and meaning it.

## Eating Our Own Cooking: WordPress

The reference server proves the architecture, but it's a greenfield implementation. The web's installed base is a different problem — and roughly 43% of it is WordPress.

So we built a WordPress plugin. [MDF Analytics](https://github.com/bitcryptic-gw/mdf-analytics-wp) is Phase 1 of a planned full integration: a single-file, dependency-free plugin that passively classifies and logs agent traffic — known AI agents by user-agent fingerprint, likely-automated clients by the absence of browser markers, and crucially, any request that sends `Accept: text/markdown`. No IP addresses stored, 90-day retention, MIT licensed.

It also serves a curated `llms.txt` at the site root — with proper conditional request handling, because if you're going to preach efficient agent serving, your own llms.txt had better return a `304` when nothing's changed.

We've had it running on our own sites. The early numbers tell the story better than the proposal did:

In the first days of logging on a small personal site, **over 900 requests came from known automated clients**. The number that asked for markdown? **One.** And it was our own test.

That's the gap, measured. Agent traffic is already the majority of requests on an ordinary site, all of it paying the HTML tax, none of it signalling — because there's currently nothing to signal *to*. Phase 2 of the plugin adds the answer: wallet integration and real 402 responses, turning that dashboard's "estimated missed" column into actual earnings. Phase 3 generates the markdown from WordPress content automatically — one toggle from analytics to full MDF participation.

## Who Should Consider This

Two weeks of building and community discussion have sharpened our view of who MDF is actually for right now:

**Documentation and technical content sites** are the natural first adopters. The content is usually markdown-native already, the audience is disproportionately agentic, and serving clean markdown is nearly free to implement. Even at $0.00 across the board, the token savings to your consumers are real and the `/mdf.json` capability document costs you nothing.

**Independent publishers and bloggers** are who the payment tiers exist for. If most of your traffic is now automated and none of it compensates you, a one-satoshi price on agent fetches is not a paywall — it's the smallest possible assertion that your work has a price. The Lightning rail makes that economically sane at any scale.

**Operators of private or internal knowledge bases** should look at the high-price tier, where payment doubles as authentication. Pay, get a scoped bearer token, no OAuth dance, no API key lifecycle. It's a genuinely different access model and it's fully implemented in the reference server today.

**Agent builders** are the other side of the market. If you're paying the HTML tax at scale, MDF-compliant endpoints are 80% cheaper to consume. Supporting `Accept: text/markdown` plus the 402 flow costs you a few hundred lines and positions you for a web where the best content increasingly sits behind a one-satoshi handshake.

If your site is heavily interactive, app-like, or its value lives in the presentation layer rather than the prose — MDF has little to offer you yet, and we'd rather say that plainly than oversell.

## Try It

Everything is live, open, and self-hostable:

```bash
# The full loop, with real Lightning payment
curl -H "Accept: text/markdown" https://mdf-demo.bitcryptic.com/micropayment/intro

# Validate any MDF deployment
# github.com/bitcryptic-gw/mdf-validator

# Measure agent traffic on your WordPress site
# github.com/bitcryptic-gw/mdf-analytics-wp
```

The proposal remains at [github.com/bitcryptic-gw/mdf](https://github.com/bitcryptic-gw/mdf), the open questions remain open, and the issues tab remains the front door. The most valuable contributions so far have come from people telling us what we got wrong. More of that, please.

---

*MDF is a community proposal by Gary Walker ([BitCryptic™](https://bitcryptic.com)) and Graham Hall ([Slepner](https://slepner.com.au)). It is not affiliated with or endorsed by Cloudflare, Anthropic, Answer.AI, Alby, or any other organisation mentioned herein.*
