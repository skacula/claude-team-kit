---
name: plan-milestones
description: Milestone creation phase. Turns a research report, design doc or agreed scope into a plan of milestones (each with a goal and testable acceptance criteria) split into PR-sized issues, then, after the user approves the draft, creates the labels, GitHub milestones and issues (or Jira Epics and Stories) and keeps the plan file in sync with them. Also refreshes plan status from GitHub or Jira ("plan status", "sync the plan", "what's left in M3"). Use when the user says "plan milestones", "break this into issues", "create the milestones", "turn the research into a plan", "what's next", or after /flow:research finishes.
---

# Plan milestones

GitHub milestones and issues are the **shared source of truth** for what's planned and done. The plan file (`docs/PLAN.md` by default) is the narrative around them: fixed decisions, invariants, the status table and the order of work. Both must say the same thing.

Two failure modes to avoid:
- **Issues nobody can finish.** A slice is done when its acceptance boxes can be ticked by running something. "Improve performance" is not a slice.
- **Writing to GitHub before the user agreed.** Creating dozens of issues is visible to the whole team and tedious to undo. Draft first, create after approval.

**Jira:** if the instructions file says the repo tracks issues in Jira, or the user asks for the plan in Jira, follow [jira.md](jira.md) for sections 1, 4 and 5. Milestones become Epics and slices become Stories. Everything else here applies as written.

There are two modes: **create** (sections 1–4) and **status** (section 5). With no plan file and no milestones yet, it's create.

## 1. Read the inputs

- The research report(s) under `docs/research/`, the design doc, or what the user described. Read them in full.
- The repo's instructions file (`AGENTS.md`, or `CLAUDE.md`/`QWEN.md` if that's what it has), an existing plan file, and what GitHub already has. One call each:
  ```bash
  gh repo view --json nameWithOwner,defaultBranchRef
  gh api "repos/{owner}/{repo}/milestones?state=all&per_page=100" --jq '.[] | [.number, .title, .state, .open_issues, .closed_issues] | @tsv'
  gh label list --limit 200
  gh issue list --state all --limit 200 --json number,title,state,milestone,labels
  ```
- Fixed decisions and invariants already agreed are not up for debate. Build on them.

## 2. Draft the plan

Write the draft to `docs/PLAN.md` (or the path the repo already uses) following [plan-template.md](plan-template.md).

**Milestones** (`M0`, `M1`, …):
- `M0` is always the skeleton: the repo builds, tests and lints in CI, an instructions file (`AGENTS.md`) exists, and the thinnest end-to-end path works (one request through every layer). Everything else builds on it.
- Each milestone has a **goal** a user could observe, and **acceptance** criteria that can be checked by running something, per OS where that matters.
- Mark **gates**: milestones whose acceptance is a go/no-go for the next phase.
- Order by dependency, and say which milestones can run in parallel.

**Slices** (each becomes one issue and one PR):
- Title: `M<m>.<n>: <outcome, imperative>`, e.g. `M2.3: Reset a forgotten password`.
- Body: a goal sentence, a checklist of acceptance criteria (`- [ ] …`), `Depends on #…`, and an **Out of scope** line when the boundary isn't obvious. Use the repo's slice issue template (`.github/ISSUE_TEMPLATE/slice.md`) if it has one.
- Size it to be reviewable in one sitting. Prefer vertical slices (a thin feature through every layer) over horizontal ones ("all the types", then "all the endpoints").
- Name the decision when a slice can't start without one: put it under **Decision needed** with numbered options and your recommendation.

**Labels**: `area:<part>` (one per major component of the architecture), `os:<name>` when the project ships on several OSes, `follow-up` (found while delivering a slice), `phase-gate`, plus GitHub's defaults. Reuse labels that exist.

## 3. Review with the user

Show the user the milestone table and the slice titles per milestone, the labels you'll create, and anything under **Decision needed**. Then ask for approval. Apply their changes to the draft and show it again if the changes were large. **Create nothing on GitHub until they approve.**

## 4. Create on GitHub

Create in this order, skipping anything that already exists with the same title (so a re-run never duplicates):

1. **Labels**: `gh label create "<name>" --description "<what it covers>" --color <hex>`. Add `--force` only to update a label the user asked you to change.
2. **Milestones**: there's no `gh milestone` command, so use the API. Put the goal and the acceptance criteria in the description:
   ```bash
   gh api "repos/{owner}/{repo}/milestones" -f title="M1: <name>" -F description=@<file>
   ```
3. **Issues**, in milestone order, so that `Depends on #N` can name real numbers. The list from section 1 stops at 200 issues, so check each title right before creating it, and skip it if an issue with that title is returned:
   ```bash
   gh issue list --state all --search "\"M1.2:\" in:title" --json number,title,state
   ```
   Write each body to a file with your file-writing tool (not a heredoc, so it works on every OS), then:
   ```bash
   gh issue create --title "M1.2: <title>" --body-file <file> --milestone "M1: <name>" --label "area:<x>" --label "os:<y>"
   ```
4. Write the issue numbers back into the plan file next to each slice, and fill in the status table.
5. Ask whether to commit the plan file. On a team it should be committed (on a branch, with a PR) so everyone sees the same narrative. If the user keeps it local, add it to `.git/info/exclude`.

Report the milestone URLs and the number of issues created per milestone.

## 5. Status mode

When asked for status, or after a PR merges:
- Read the milestones and issues as in section 1, plus recently merged PRs: `gh pr list --state merged --limit 30 --json number,title,mergedAt,closingIssuesReferences`.
- In the plan file: tick slices whose issue is closed (with the PR number), update the status table (⬜ not started, 🟡 in progress, ✅ done), and add new issues that appeared in a milestone (follow-ups) under it.
- Flag drift both ways: an issue closed without its plan line ticked, a plan line with no issue (search its title as in section 4 before flagging it, since the list stops at 200), an issue whose milestone changed, an open issue whose acceptance boxes are all ticked.
- A milestone with no open issues left: check its acceptance criteria were actually demonstrated (a gate needs its end-to-end check run), then **ask** before closing it: `gh api -X PATCH "repos/{owner}/{repo}/milestones/<n>" -f state=closed`.
- Report: per milestone, done/total, what's in progress (open PRs), what's next, and anything blocked on a decision.
