# bitcryptic-blog-content

Markdown content source for [blog.bitcryptic.com](https://blog.bitcryptic.com).

This repo is the source of truth for all blog posts. The Astro app at
[bitcryptic-gw/bitcryptic-blog](https://github.com/bitcryptic-gw/bitcryptic-blog)
mounts this directory at runtime and serves posts directly.

## Frontmatter schema

```yaml
---
title: "Post title"
slug: "url-slug"           # used as the URL path: /blog/{slug}
date: 2026-01-01           # publication date
updated: 2026-01-02        # optional, last updated date
author: "BitCryptic"       # defaults to BitCryptic if omitted
tags: ["tag1", "tag2"]     # optional
description: "..."         # required, used in index, RSS, and og:description
status: draft | published  # only published posts are served
---
```

## Adding a post

1. Create a new `.md` file in the repo root
2. Add frontmatter matching the schema above
3. Write content in standard markdown
4. Set `status: published` when ready to go live
5. Commit and push — the server picks it up within the next poll cycle

## Authoring via k2

k2 (NanoClaw agent) has commit access via deploy key.
Draft posts (`status: draft`) can be committed freely — they won't be served until
the status is flipped to `published`.
