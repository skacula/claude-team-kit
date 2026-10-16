---
name: review-pr
description: Review process for a pull request. As a reviewer, checks a PR against its issue's acceptance criteria and the invariants in the repo's AGENTS.md or CLAUDE.md, runs the checks, hunts bugs, verifies each finding, and drafts a review that is posted only after the user approves. As an author, works through review comments on your own PR, fixing and replying to each. Use when the user says "review PR 142", "review #142", "review this PR", "look at <PR url>", "address the review comments", "respond to the review on my PR", or asks whether a PR is ready to merge.
---

# Review a PR

Your review is **input to a human reviewer, never the approval**. A teammate approves and merges. The goal is that the human spends their time on judgement calls, because the checkable things were already checked.

Two failure modes to avoid:
- **Noise.** A review with twenty speculative nits buries the one real bug. Report only what you verified, ranked by severity.
- **Posting without the user's OK.** Review comments notify the author and the team. Draft first; post after approval.

Write review bodies to a file with your file-writing tool and pass `--body-file`, so the steps work on every OS.

Pick the mode: **reviewing someone's PR** (sections 1–5) or **addressing review comments on your own PR** (section 6).

## 1. Gather

```bash
gh pr view <N> --json number,title,body,author,baseRefName,headRefName,closingIssuesReferences,files,additions,deletions,reviewDecision,statusCheckRollup,url
gh pr diff <N>
gh pr checks <N>
```
- Read the linked issue(s) in full, with comments, and their milestone's acceptance criteria. That's the definition of done.
- Read the repo's instructions file (`AGENTS.md`, or `CLAUDE.md`/`QWEN.md` if that's what it has). Its invariants and "must also update" rules are the checklist for this repo.
- Read existing review comments (`gh api "repos/{owner}/{repo}/pulls/<N>/comments"` and `gh pr view <N> --comments`) so you don't repeat them.

To run anything, check the PR out. First run `git status`; if the tree has uncommitted changes, ask the user before switching branches. Then `gh pr checkout <N>`. For a long review, a separate git worktree keeps the user's checkout untouched.

## 2. Check

Go through these in order, and note evidence for each:

1. **Scope.** Does the diff do what the issue asks and no more? Unrelated changes belong in their own PR or a follow-up issue.
2. **Acceptance.** For each acceptance checkbox in the issue: met, not met, or can't tell, with the line, test or command output that shows it.
3. **Invariants.** Each rule in the instructions file that the diff touches is kept. Security rules first: auth on every route, input validation, secrets kept out of code and logs, pinned and verified dependencies.
4. **Tests.** New behaviour has tests for both the positive and the negative cases, and every frontend change has tests. Run the repo's checks (format, lint, typecheck, tests) and record the results. If CI already ran them on this commit, cite the run instead of re-running.
5. **Docs.** The instructions file is updated when the PR adds a command, component, route, convention or invariant; the README, when setup changed.
6. **Bugs.** If your tool has the `code-review` and `security-review` skills (Claude Code does), run `/code-review high <N>`, and `/security-review` when the diff touches auth, crypto, network exposure, parsing of untrusted input or permissions. Otherwise do that pass yourself: read every changed function whole, and for each ask what happens on empty, huge, malformed and hostile input, on every error path, and under concurrent use. Hand specialised parts to the read-only reviewer agents, in parallel when there's more than one: `rust-reviewer` when `.rs` files or `Cargo.toml` changed, `ui-ux-reviewer` when components, pages or styles changed, `qa-automation-engineer` for test gaps. Their findings still go through the verification below. Where installed, add `/ponytail-review` (is every line needed, is it safe), and for a risky or large branch `/pstack:thermos` (bug, security and maintainability audit), or `/pstack:interrogate` for a contested design.
7. **Try it.** When the change is user-visible and the repo has a way to run it, run it and look (the `ui-ux-reviewer` does this for UI). Say which OS you checked on, and which you couldn't.

**Verify every finding before reporting it**: reproduce it, write the failing test, or point to the exact lines and the input that breaks them. Drop what you can't substantiate. A problem that already exists on the base branch isn't this PR's fault: it becomes a follow-up issue (as in the `work-on-issue` skill, step 5), not a blocking comment.

## 3. Draft the review

```markdown
**Summary:** <one or two sentences: what the PR does and the overall verdict>

**Acceptance:** <each criterion: ✅ met (evidence) / ❌ not met / ❓ can't tell (what's needed)>

**Checks run:** <commands and results, or the CI run>

### Blocking
1. `<file>:<line>`: <problem>. <How to reproduce, or the input that breaks it.> <Suggested fix.>

### Should fix
1. …

### Nits (optional)
1. …

### Questions
1. …

**Follow-ups filed:** #<n> <title> (existing problems, not caused by this PR)
```

The verdict follows from the findings: **request changes** if anything is blocking, **comment** if there are only should-fix items or questions, **approve** only if the user is the reviewer and asks for it.

## 4. Show the user, then post

Show the draft and the verdict. After the user approves (and edits):
```bash
gh pr review <N> --comment --body-file <file>           # or --request-changes, or --approve
```
For line-by-line comments, the `code-review` skill can post its findings inline where it exists (`/code-review <level> <N> --comment`); otherwise put `file:line` in each finding of the review body.

Never merge, and never approve on the user's behalf unless they ask.

## 5. Report back

The PR link, the verdict, the blocking items in one line each, the checks run, and any follow-up issues filed. If you checked out the PR branch, say so, and offer to switch back.

## 6. Addressing review comments on your own PR

1. Gather every unresolved comment: `gh pr view <N> --comments` and `gh api "repos/{owner}/{repo}/pulls/<N>/comments"`.
2. List them for the user with what you plan for each: **fix**, **disagree** (with the reason), or **question for the reviewer**. A comment that asks for a product decision goes to the user.
3. After the user agrees, fix each in its own small commit (message names what it addresses), run the repo's checks, and push.
4. Draft a reply per comment (what changed and the commit, or why not) and post the replies once the user approves: `gh api -X POST "repos/{owner}/{repo}/pulls/<N>/comments/<comment-id>/replies" -F body=@<file>`. Leave resolving threads to the reviewer.
5. Requests that go beyond the PR's issue become follow-up issues, linked in your reply.
