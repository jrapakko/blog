+++
title = "Workout app: two years in"
date = 2026-10-24
draft = true
[taxonomies]
tags = ["workout-app","angular","spring","claude"]
+++

<!-- TODO: one or two lines. What the app is (a workout tracker: build
workouts, put them in a routine, log sets) and why you built your own
instead of using an existing app. Link the repo:
https://github.com/jrapakko/workout-app -->

## Where it started (2024)

<!-- TODO: the first version. From the git history:
- May–Jun 2024: initial commit, drag-drop workout cards, regimen component,
  input mocks with no real functionality yet
- Aug 2024 (42 commits): Spring REST backend, ExerciseSet, regimen saving,
  workout/regimen join table, login stub
What were you trying to solve, and what did it look like then? -->

## The gap, and coming back (2025)

<!-- TODO: about a year with no commits. Then Jul–Aug 2025 (~80 commits):
- Angular 18 -> 20 upgrade, standalone components, new @if/@for control flow
- Keycloak + OAuth2, Spring Security, CORS
- multi-user support, logout, user name in navbar
- the first real test suite: integration tests for workouts and regimens
Why did you come back to it? -->

## Making it real (May 2026)

<!-- TODO: the push to actually deploy it (54 commits):
- Angular 21, swapped ng-bootstrap for Angular Material, new styling
- Keycloak in prod mode, nginx frontend build, compose stack
- /api/regimen/edit to load the routine page in one round-trip
- "code smell cleanup" passes, a few fixes credited "ty claude"
- empty states instead of 404s ("so user not scared when no workouts")
This is where Claude Code starts showing up. How did that change how you
worked on it? -->

## Native apps

<!-- TODO: the SwiftUI iOS port and the Android app (May 2026), both
talking to the same Spring backend with Keycloak login. How far along
are they, and what was porting like? Skip this section if they're not
ready to talk about. -->

## What's new (October 2026)

<!-- TODO: the latest additions:
- monorepo: backend and frontend in one repo, with an OpenAPI contract
  that generates the frontend types
- Flyway migrations, tested against real MariaDB
- workout history: session log and per-exercise progress
- personal records and estimated one-rep max
- weekly volume trends
- consistency heatmap, streaks and rotation adherence
- next-weight suggestions on the dashboard
- reviewer agents for changes
Screenshots of the history page and heatmap would help a lot here. -->

## Lessons

<!-- TODO: what two years on and off taught you. Coming back to old code,
upgrading Angular across several major versions, what you'd do
differently. -->

## What's next

<!-- TODO: what you're adding next, and whether other people can use it. -->
