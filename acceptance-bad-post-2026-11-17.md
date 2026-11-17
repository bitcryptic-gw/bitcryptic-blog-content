---
title: "Acceptance test — bad post (T2, must be rejected)"
slug: "acceptance-bad-post-2026-11-17"
date: "not-a-real-date"
author: "bitcryptic-bot"
description: "Automated acceptance test: this frontmatter is deliberately invalid and must NOT reach the live site."
tags: ["test"]
status: published
---

This post is deliberately invalid (the `date` cannot be parsed). It exists to
prove the sync loop refuses to publish a bad merge and alerts instead.
