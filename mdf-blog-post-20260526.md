---
title: "MDF — Markdown First: A Proposal for the Agent-Readable Web"
slug: "mdf-markdown-first"
date: 2026-05-26
author: "Gary Walker & Graham Hall"
description: "The web was built for human eyes. AI agents are paying an enormous tax for that. We think there's a better way — and we've written the proposal."
tags: ["ai", "open-source", "web-standards", "self-hosted", "bitcoin"]
status: published
---


If you run a website and you've been watching your server logs lately, you've probably noticed something: a significant and growing portion of your traffic isn't human. AI agents, crawlers, and automated pipelines are fetching your content constantly — and they're doing it the hard way.

They request your HTML page. They receive a document full of navigation menus, JavaScript bundles, cookie banners, advertising scaffolding, and layout markup. Then they laboriously strip all of that away to find the few hundred words of actual content they came for. Every single time.

This is wasteful by design, and nobody wins.

## The HTML Tax

Here's a number that puts it in perspective: a typical blog post fetched as HTML consumes roughly 5–10 times more tokens than the actual content it contains. One benchmark from Cloudflare's engineering team shows a single page consuming 16,180 tokens as HTML versus 3,150 as markdown — an 80% overhead, every fetch, at scale.

For large AI operators processing millions of pages daily, that overhead represents real money and real compute. For the content creators whose work is being consumed, it represents nothing — no attribution, no compensation, no signal that their content is even being used.

Meanwhile, here's the thing that always struck me as slightly absurd: most of that HTML was generated *from* markdown in the first place. Static site generators, documentation platforms, technical blogs — they almost all author in markdown and compile to HTML. Agents are paying a significant cost to reverse a transformation that didn't need to happen for them in the first place.

## What Already Exists (And Why It's Not Enough)

This problem hasn't gone unnoticed. A few partial solutions exist:

**llms.txt** — proposed by Jeremy Howard at Answer.AI in 2024 — gives site owners a way to publish a markdown index at `/llms.txt`, guiding agents to key resources. It's a good start, widely adopted by documentation-heavy sites like Stripe and Anthropic, but it's a discovery mechanism only. It doesn't change how content is served.

**HTTP content negotiation** has always technically allowed this: an agent sends `Accept: text/markdown` and a server that supports it can respond accordingly. Clean, standards-based, no new infrastructure. But it requires per-site implementation and there's no discovery layer, no payment mechanism, and no access policy system.

**Cloudflare's Markdown for Agents** (launched early 2026) productises content negotiation at the CDN edge — converting HTML to markdown in real time for any site behind Cloudflare. Impressive, and the token savings are real. But it's still conversion from HTML, the source of truth hasn't changed, and it's tied to one vendor's infrastructure.

None of these address the underlying architectural issue. And none of them give content creators any mechanism to be compensated for, or even signal their intent about, agent consumption of their work.

## Introducing MDF — Markdown First

Graham Hall and I have spent some time thinking about what a coherent end-to-end architecture would look like. We're calling it **MDF — Markdown First**, and we've published the proposal on GitHub.

The core idea has three parts.

**Markdown is the source of truth.** Not a conversion target, not a fallback — the canonical document. HTML is rendered from it dynamically for browsers. This is already how most modern site generators work internally; MDF just makes it the explicit contract with the outside world.

**Agents are first-class citizens.** MDF-compliant endpoints natively serve markdown via standard HTTP content negotiation at the same URL. No new protocol, no new port, no separate URL scheme — just `Accept: text/markdown` and a server that honours it. Alongside the content, a `/mdf.json` capability document tells agents everything they need to know: what's available, what it costs, and what they're allowed to do with it.

**Price is access policy.** This is the part we're most excited about. Rather than maintaining separate robots.txt directives, authentication layers, and paywalls as distinct systems, MDF unifies access control into a single price signal. The amount you set communicates your intent:

| Price | What it means |
|-------|---------------|
| `$0.00` | Open — serve immediately |
| `$0.0001` | Micropayment — offset infrastructure costs; still cheaper for AI operators than parsing HTML |
| `$1.00+` | Premium gated content |
| `$100.00+` | Private — payment triggers auth token issuance rather than content delivery |

That last tier is worth dwelling on. At high price points, the payment stops being a transaction and starts being a credential request. You pay, the server verifies, you receive a time-limited bearer token. Full authentication with no passwords, no OAuth, no API key management — payment is the identity proof. For operators running private knowledge bases or internal documentation, this is a genuinely different model.

