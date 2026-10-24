+++
title = "Scree vocabulary: teaching search what a name looks like"
date = 2026-10-24
draft = false
[taxonomies]
tags = ["scree","mcp","embeddings","search"]
+++

So the original [scree](@/posts/scree.md) was good at finding lines in the docs that related to the embeddings of a search phrase. I added SQLite FTS5 for keyword search. It still didn't result in implementers using scree -- they still reach for `grep` (it's built in, and it shows more around each match). So it was back to the drawing board: time to go look at the code files, not just Markdown.

## Two kinds of search

Semantic search is what scree started with: embed the question, then find the chunks whose embeddings are closest. It's good at meaning -- ask "how do we log in" and it finds the section titled "Authentication", even though they don't share a single word.

Keyword search is the opposite: exact words, no meaning. That's what you want when you're looking for a function name or a config key. SQLite has it built in as FTS5, so that's what scree uses.

Hybrid runs both and merges the two lists with reciprocal rank fusion -- anything that ranks high in either list floats to the top.

## Why not always use both

The easy answer would be: always run hybrid. So I tested it on 16 plain-language questions and 7 identifier queries over my real notes. The score is MRR (mean reciprocal rank): roughly, how high the right file lands. 1.0 means it's first every time.

| mode | plain questions | identifiers |
|------|-----------------|-------------|
| semantic only | 0.89 | 0.46 |
| hybrid, always | 0.76 | ~0.80 |
| auto | 0.89 | 0.80 |

Always-on hybrid fixed the names, but plain questions got worse -- the keyword side drags in chunks that share words but not meaning. Turning the keyword side down to half weight only got plain questions back to 0.83. So scree picks per query instead: that's `auto`, and it's the default.

## How `auto` decides today

Right now it's a pretty simple rule: if a word has a letter plus an `_` or a digit, it's probably a name, and the query goes to hybrid. Otherwise it's semantic.

- counts: `PURGE_OLD_RECORDS`, `psycopg_pool`, `R7-Q2`, `v2`
- doesn't count: `ID`, `API`, `2026`, `well-known`

That's what got identifiers from 0.46 to 0.80 without touching plain questions.

## Where it falls short

`psycopg`. No underscore, no digit -- to the rule it looks like any other word, so it goes to semantic search, which has no idea it's a library. Same with a type name like `PageFetcher`. The rule can't tell a name from a word just by looking at it.

## The narrow vocabulary

So instead of guessing what a name looks like, ask the repo what names it actually has. On every sync, scree collects them locally from:

- dependency manifests: `requirements*.txt`, `pyproject.toml`, `package.json`, `go.mod`, `Cargo.toml`
- mixed-case names (`PageFetcher`, `useState`) from imports and declarations in Python, JavaScript, TypeScript and Go

Only mixed-case names from the code, on purpose: a function called `run` or a variable called `text` is just a word, and it shouldn't send "how do I run the tests" to hybrid.

And only from name positions. `const PageFetcher = ...` gives `PageFetcher`, not whatever's on the right of the `=`. Strings, comments and Python docstrings get skipped, so a value written in code never turns into a "name". It's line-based regexes, not a real parser, so it can still misread something -- but the worst a misread can do is send a question to hybrid. It never gets printed or embedded.

If a word in the query is on the list, it's a name, and the query goes to hybrid. It doesn't matter how often it shows up -- just whether it's there. `PageFetcher's` counts as `PageFetcher` too.

It's narrow on purpose: it only knows this repo's names, nothing else. It's built on your machine, follows the same ignore rules as the rest of the index, skips hidden folders like `.claude/` and vendored folders like `node_modules`, and only keeps the lowercased names. The code itself never gets chunked, embedded or sent to OpenRouter.

There was a way around that, of course: a symlink like `notes.md -> secret.py` would have sent code to the embedder under a note's name. Now a link has to stay in its own lane -- markdown points to markdown, code to code -- and anything that isn't markdown never reaches the embedder.

`scree doctor` reports how many names it found and how many of those the notes also mention, and `-v` shows a sample -- but only of names the notes already contain. A name that's only in the code never gets printed.

## Keeping ordinary words out

The catch: a lot of package names are just words. `sync`, `requests`, `next`, `config` -- if those counted, half of all plain questions would get routed to hybrid, and we're back to the 0.76 problem.

So there's a stoplist of about 350 words:

- question and function words: `how`, `what`, `the`, `with`, `does`
- package names that are ordinary words: `sync`, `text`, `http`, `json`, `requests`, `config`, `utils`
- plain words that code writes in mixed case: `setUp`, `tearDown`, `fileName`, `userName`

Go modules got their own rule. `golang.org/x/sync` only adds its full path, because its last part is just `sync`.

It's not perfect -- a word the stoplist misses can still send a plain question to hybrid.

## What I ruled out

- **Rare words from the keyword index.** The first idea was: if a word is rare in the docs, it's probably a name. But library names like `psycopg` show up all over a repo's docs, so a "rare word" signal misses exactly the names you want.
- **A language server.** `workspace/symbol` would be the full version of the same idea, but it means a process per language. Claude Code has its own LSP support now, so that might make it unnecessary anyway.
- **Only counting names the notes mention.** This one I actually built, then took back out. The logic seemed sound: keyword search can only find words that are in the notes, so why route on a name the notes never use? But hybrid's keyword side matches *every* word in the query, not just the name, so a code question still gains from it. On a repo whose docs rarely name identifiers, the gate cut the code questions going to hybrid from 12 of 15 to 2 of 15. So routing goes on the vocabulary alone, and the gate only lives on in `doctor`'s sample.

## Results

The short version: scree reached the answer with a median of 10–20 KB of context. grep needed anywhere from 46 KB to 657 KB.

I measured it on two repos with both code and docs: `test-repo`, a FastAPI backend with lots of documentation for future work, and scree's own repo. My notes (`~/notes`) are in there too, as the docs-only set. Routing on the vocabulary gave a modest gain on questions that name something in the code, and -- the part I cared about most -- it didn't send a single plain question to hybrid. That took the stoplist and the Go full-path rule, though.

### Accuracy: file MRR (1.0 = right file always ranked first)

```text
                            0         0.25       0.5        0.75       1.0
                            |----------|----------|----------|----------|
