+++
title = "Does scree actually help?"
date = 2026-10-24
draft = true
[taxonomies]
tags = ["claude","scree","mcp","embeddings"]
+++

<!-- TODO: one or two lines. The question from the last post (link it with
[last post](@/posts/scree.md)) and the short answer, so readers who stop here
still get it. -->

## The question

<!-- TODO: what "help" means here. Less context per sub-agent? Fewer tokens
per task? Faster answers? Pick one or two things you can actually measure. -->

## Setup

<!-- TODO: the project and the notes it ran over (how many md files and
ADRs, total size). Which tasks you ran, and how many times each. What
"without scree" means: sub-agents reading files directly? -->

## How I measured

<!-- TODO: where the numbers come from (Claude Code's /cost or /context,
OpenRouter's dashboard, logs). Anything that makes the comparison less
than fair. -->

## Results

<!-- TODO: a small table works well here, e.g.

| task | without scree | with scree |
|------|---------------|------------|
|      |               |            |

Context used, tokens, cost, time, and whether the answer was right. -->

## What surprised me

<!-- TODO: where it helped more or less than expected. Misses, where search
returned the wrong chunk or the sub-agent read the whole file anyway. -->

## Cost

<!-- TODO: actual embedding spend for indexing plus queries, compared with
the $1/day cap and the ~1c/day guess from the last post. -->

## Verdict

<!-- TODO: keep using it, change it (chunk size? model?), or drop it. -->
