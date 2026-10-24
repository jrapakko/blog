+++
title = "How I'm using Claude Code (so far)"
date = 2026-10-24
draft = true
[taxonomies]
tags = ["claude","workflow","agents"]
+++

<!-- TODO: one or two lines up front: you're new to this, this is what's
working for you a few weeks in, not a best-practices guide. Set the tone so
readers take it as notes, not advice. -->

## How it started

<!-- TODO: the progression, from the git history:
- May 2026, workout-app: Claude as a helper. Commit messages like "fixed
  dashboard regression ... -- ty claude" and "authguard now fixes a possible
  wire race ... (ty claude)". You still wrote most of it.
- Oct 16, a side project: first project built with a team of sub-agents from day
  one. 215 of 219 commits co-authored with Claude.
- Oct 23, scree: same pattern on a small tool, in a few hours (31 of 32).
- Oct 24, workout-app: brought the pattern back to your oldest project.
What made you go from "ask Claude to fix this" to setting up agents? -->

## The basic team

<!-- TODO: the four roles that show up in all three projects
(.claude/agents/):
- architect (Opus): plans and reviews designs, doesn't write code
- implementer (Sonnet or Opus): writes code to the plan, runs the tests
- verifier (Sonnet): checks the work against the spec, "does not fix
  anything"
- red-team (Opus): tries to break it
Why split it up instead of one agent doing everything? Good place for a
small diagram: architect -> implementer -> verifier / red-team -> back. -->

## Picking models per role

<!-- TODO: Opus where judgment matters (architect, red-team), Sonnet for
routine work and review, and in the side project a Haiku "implementer-lite" for
mechanical work like renames, boilerplate and fixtures. What did that do
to cost or speed? Did you get any of it wrong at first? -->

## Read-only reviewers

<!-- TODO: the reviewers and red-team only get Read, Grep, Glob and Bash,
no Edit or Write. Why that matters: a reviewer that can't "fix" things has
to report them, so you see what was wrong. -->

## Specialists

<!-- TODO: agents for one area. The side project has ui-designer and seo-marketing.
workout-app has a reviewer per area: backend, frontend, API contract, and
deploy config. When is a specialist worth adding, and when is it overkill? -->

## Checking the checkers

<!-- TODO: the finding-verifier in workout-app. Reviewers produce false
positives, so each finding goes to a skeptic that tries to refute it before
anything gets fixed. Did you see reviewers be confidently wrong? An
example would be good here. -->

## Workflows

<!-- TODO: workout-app's .claude/workflows/:
- review-change: one reviewer per area of the repo, then one skeptic per
  finding
- implement-feature: plan -> implement -> review -> fix confirmed findings
  once, and leave the changes uncommitted for you to look at
Why "once" and why uncommitted? Keeping yourself in the loop. -->

## Keeping context small

<!-- TODO: how you keep agents from drowning in context:
- the side project's CLAUDE.md calls itself a "short index": every agent loads it,
  so the detail lives in docs/ and ADRs, read only when needed
- a decisions table where nothing is final unless marked Decided, with
  who decided and when
- workout-app has a CLAUDE.md per app (backend/, frontend/)
- scree: the side project's agents have the mcp__scree tool to search the docs
  instead of reading them whole. Link the scree post:
  [scree](@/posts/scree.md) -->

## Letting the tools catch mistakes

<!-- TODO: workout-app generates the frontend's TypeScript types from the
backend's openapi.json, so a breaking API change shows up as a type error
instead of a runtime bug. Tests run Flyway migrations against real MariaDB.
Guardrails like these matter more when an agent writes the code. -->

## What I'm still figuring out

<!-- TODO: be honest about being new. Ideas: how much to review by hand,
cost, when the agent setup is too much for a small change, what you'd
tell yourself on day one. -->