~/notes, plain (16)  scree  ██████████████████████████████████▋        0.891
                     grep   ████████████▉                              0.322
test-repo, plain (8) scree  ████████████████████████████████████████   1.000
                     grep   ████████████████▋                          0.417
test-repo, code (15) scree  ██████████████████████████████████▋        0.867
                     grep   █████████████████████████▊                 0.644
~/notes, idents (7)  scree  ███████████████████████████████▉           0.798
                     grep   █████████████████████████████████          0.827
scree, code (15)     scree  █████████████████████████████████▎         0.833
                     grep   █████████████████████████████████████▎     0.933
scree, plain (8)     scree  ███████████████████████████████████        0.875
                     grep   ███████████████████████████████████        0.875
```
scree = v0.2.0 `auto`; grep = the better of the two grep strategies.

### Context used to reach the answer (median KB, lower is better)

```text
                            0         200       400       600       800 KB
                            |---------|---------|---------|---------|
~/notes, plain       scree  ▌                                         10.5
                     grep   █████████████████████████████▉            598
test-repo, plain     scree  ▊                                         15.0
                     grep   ████████████████████████████████▉         657
test-repo, code      scree  █                                         19.7
                     grep   █████▎                                    106
~/notes, idents      scree  ▊                                         14.7
                     grep   ███████████████████████                   460
scree, code          scree  ▉                                         18.5
                     grep   ███▉                                      78.4
scree, plain         scree  ▊                                         15.4
                     grep   ██▍                                       46.5
```
grep = whichever strategy used less context. Roughly 4 KB ≈ 1k tokens.

### v0.1.0 vs v0.2.0 `auto` (file MRR)

```text
                            0.7      0.8      0.9      1.0
                            |--------|--------|--------|
test-repo, code      v0.1.0 ███████████▌                 0.828
                     v0.2.0 ███████████████              0.867
scree, code          v0.1.0 █████████                    0.800
                     v0.2.0 ████████████                 0.833
~/notes, plain       both   █████████████████▎           0.891
~/notes, idents      both   ████████▉                    0.798
test-repo, plain     both   ███████████████████████████  1.000
scree, plain         both   ███████████████▊             0.875
```
Note: this chart's axis starts at 0.7, so the differences look bigger than they are.

| | scree | grep |
|---|---|---|
| Time per query | ~0.55 s (embedding API call) | < 0.03 s |
| Plain questions, right file first | 14/16 (~/notes), 8/8 (test-repo) | 3/16, 2/8 |

Modest, but it's the right direction, and it doesn't cost the plain questions anything.

grep isn't beaten everywhere, though. On exact names it still edges out scree: 0.827 vs 0.798 on my notes' identifier questions, and 0.933 vs 0.833 on code questions about scree's own repo. But it gets there by reading about 30 times as much context on the notes and 4 times as much on the repo -- and that's the context scree is trying to save.

I've also turned on a hook in one repo to track when agents use scree, and whether it actually helps them.

## Open questions

- Keyword search (BM25) already discounts words that show up everywhere. So how much of the gain is the routing, and how much would semantic search have found anyway?
- Scree reads code now, even if it's just names. Does that change how it should describe what it indexes?
- Did it fix the original problem -- are agents actually using scree now? That's what the hook is for.

Agents may not be using scree much yet -- but when it's used, the context is significantly lower.

It's merged into main now, and shipped in [v0.2.0](https://github.com/jrapakko/scree/releases/tag/v0.2.0). I'll update this once I've got more numbers.
