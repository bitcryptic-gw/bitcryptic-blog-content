---
title: "MDF 0.3: What's Changed, What We Got Wrong, and Where It Stands"
slug: "mdf-0-3-whats-changed"
date: 2026-11-15
description: "Markdown First is now on x402 v2 with Lightning and USDC both settling live in production. Here's what changed since our last post, and two corrections we owe readers."
author: "Gary Walker"
tags: ["MDF", "x402", "Lightning", "agents", "AEO"]
status: published
---

It's been five months since the last MDF post. In that time Markdown First went from a proposal with stubbed payment rails to a working spec with two payment rails settling real money in production. Along the way we found bugs that mean some of what we published earlier was wrong. This post covers where MDF is now, what changed, and the corrections, in that order.

If you're new here: MDF is a small open standard for serving web content to AI agents as markdown, and optionally charging for it per fetch. A site publishes `/mdf.json` describing what it offers and what each section costs. An agent asks for `text/markdown`, gets clean content for free sections, and gets an HTTP 402 with payment options for priced ones.

## Where MDF stands today

- **Spec 0.3.1**, **reference server 0.3.0** and **validator 0.3.0** are released and tagged.
- The demo at `mdf-demo.bitcryptic.com` runs 0.3.0 on two hosts behind a failover proxy, and passes all 56 checks in our open-source validator, covering markdown delivery and payment. You can run the same validator against your own site.
- **Both payment rails have settled live in production.** A 1-sat Lightning payment and a 0.01 USDC payment on Base mainnet both went through on 13 November. The USDC one was the demo's first mainnet settlement.
- **ProofLines (prooflines.org) is the first third-party site running MDF 0.3.x**, which is the point where a spec starts to mean something.

## What changed

**MDF now speaks x402 v2, and it was a hard cut.** In November Graham and I decided to drop x402 v1 entirely rather than run both. A priced request now returns a standard x402 v2 `PaymentRequired`, mirrored in the `PAYMENT-REQUIRED` header. MDF's own fields (price summary, source size, token estimate, expiry) sit under `extensions.mdf`, so any x402 v2 client can pay an MDF site without knowing anything about MDF.

**Lightning is a first-class x402 rail.** Rather than running Lightning as a separate protocol beside x402, a Lightning invoice is now just another entry in the 402's `accepts[]` list, using x402's `exact` scheme on the `lnbtc` network. L402, which our earlier posts described as the Lightning path, is now an optional compatibility challenge for older clients. An agent picks a rail; the server doesn't care which.

**Each rail is priced in its own units.** There's no exchange rate inside the server any more. The demo's ladder today:

| Section | Price |
|---|---|
| `/`, `/docs/` | free |
| `/micropayment/` | 1 sat (Lightning) |
| `/premium/` | 0.01 USDC (Base) |
| `/private/` | 0.10 USDC (Base), buys a 24-hour access token |
| `/testnet/` | 0.001 USDC (Base Sepolia) |
| `/showcase/` | 0.02 USDC **or** 20 sats, your choice |

The `/showcase/` route is the interesting one: the same content offered on both rails at once.

**Settlement is self-hosted.** USDC payments settle through our own x402 v2 facilitator, one instance per demo host, rather than a third party. Lightning invoices are minted over Nostr Wallet Connect with a connection that can create invoices and nothing else: it can't pay or send. Replay protection is shared between both hosts, so a payment can't be reused against the other host or after a restart.

## Corrections we owe you

**1. Lightning invoices were 1/1000th of the advertised price for nearly four months.** From 30 May to 20 September 2026, the reference server passed prices to our Lightning node in satoshis when the node expected millisatoshis. Every Lightning invoice it minted was a thousandth of the advertised amount. The 1-sat micropayment route was worse: a thousandth of a satoshi isn't a valid invoice, so that route returned a 402 with **no invoice at all** for the whole period.

That includes the example in our June post, which told you to "pay the one-satoshi invoice". It wasn't payable when we published it. The advertised prices never changed; the conversion was wrong. It was fixed in reference server 0.2.6, deployed and verified live on 20 September. No invoice or amount quoted from that period is evidence of what MDF costs. The current prices are the ones above, and the ones in the live `/mdf.json`.

**2. Two payment-verification bugs.** Our earlier posts described the payment flows as finished, so these need saying. Version 0.2.8 (2 October) removed a fallback that would have let someone reuse an already-settled payment to unlock a route. Version 0.2.9 (11 November) fixed the token endpoint, which would issue a 24-hour access token without any payment, so the token tier was effectively free to anyone who read `/mdf.json`. Both affected only the demo's own sample content. Both are fixed, with tests.

The two earlier MDF posts now carry a dated "kept for the record" banner and marked inline corrections. We haven't rewritten them, because they're accurate records of where the project was, apart from the errors now marked.

## Why markdown, and why now

Our original measurement found the same page cost about 5x more tokens as HTML than as markdown. That ratio is why MDF exists: if agents are going to read the web at scale, the cheap format should be the default one, and the people producing the content should have a way to be paid for it.

The rest of the industry has moved the same way. Cloudflare now converts pages to markdown at the edge for agents that ask with `Accept: text/markdown`, and coding agents send that header already. Yet when Cloudflare scanned the 200,000 most-visited domains this year, only 3.9% served markdown when asked. There's room for sites that do it properly, with markdown written at the source rather than converted at the edge.

To be clear about what MDF does and doesn't do for AI search: we're not claiming it gets you cited more by answer engines, because nothing shows that yet. What it does is make your content cheap and clean for agents to read, let you see how agents actually use your site, and give you a way to charge for the parts worth charging for, while the parts you want quoted stay free.

## Try it

Free content, as markdown:

```
curl -H "Accept: text/markdown" https://mdf-demo.bitcryptic.com/docs/getting-started
```

What the site offers and what it costs:

```
curl https://mdf-demo.bitcryptic.com/mdf.json
```

A priced route, which returns a 402 listing the Lightning payment option:

```
curl -i -H "Accept: text/markdown" https://mdf-demo.bitcryptic.com/micropayment/intro
```

Check your own site with the validator, or read the spec, at `github.com/bitcryptic-gw/mdf`. If you run a site and want to try MDF, or you build agents and want to pay for content properly, get in touch.

## What's next

We're cleaning up after the cutover (retiring the old v1 facilitator and the old images), then building payment support into `mdf_fetch`, our MCP tool, so agents can pay MDF sites directly. Next up after that is a proper home for MDF beyond this blog.