The payment layer supports two rails. [x402](https://x402.org) handles EVM-compatible chains — USDC on Base is the natural first implementation, but any EVM chain with stablecoin support works. [L402](https://docs.lightning.engineering/the-lightning-network/l402) handles Bitcoin via the Lightning Network: sub-second, sub-cent micropayments with no on-chain settlement latency. L402 combines a Lightning invoice with a macaroon bearer credential, so payment and access token issuance happen in a single round trip. Both use the long-reserved `402 Payment Required` status code. MDF's payment flow is identical regardless of which rail executes it — sites advertise what they accept, agents use what they support.

## Content Freshness and Agent Subscriptions

Polling is as wasteful for agents as it was for RSS readers in 2003. MDF addresses this by extending standard RSS/Atom feeds with agent-semantic change metadata — an `mdf:change_type` field on each feed entry that tells agents *what kind* of change occurred, not just that something changed.

A `pricing_change` entry, for example, is immediately actionable for an agent's budget logic without re-fetching any content at all. A `retraction` tells an agent to remove something from its context. A `content_update` includes a significance score so agents can threshold out trivial edits.

For sites that want push rather than polling, MDF recommends [WebSub](https://www.w3.org/TR/websub/) (the W3C push standard, formerly PubSubHubbub) as the notification transport. Agents that support it receive real-time change notifications; those that don't fall back to feed polling gracefully.

## What MDF Is Not

Worth being direct about a few things:

It's not a new protocol. Everything runs over standard HTTP.

It's not a Cloudflare competitor or replacement. It's infrastructure-agnostic — a Caddy plugin, an Nginx module, a Bun middleware, or a CDN feature can all implement it.

It's not DRM. MDF can't prevent determined scrapers. What it does is create a standard economic incentive for compliant behaviour and an audit trail for non-compliant behaviour. Agents that ignore the payment signal self-identify.

And it's not trying to replace any of the existing standards it builds on. MDF's design philosophy is to act as a **connective layer** — linking llms.txt, HTTP content negotiation, x402, L402, RSS/Atom, and WebSub into a coherent architecture rather than replacing any of them. Implementors shouldn't need to abandon anything they already have.

## Where This Sits in the Broader Picture

For those who've been following [BitCryptic Compute](https://bitcryptic.com) — the crypto-native infrastructure marketplace we've been developing — MDF fits naturally as a concrete, deployable use case. Operators run MDF servers, earn micropayments for content served to agents, and BitCryptic Compute provides the payment routing and settlement layer. It positions the platform not just as an AI inference marketplace but as infrastructure for the emerging agent-readable web.

But MDF itself is a community proposal, not a BitCryptic product. We're publishing it openly, we're not trying to own it, and we think it's most valuable if the community builds on it independently of us.

## The Reference Implementation

The reference implementation is complete and live. `mdf-server` is a self-hostable Docker image deployed at **https://mdf-demo.bitcryptic.com** — you can hit it right now.

It serves markdown natively from a content directory, renders HTML dynamically for browser requests, auto-generates `/llms.txt` and `/mdf.json` from a single `mdf.yaml` config, and handles all three payment tiers end-to-end. Both x402 (EVM) and L402 (Lightning) payment rails are stubbed — structural validation is in place, on-chain and Lightning invoice verification are the next milestones. Bearer token issuance for high-price-tier access is fully implemented.

Try it:

```bash
# Discover capabilities
curl https://mdf-demo.bitcryptic.com/mdf.json

# Request markdown directly
curl -H "Accept: text/markdown" https://mdf-demo.bitcryptic.com/

# Free content
curl -H "Accept: text/markdown" https://mdf-demo.bitcryptic.com/docs/getting-started

# Paid content — returns 402 with payment instructions
curl -H "Accept: text/markdown" https://mdf-demo.bitcryptic.com/premium/deep-dive

# Private content — returns 402 with auth endpoint hint
curl https://mdf-demo.bitcryptic.com/private/internals
```

## Open Questions

We're being deliberately transparent about what isn't solved yet. The spec has a full open questions section, but two worth highlighting here:

**Payment rail standardisation** — both x402 and L402 are first-class rails in MDF, but recommending a default reduces friction for agent implementors who need to support something out of the box. We're seeking community input before committing to a recommendation.

**Update gaming** — any system that prices re-fetches creates an incentive to manipulate change frequency. We've identified the attack vectors and have candidate mitigations (time-window access, significance floors, feed-level subscriptions) but haven't committed to a canonical approach yet. This one benefits from implementation experience before it gets locked down.

## Get Involved

The full proposal is at [github.com/bitcryptic-gw/mdf](https://github.com/bitcryptic-gw/mdf). The concept document covers the complete architecture, the JSON Schema for `/mdf.json` is ready for implementors to validate against, and the open questions section is an explicit invitation to push back, extend, or challenge the ideas.

If you're building AI agents, running content sites, or working on web standards — we'd genuinely like to hear what we've got wrong. Open an issue.

---

*MDF is a community proposal by Gary Walker ([BitCryptic™](https://bitcryptic.com)) and Graham Hall ([Slepner](https://slepner.com.au)). It is not affiliated with or endorsed by Cloudflare, Anthropic, Answer.AI, or any other organisation mentioned herein.*
