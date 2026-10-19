---
name: work-on-issue
description: End-to-end workflow for picking up a GitHub issue or a Jira work item. First checks whether earlier work already fixed it; if so, drafts a Product update and closes it. If not, creates a linked branch, implements and tests the fix, and opens a PR for review. Out-of-scope findings become new issues. Use whenever the user says "let's work on #123", "let's work on <github issue url>", "work on issue 57", "pick up #186", "do M2.3", "take a look at issue …", "start on …", "work on WEB-123", or otherwise hands over an issue number, a Jira key or a URL to be worked, even without the words "issue" or "GitHub".
---

# Work on an issue

The user hands you an issue (a number like `186` / `#186`, a slice like `M2.3`, or a URL like `https://github.com/org/repo/issues/186`). Your job runs in this order:

1. Understand the issue.
2. Check whether earlier work already resolved it.
3. **If it's resolved:** show the user a Product update. Once they approve it, post it and close the issue.
4. **If it's not resolved:** create a branch linked to the issue, do the work, and open a PR for review.
5. Throughout: anything real you notice that is *not* part of this change becomes its own new issue.

**Jira:** if the instructions file says the repo tracks issues in Jira, or you're handed a Jira key (`WEB-123`) or an `atlassian.net` URL, follow this file with the commands in [jira.md](jira.md). Branches and PRs still go to GitHub.

The two failure modes to avoid are **closing an issue that isn't actually fixed** and **letting a PR sprawl** into unrelated fixes. Most of the steps below exist to prevent one of those two.

Write issue and PR bodies to a file with your file-writing tool and pass `--body-file`. Don't use shell heredocs: the same steps must work on macOS, Linux and Windows.

## 0. Get oriented

- Resolve the issue reference. With a URL, take `owner/repo` and the number from it, and pass `-R owner/repo` to `gh` if it isn't the current checkout's repo. If the current directory isn't a checkout of that repo, tell the user and stop. Don't work in the wrong repo. With a slice like `M2.3`, find it with `gh issue list --search "M2.3 in:title" --state all`.
- If `gh` answers 404 for a repo you know exists, the wrong account is probably active: check `gh auth status` and ask the user to `gh auth switch`. Never read tokens yourself.
- Run `git status`. If the working tree has uncommitted changes, or you're on a branch other than the default branch, ask the user what to do before going further. Never stash, reset or discard their work yourself.
- Read the repo's instructions file (`AGENTS.md`, or `CLAUDE.md`/`QWEN.md` if that's what it has) (and any file it references) if it isn't already in context. It defines the conventions everything below must follow: test commands, architecture rules, invariants and anything that "must also be updated".
- Learn the house style from recent history. One call covers it:
  ```bash
  gh pr list --state merged --limit 5 --json number,title,headRefName,body
  ```
  Note the branch naming (e.g. `wcolon/125-local-time-zone`), the PR title format (e.g. `Show dates in the user's time zone (#125)`) and the PR body sections. Match them. With no history yet, use `<github-login>/<N>-<slug>` (`gh api user --jq .login`) and the PR body below.

## 1. Review the issue

```bash
gh issue view <N> --json number,title,body,state,stateReason,labels,milestone,author,createdAt,comments,url
```

- If it's already closed, say so and ask whether to reopen or leave it. Don't redo closed work.
- Read **all** the comments, not just the body. Later comments often narrow the scope, record a decision, or say that something was partly fixed.
- If it belongs to a milestone, read the milestone's description (its goal and acceptance criteria) and the slice's entry in the plan file (`docs/PLAN.md` or `PLAN.md`), if the repo has one. Check that the issues it `Depends on` are closed; if one isn't, tell the user before starting.
- Look at any screenshots or attachments that are referenced.
- Work out precisely what "done" means: the issue's acceptance checkboxes are the minimum. If the issue is a question or lists options under "Decision needed", check whether the comments already answered it. If they didn't, the decision belongs to the user. Present the options with your recommendation and wait, rather than picking one and coding it.

