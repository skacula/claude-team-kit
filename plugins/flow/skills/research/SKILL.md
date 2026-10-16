---
name: research
description: Deep research phase before any code or plan. Turns an idea, a design doc (PDF, Markdown, slides) or a technical question into a sourced research report with options compared, a recommendation, risks, and the decisions a human must make. Use when the user says "research X", "let's do the research phase", "compare options for …", "is X viable", "what should we use for …", hands over a design document to study, or starts a new project or milestone that has open technical questions. Does not create issues or write production code.
---

# Research

The output of this phase is a **report a teammate can read in ten minutes and act on**, plus a short list of **decisions the human makes**. The planning phase (`plan-milestones`) builds on it, so anything left vague here turns into a wrong milestone later.

Two failure modes to avoid:
- **Unsourced claims.** "X supports Windows" with no link, version or test is a guess. Every load-bearing claim needs a primary source or a check you ran.
- **Deciding for the human.** Choices about scope, cost, licensing and risk belong to the user. Present options with a recommendation and wait.

## 0. Frame it

- Read what you were handed **in full**: every page of a design doc, every comment on an issue. Read the repo's instructions file (`AGENTS.md`, or `CLAUDE.md`/`QWEN.md` if that's what it has) and any existing `docs/research/` and plan files, if there are any.
- Write down, and show the user in a few lines:
  - **Goal**: what the research must let us decide or build.
  - **Constraints**: OSes, languages, budget (including CI minutes), licence policy, security or privacy rules, deadlines.
  - **Non-goals**: what we are not deciding now.
  - **Questions**: the 3–8 concrete questions the report will answer.
- If the goal or the constraints are unclear, ask before researching. One round of questions here saves a wasted report.

## 1. Gather

For each question:
- **Prefer primary sources**: official docs, release notes, changelogs, the project's repository (recent commits, open issues, releases), licence files, standards. Use blog posts only to find primary sources.
- **Record versions and dates.** Facts about software change. Write "Lima 2.2.1 (released 2026-09-30)", not "Lima".
- **Check the licence of every candidate dependency, including its latest major version.** Licences change between majors (a UI library moving to a commercial licence in v5 is a real example). Note anything that isn't MIT, Apache-2.0, BSD or compatible with the project's licence.
- **Check every OS we ship on.** Note what is untested or unsupported per OS.
- **Check maintenance health**: last release, issue response, bus factor.
- **Check cost**: hosted service pricing, CI minutes (macOS and Windows runners usually bill at a multiple of Linux), hardware.
- **Read the existing code** when the question is about this repo. Cite `file:line`.

If you have no web search tool (some models and providers don't offer one), fetch the official pages you know directly, read repositories, releases and licences with `gh`, and tell the user which questions that left thin.

When several questions are independent and broad, you may hand each to a subagent in parallel, **but only if the user asked for that or agreed to the extra cost**. Give each subagent the goal, the constraints and its one question, and ask for sources with each claim.

## 2. Verify what matters

A claim that a decision rests on deserves a check you ran yourself, when it's cheap:
- Install it in a scratch directory and run the example.
- Write a 20-line spike that proves the API does what the docs say.
- Run the tool on each OS you can reach, and note the ones you couldn't.

Keep spikes out of the repo (use a scratch directory). Mention in the report what you ran and what it showed.

## 3. Write the report

Write it to `docs/research/<YYYY-MM-DD>-<short-slug>.md` in the repo, following [report-template.md](report-template.md). Rules:
- Lead with the **recommendation and why**, in a few sentences.
- Compare options in a table: one row per option, columns for the criteria that matter for *this* decision.
- Put each source next to the claim it supports, with the date you accessed it.
- **Decisions needed**: numbered, each with options, your recommendation and what changes if we choose otherwise.
- **Proposed fixed decisions** and **proposed invariants**: the rules every milestone must keep (for example "the API binds loopback only"). Once the user accepts them, the plan and the instructions file carry them, and nobody relitigates them.
- Leave out what doesn't help the decision.

Write files with your file-writing tool, not shell heredocs, so the same steps work on macOS, Linux and Windows.

## 4. Hand over

- Show the user the summary, the decisions needed and the report's path.
- Once the user answers, record the answers in the report's **Decisions** section with the date, and move the accepted ones to **Fixed decisions**.
- Ask whether to commit the report (on a branch, with a PR, following the repo's conventions) or keep it local.
- Suggest the next step: `/flow:plan-milestones docs/research/<file>.md`.

Do not create GitHub issues, milestones or production code in this phase.
