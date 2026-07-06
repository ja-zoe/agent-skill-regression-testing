---
name: regression-testing
description: Architect and maintain a committed end-to-end regression suite that gates merges, so changing a shared seam can't silently break existing behavior. Owns the how of testing — a single runnable pass/fail command, deterministic seed/teardown fixtures, and an explicit per-project invariant catalog. Agnostic and self-contained: it depends on no other skill and plugs into any codebase; integration with a merge gate or a delivery pipeline is expressed as a functional contract, not a named dependency. Use when setting up testing on a project, adding or updating e2e specs, defining invariants, or when a change touches an auth/tenancy/routing/data-resolution seam and dependent behavior must be re-verified.
metadata:
  author: julian
  version: "0.1.0"
---

# Regression Testing

## Overview

This skill owns a project's **committed, re-runnable end-to-end regression suite** and the
discipline of keeping it honest. It exists because ad-hoc verification — write a script, run it
once, delete it — only proves the happy path of the thing you just built and never re-checks what
you might have broken elsewhere. The suite is the memory that ad-hoc testing lacks.

It is deliberately scoped to the **how of testing**, and it is **self-contained**: it names and
depends on no other skill or tool. Its one outward-facing contract is that it exposes a **single
command** (conventionally `test:e2e`) that exits zero when every invariant holds and non-zero
otherwise. Anything that wants to enforce or sequence testing calls that command; this skill asks
nothing of them in return.

## The invariant catalog (the core idea)

A regression suite is not "tests for each feature." It is a list of **invariants** — properties
that must hold no matter what changes — each backed by a spec. Every project declares its own
catalog in `e2e/INVARIANTS.md`. Tests assert invariants; they are not a re-description of features.

Write invariants as "given X, Y is always true," e.g.:
- *Each tenant surface renders its own data, branding, and theme — never another tenant's.*
- *An authenticated user is never shown the sign-in form; an unauthenticated user never sees app data.*
- *A gated feature is inaccessible below its plan and accessible at or above it.*

Invariants are phrased against observable behavior, so they survive refactors: they fail the moment
a property breaks, regardless of which feature the change was aimed at.

## The regression reflex (the rule that catches cross-feature breakage)

**When a change touches a SEAM — anything many features depend on (auth, tenancy/scoping, routing,
data/branding resolution, the data-access layer) — run the full suite and add an invariant for any
dependent property not yet covered.** A seam change is exactly where "I tested the feature I built"
fails, because the breakage lands in a *different* feature. This reflex is the point of the skill;
honor it over convenience.

## Architecture

```
<test config>            committed. headless, single worker, deterministic base URL(s).
e2e/
  INVARIANTS.md          the catalog: one line per invariant + which spec covers it.
  fixtures.<ext>         seed + teardown of known entities; every spec starts from a known state.
  <invariant>.spec.<ext> one spec per invariant (or a small cluster).
```

- **Determinism first.** Seed a fixed set of entities before the run; tear them down after. Never
  assert against data a human happened to create. Reuse a stable, non-interactive auth path rather
  than a live third-party flow.
- **Assert invariants, not implementation.** Prefer role/testid selectors and observable outcomes
  (URL, rendered attributes, visible content) over internal state.
- **One command.** Expose the suite as a single command (`test:e2e` by convention) runnable with no
  arguments, returning the right exit code. Keep it fast enough to gate a merge.
- **Tool-agnostic.** Any e2e runner works (Playwright is a fine default) as long as it reduces to
  one deterministic pass/fail command.
- **Resource discipline.** One server at a time; kill what you start; headless; single worker.

## Enforcement — the integration contract

Enforcement is expressed **functionally**, so this skill works with any project's tooling and names
none of it:

- **Provides:** one command `test:e2e` → exit `0` (all invariants hold) / non-zero (a regression),
  plus a human-readable `e2e/INVARIANTS.md` others can read.
- **Expects:** nothing. To make the suite a real gate rather than a suggestion, a project wires that
  command into whatever **blocks a merge** — a pre-merge hook, a CI check, a review step — so a red
  suite prevents the merge. To slot into a **delivery pipeline**, that pipeline runs the command in
  its verify step. Both are roles ("the thing that gates merges", "the thing that sequences work"):
  any tool filling a role plugs in; this skill references none by name.
- **Failure is blocking, not advisory.** A green suite is a merge precondition. A flaky test is a
  broken test — quarantine or fix it at once; a suite you distrust gets ignored.

## Boundaries — what this is NOT

- **Not a paper trail / spec system.** It doesn't own how work is planned, tracked, or logged.
- **Not an orchestrator.** It doesn't decide when to plan/build/ship.
- **Not unit tests.** This is the end-to-end safety net at the app boundary; lighter unit and
  integration tests are welcome but are a separate concern.