## 2. Was it already resolved?

Issues are often fixed as a side effect of other work, and the reporter is never told. Search before you code:

```bash
# PRs and issues that mention it
gh search prs  --repo <owner/repo> "<N>" --json number,title,state,closedAt,url
gh search issues --repo <owner/repo> "<N>" --json number,title,state,url
# Commits that mention it, and everything since it was filed that touched the relevant code
git log --oneline --grep='#<N>[^0-9]' --grep='#<N>$'
git log --oneline --since=<createdAt> -- <files the issue concerns>
# Related later work, searched by the issue's key words
gh search issues --repo <owner/repo> "<key words>" --state closed --json number,title,closedAt,url
```

A mention isn't proof. **Verify against the current code on the default branch.** Find the code path the issue describes and read what it does now. Where you can, reproduce the issue's own example: run the relevant tests, or write a quick check. Then reach one of these verdicts:

| Verdict | What it means | Next step |
|---|---|---|
| **Resolved** | Current code clearly does what the issue asks, and you can name the PR/commit that did it | Go to step 3 |
| **Partly resolved** | Some of it was fixed by other work, the rest remains | Go to step 4; the PR says which part was already done and by what |
| **Not resolved** | The behaviour is still there | Go to step 4 |
| **Can't tell from code** | The issue needs real data, a browser or the live app to confirm | Tell the user what you checked and what still needs checking; ask how to proceed |

If the issue duplicates another one that is still **open**, that isn't "resolved". Tell the user and let them choose which one to keep.

## 3. Resolved: post a Product update and close

The person who filed the issue is often not a developer. Write so that they can tell the fix happened and how to see it, without reading code. Draft it, **show it to the user with the evidence** (PR/commit and what you verified), and post only after they approve:

```markdown
**Product update:** this is fixed by #<PR> (<short PR title>), merged <Month D, YYYY>.

**What was wrong:** <one or two plain sentences: what the reporter saw and why>

**What changed:** <what it does now, in the reporter's terms: page names, button names, the numbers they'd see>

**How to check:** <where to go in the app and what to look for; use the issue's own example if it had one>

If you still see the old behaviour, reopen this issue and add a screenshot.
```

Leave out sections that add nothing (a typo fix needs no "How to check"). Prefer page and button names to file paths. Mention code only when the reporter is a developer.

After approval:
```bash
gh issue comment <N> --body-file <file>
gh issue close <N> --reason completed
```
Report the issue URL back. Then stop. There's no branch or PR in this path.

## 4. Not resolved: branch, build, PR

### Branch linked to the issue
Create the branch with `gh issue develop` so it appears under the issue's **Development** section. A plain `git checkout -b` doesn't link it:

```bash
git checkout <default-branch>
git pull --ff-only
gh issue develop <N> --base <default-branch> --name <prefix>/<N>-<short-slug> --checkout
```
Take the prefix from the repo's existing branch names (e.g. `wcolon/`). Make the slug a few words describing the fix.

### Plan, when it's more than a small fix
For a slice that touches several files or layers, or leaves choices open, write a short plan first (files to change, tests to add, choices and your recommendation) and get the user's OK before editing. Use your tool's plan mode if it has one. A one-file fix doesn't need a plan.

