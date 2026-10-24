+++
title = "Scree, or How I learned to stop fearing Claude Code"
date = 2026-10-24
[taxonomies]
tags = ["claude","scree","mcp","embeddings","vectordb"]
+++

[Scree](https://www.github.com/jrapakko/scree) is an MCP server that gives Claude Code semantic search over the ADRs and documented md files so that sub-agents don't have to read in long lines.

## The problem

I kept running into large context window on a project I was doing using sub-agents -- I thought to RAG, and was out for a walk: I decided to brainstorm an MCP server using OpenRouter embeddings and a vector db.

The context window kept growing close to the limit, and the md files kept growing larger and larger. So it was time for an edit to the md file structure. But could there be a better way?

## How it works

Scree indexes markdown files. It uses ~1600 character chunks. These get embedded as a feature vector in sqlite's vec1 vector-extension. The MCP tool then lets Claude ask scree to embed a question and search for similar vectors in the db, speeding up lookup over a large set of markdown files, and hopefully reducing context size as sub-agents can look up answers.

## Building it with Claude Code

As I've slowly started leveraging AI, I went to claude, built up a quick spec, and then had claude code take a crack at it when I got home.

Some hours later: with a sub-agent set, a model choice, OpenRouter loaded up (< $1 daily spend) openai/text-embedding-3-small -- and multiple passes (with red team, verification, and new implementation) scree is getting ready for use.

It started somewhat buggy, and not very secure:
```
High
- H1, symlinks: this is the same leak the verifier found, but worse. A planted link like key.md -> ~/.ssh/id_rsa gets indexed by the watcher with no user action. Its contents are sent to OpenRouter, stored in the index, and returned by search_notes. A note with prompt injection could also ask Claude to call reindex on such a link.

Medium
- M1, oversized results: search_notes has no size cap. One 5 MB line or an unclosed code fence produced an 11 MB result, flooding Claude's context.
- M2, forged results: note text and filenames can imitate scree's result headers. That lets injected text pose as scree output, and nothing marks note text as untrusted.
- M3, network exposure: serve -http 0.0.0.0:… gives the whole local network unauthenticated access to search, read and reindex.
- M4, slow API: a hung embeddings API blocks each call for about 3 minutes, which is 3 retries at a 60 s timeout. The watcher stalls for the whole time.
- M5, renamed folders: after a folder is renamed inside the notes tree, the watcher silently stops seeing changes in it.

Low
- L1: read_note can reach .git/config (and tokens in it) through a symlink inside the folder.
- L2: a 200 MB note crashes the process, which would make the systemd service restart in a loop.
- L3: terminal escape sequences in notes pass straight through to the CLI and logs.
- L4: error messages from the API are echoed verbatim, so a key could leak if a provider or proxy ever echoed it back.
- L5: index.db is readable by every user on the machine (mode 0644).
- L6: the watcher keeps running in a broken state if the database is deleted or the notes folder is moved away.
```

It allowed prompt injections, symlinking to SSH keys and all sorts of not great things.

But as it's moved forward, these have been fixed, and onto others:
```
H2, the wrapper scripts can run a command. They load the settings file by executing it as shell, and the README says to write values without quotes. A notes path containing a space (for example, Obsidian on iCloud: …/Mobile Documents/…) then runs part of the path as a command, which can resolve inside the repo Claude Code opened. The red-team demonstrated a cloned repo planting that executable. Your ~/notes path has no space, so you're not exposed, but anyone following the README with such a path would be.
```

## Numbers

I've set max daily spend at $1. Under my use, I should be using about 2 magnitudes less than that (maybe 1c/day) and memory usage ranges from idle 13~15mb to load of 29mb.

## What's next

I'll update with more info when it's not 3am.
