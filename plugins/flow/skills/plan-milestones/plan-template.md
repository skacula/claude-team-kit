# <Project> plan

Source: <research report(s) / design doc, with paths or links>. Repo: <owner/repo>. Last updated: <YYYY-MM-DD>.

**Workflow:** one PR per slice, branch `<handle>/<issue>-<slug>` off `<default branch>`, PR body `Closes #<issue>`.
**Tracking:** every slice below is a GitHub issue titled `M<m>.<n>: …` in its milestone (in Jira: a Story under its milestone's Epic in project `<KEY>`). Close the issue and tick the line here together.

## Status at a glance

| Milestone | What | Gate | Status |
|---|---|---|---|
| M0 | Skeleton: builds, tests, CI, AGENTS.md, thinnest end-to-end path | | ⬜ |
| M1 | <name> | | ⬜ |
| M2 | <name> | ✅ <gate name> | ⬜ |

Legend: ⬜ not started · 🟡 in progress · ✅ done.

## Fixed decisions (do not relitigate)

- <decision> (from <research report>, decided <YYYY-MM-DD>)

## Invariants (every milestone must keep these)

- <invariant>, checked by <test or CI step>

---

## ⬜ M0: Skeleton

Goal: <what a user or developer can observe when it's done>

Acceptance:
- [ ] <checkable by running something; per OS where it matters>

Slices:
1. ⬜ #<n> **M0.1: <title>**: <one line>
2. ⬜ #<n> **M0.2: <title>**: <one line>

## ⬜ M1: <name>

Goal: …

Depends on: M0. Can run in parallel with: —.

Acceptance:
- [ ] …

Slices:
1. ⬜ #<n> **M1.1: <title>**: <one line>

---

## Decisions needed

1. **<decision>** (blocks M<m>.<n>): options …, recommendation …

## Follow-ups not yet in a milestone

- #<n> <title>
