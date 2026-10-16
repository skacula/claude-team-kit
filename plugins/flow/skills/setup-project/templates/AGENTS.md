# AGENTS.md

Instructions for AI coding agents (Claude Code, Qwen Code, Codex and others) working in this repository. Claude Code reads it through `CLAUDE.md`, which imports it.

## Commands

<!-- Every command a session needs, copy-pasteable, run from the repo root unless it says otherwise. Note prerequisites ("build the UI before the workspace build") and slow ones. -->

```sh
<install dependencies>
<format check>
<lint>
<typecheck>
<test everything>
<test one file / one test by name>
<run the app locally>
```

CI (`.github/workflows/<file>`) runs <what> on <which OSes>, for <which events>.

## Workflow

- Work is planned as GitHub milestones `M<m>: <name>` with issues `M<m>.<n>: <title>`; the narrative and status are in `docs/PLAN.md`. Research reports are in `docs/research/`.
- One issue, one branch, one PR: branch `<handle>/<issue>-<slug>` created with `gh issue develop`, PR body `Closes #<issue>`, title `<Imperative summary> (#<issue>)`.
- Every PR has tests for the positive and negative cases; every frontend change has tests. Run <checks> before pushing.
- A PR that adds a command, a component, a route, a convention or an invariant updates this file.
- Problems found outside the PR's scope become new issues labelled `follow-up`.

## Architecture

<!-- What the system is in two sentences, then one bullet per component: its path, its job, what it depends on, and the non-obvious rules. Name types and files; skip what the code makes obvious. -->

- `<path>`: <job>.

## Invariants (enforced in code and tests; preserve them)

<!-- Security, privacy and correctness rules that must never be weakened, each with the test that checks it. -->

- <rule>, checked by `<test>`.

## Conventions

<!-- Error handling across boundaries, naming, where types are mirrored and must be kept in sync, data flow. -->

## Testing patterns

<!-- How tests are written here: fixtures, fakes, what's mocked, where integration tests live. -->