### Do the work
- If `pstack` is installed, its `poteto-mode` playbooks may drive this step, and `ponytail` keeps the diff to the smallest complete change. They shape how the code is written. This skill still decides how it reaches GitHub: when a playbook's own branch or PR step differs, follow this skill's conventions and gates.
- Keep the change scoped to the issue. When you notice something else, write it down for step 5 and don't fix it here, unless leaving it would make this change wrong.
- Follow the instructions file to the letter. Rules like "a new route must also go in X" or "adding an error kind means updating these four places" are why repo tests exist. Never weaken an invariant it lists to make a change easier; if one is in the way, stop and ask.
- Add or update tests for both the positive and the negative cases. Every frontend change gets tests that validate it.
- Run the repo's checks (format, lint, typecheck, tests), plus the build if the change could affect it. Before you claim anything works, they must pass. If something fails and you can't fix it, say so plainly in the PR and in your report. When CI minutes are limited, run the checks locally and say so in the PR.
- When the change adds a command, a component, a route, a convention or an invariant, update the instructions file in the same PR so the next session knows. Keep its style: short, factual, no history.
- Before committing, review your own diff (`git diff <default-branch>...`) for correctness, leftover debugging and scope creep. For a larger change, run the `code-review` skill on it if your tool has one (Claude Code: `/code-review high`); otherwise reread the diff once more looking only for bugs: edge cases, error paths, inputs from outside. Fix what you confirm. Use the matching read-only reviewer agent: `rust-reviewer` for Rust changes, `ui-ux-reviewer` for UI changes, `qa-automation-engineer` for test coverage, and any reviewer the repo defines itself.

### Commit, push, open the PR
- Commit message in the repo's style, e.g. `<Imperative summary> (#<N>)`, plus whatever attribution lines the session requires.
- `git push -u origin HEAD`
- `gh pr create --title "<title>" --body-file <file>` with a title matching recent PRs and a body that follows their sections. With no house style yet, use the repo's `.github/pull_request_template.md`, or:
  ```markdown
  Closes #<N>.

  <One paragraph: what this adds and why, in plain words.>

  ## Choices made        (only when the issue left choices to the PR: each choice, what was picked and why)
  ## What was wrong
  ## What changed
  ## Where to look        (files, pages or commands a reviewer can open to see it)
  ## Tests                (what was added; what was run, where, and the result)
  ## Follow-ups filed     (issues from step 5, if any)
  ```
  If the issue was partly resolved, say up front which part earlier work fixed and cite it.
- Give the user the PR link. Don't merge it: a human reviews and merges (see the `review-pr` skill).

## 5. Findings outside the change → new issues

While reviewing and working, you'll notice real problems the issue doesn't cover: other bugs, inconsistencies, missing validation, stale docs. Don't fold them into this PR. They make it harder to review and risk regressions nobody asked for. List them for the user, then file each one as its own issue after their OK:

1. **Make sure it's real.** Reproduce it or point to the exact lines. Check that it exists on the default branch, and wasn't introduced by your change. If your change introduced it, it isn't a finding: fix it in the branch.
2. **Check for duplicates** with `gh search issues --repo <owner/repo> "<key words>" --state open`. If one exists, comment there with the new detail instead of filing again.
3. **File it** in the style of the repo's existing issues, using only labels that already exist (`gh label list`), plus `follow-up` if the repo has it, and in the current milestone only if it blocks that milestone's acceptance:
   ```markdown
   ## Problem
   <what happens, with file:line references and a concrete example (inputs → what you see)>

   ## Suggested fix        (or "## Decision needed" with numbered options when it's a product call)

   Found while working on #<N>. This is existing behaviour; #<N>'s change didn't introduce it.
   ```
   ```bash
   gh issue create --title "<symptom, stated plainly>" --body-file <file> --label <bug|enhancement|question> --label follow-up
   ```
4. List the new issue numbers in the PR's **Follow-ups filed** section.

Skip things that are pure style preferences, or that you can't substantiate.

## 6. Report back

End with a short summary for the user:
- The verdict from step 2, and why.
- Either the closed issue and its Product update link, **or** the branch and PR link, what changed, and the test results (failures included, verbatim).
- Any follow-up issues filed, with one line each.
- Anything that still needs a human: a decision, a browser check, a check on another OS, real-data QA.
- If the repo has a plan file, remind the user that `/flow:plan-milestones status` ticks the slice once the PR merges.
