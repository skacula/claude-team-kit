# How we work with Claude Code

This guide is for engineers joining any of our projects. It covers setting up a machine once, setting up a repo once, and then the four phases every project goes through: **research → milestones → issues → review**. Each phase has a skill in the `flow` plugin that does the legwork. You make the decisions.

What you need installed is in [tools.md](tools.md). Everything here works the same on macOS, Linux and Windows, and with models other than Claude, such as Qwen or local models: see [other-models.md](other-models.md).

## The short version

| Phase | You type | You get | You decide |
|---|---|---|---|
| Set up a repo (once) | `/flow:setup-project` | `AGENTS.md`, shared settings, templates, labels, as a PR | What goes in `AGENTS.md`; which labels |
| 1. Research | `/flow:research <idea, doc or question>` | `docs/research/<date>-<slug>.md`: options compared, sources, risks, decisions needed | Every item under **Decisions needed** |
| 2. Milestones | `/flow:plan-milestones docs/research/<file>.md` | Draft `docs/PLAN.md`, then GitHub milestones, labels and `M<m>.<n>:` issues | Approve the draft before anything is created |
| 3. Issues | `/flow:work-on-issue #42` | A linked branch, the change with tests, a PR (or a Product update if it was already fixed) | Open choices; the plan for bigger slices |
| 4. Review | `/flow:review-pr 57` | A verified review draft: acceptance, checks run, blocking items | What gets posted; approval and merge are yours |
| Status | `/flow:plan-milestones status` | The plan file in step with GitHub, what's next | When to close a milestone |

Two rules run through all of it:
1. **Claude proposes, a human decides.** Nothing visible to the team (issues, milestones, review comments, merges) happens without your OK.
2. **Nothing is "done" without evidence.** A claim needs a source, a test, or a command you ran.

## The plugins around flow

`flow` owns the team process: the phases, the GitHub conventions and the human gates. Three plugins change how the agent works inside each step. We install all four, and `/flow:setup-project` enables them for everyone who trusts a set-up repo.

| Plugin | What it does | Commands you'll use |
|---|---|---|
| `ponytail` ([DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail), MIT) | Always on. Pushes every change toward the smallest complete solution, leaves a small test behind for real logic, and ends each reply with what was skipped and any risk. | `/ponytail lite\|full\|ultra`, `/ponytail-review`, `/ponytail-audit`, `/ponytail-debt` |
| `pstack` ([michael-denyer/pstack-claude](https://github.com/michael-denyer/pstack-claude), MIT) | Routes multi-file, design and debugging work through `poteto-mode` playbooks (Feature, Bug fix, Refactoring and more) and its principle skills. | `/pstack:how`, `/pstack:why`, `/pstack:architect`, `/pstack:tdd`, `/pstack:thermos`, `/pstack:interrogate`, `/pstack:babysit` |
| code-metrics (Anthropic plugin directory, MIT) | Measures complexity, long functions, large files and duplicated code, locally, with Python 3. | `/code-metrics-tracker:metrics`, `/code-metrics-tracker:refactor-targets` |

Where each one helps:

| Phase | flow skill | Helpers |
|---|---|---|
| Research | `/flow:research` | `/pstack:how` and `/pstack:why` for existing code; `/code-metrics-tracker:metrics` for a baseline |
| Milestones | `/flow:plan-milestones` | `/pstack:architect` for slices that cross module boundaries; `/code-metrics-tracker:refactor-targets` and `/ponytail-debt` to find tech-debt slices |
| Issues | `/flow:work-on-issue` | pstack's playbooks drive the build step; `/pstack:tdd` when a test comes first; ponytail keeps the diff small; `/pstack:deslop` before committing |
| Review | `/flow:review-pr` | `/ponytail-review`; `/pstack:thermos` for a risky or large branch; `/pstack:interrogate` for a contested design; `/pstack:babysit` to drive CI to green |
| Health check | none | `/ponytail-audit` and `/code-metrics-tracker:metrics`, now and then |

When a pstack playbook's own PR step differs from flow's, flow wins: branch names, `Closes #N`, the PR template, follow-up issues and the human gates.

## 1. Set up your machine (once)

1. Install the tools in [tools.md](tools.md): VS Code, Git, the GitHub CLI, Node.js LTS, and Claude Code (the CLI and the VS Code extension).
2. Sign in:
   - `claude`, then `/login` (use the team's Claude plan).
   - `gh auth login`. If you have several GitHub accounts, `gh auth switch -u <account>` picks the one that can see the repo. A `404` from `gh` on a repo you know exists usually means the wrong account is active.
3. Add the team marketplace and install the plugin. In any terminal (the VS Code integrated terminal is fine):
   ```sh
   claude plugin marketplace add skacula/claude-team-kit
   claude plugin install flow@claude-team-kit
   ```
   The VS Code extension and the CLI share the same configuration, so the plugin shows up in both. In a repo that's already set up you can skip this step: when you trust the folder, Claude Code offers to install it.
   Then install the paired plugins with the commands in the [README](../README.md#install). Run them in a terminal. The VS Code chat panel answers `/plugin` with "isn't available in this environment". ponytail's hooks need `node` on your PATH and code-metrics needs Python 3; both are in [tools.md](tools.md).
4. Optional, recommended: copy [templates/user-CLAUDE.md](../templates/user-CLAUDE.md) to `~/.claude/CLAUDE.md` (on Windows `%USERPROFILE%\.claude\CLAUDE.md`). Those are the engineering defaults Claude follows in every project. Edit it to taste; it's yours.
5. Check it: run `claude doctor`, then in a Claude session type `/flow:` and the five skills should be listed.

To update later: `claude plugin marketplace update claude-team-kit` and `claude plugin update flow@claude-team-kit`, then restart the session.

## 2. Set up a repo (once per repo)

Open the repo in VS Code, open the Claude Code panel and run `/flow:setup-project`. It looks at the repo first, tells you what it will add, and then opens a PR with:
- **`AGENTS.md`**, plus a one-line `CLAUDE.md` that imports it for Claude Code: the contract every session reads first, whatever the tool or model: commands, architecture, invariants, conventions, testing patterns. It's filled in from the repo itself, and each command is run to make sure it works.
- **`.claude/settings.json`**: enables this plugin for everyone who trusts the folder, pre-approves read-only `git`/`gh` commands, and stops Claude from reading `.env` files and keys.
- **`.vscode/extensions.json`**: the extensions for this repo's stack. VS Code offers them when you open the folder.
- **PR and issue templates**, the standard **labels** (`area:*`, `os:*`, `follow-up`, `phase-gate`), and `.gitignore` entries for personal files.

`AGENTS.md` is the most valuable file in the repo for this workflow. Keep it accurate: every PR that adds a command, component, route, convention or invariant updates it. Keep it short and factual, without history. If a session gets something wrong because `CLAUDE.md` was missing a rule, add the rule.

## 3. The workflow

### Phase 1: Deep research

**When:** a new project, a new milestone with open technical questions, or a "should we use X?" question.

**How:** `/flow:research` followed by what to research: a sentence, a path to a design doc (PDFs work), or an issue. For example:
```
/flow:research docs/design/overview.pdf — which local database should we use for offline sync on macOS, Linux and Windows?
```

**What happens:**
1. Claude restates the goal, constraints, non-goals and the 3–8 questions it will answer, and asks if anything is unclear. Correct it here; it's the cheapest point to do so.
2. It gathers facts from primary sources (official docs, release notes, repositories, licence files), recording versions and dates. It checks the licence of each candidate, **including its latest major version**, support on each OS we ship on, maintenance health and cost (CI minutes included).
3. It verifies load-bearing claims with small spikes in a scratch directory.
4. It writes the report to `docs/research/<date>-<slug>.md` and shows you the **Decisions needed**.

**Your part:** answer the decisions. They move into **Fixed decisions** and **Invariants**, which the plan and `AGENTS.md` carry from then on, so nobody re-argues them in every PR.

**Tips:**
- For a broad topic, ask for parallel research ("use subagents for each question"). It's faster but costs more tokens, so it's opt-in.
- Use a high-capability model and higher effort for research and planning (`/model`).
- Commit the report. It's the "why" behind the plan that the next engineer will want.
- On an existing codebase, `/pstack:how` explains how a part works and `/pstack:why` digs up why it was built that way. `/code-metrics-tracker:metrics` gives a baseline.

### Phase 2: Milestones and issues

**When:** after research, or when the scope is already clear.

**How:** `/flow:plan-milestones docs/research/<file>.md` (or describe the scope).

**What happens:**
1. Claude drafts `docs/PLAN.md`: a status table, fixed decisions, invariants, and milestones `M0`, `M1`, …, each with a **goal** and **acceptance criteria** you can check by running something.
   - `M0` is always the skeleton: it builds, tests and lints in CI, `AGENTS.md` exists, and the thinnest end-to-end path works.
   - Gates (a go/no-go for the next phase) are marked.
2. Each milestone is cut into **slices**, one issue and one PR each, titled `M<m>.<n>: <outcome>`, with an acceptance checklist, `Depends on #…`, and **Out of scope** where needed. Slices are vertical (thin and end to end) and reviewable in one sitting.
3. It shows you the draft. **Nothing goes to GitHub until you approve.**
4. Then it creates the labels, the milestones (goal and acceptance in the description) and the issues, writes the issue numbers back into the plan, and offers to commit the plan.

**Status:** after PRs merge, `/flow:plan-milestones status` ticks finished slices, updates the table, flags drift between the plan and GitHub, lists what's next, and asks before closing a finished milestone (after checking its acceptance was actually demonstrated).

**Source of truth:** GitHub milestones and issues. The plan file is the narrative around them. If they disagree, run status mode and fix it.

For a slice that crosses module boundaries, `/pstack:architect` sketches the types and modules before code. `/code-metrics-tracker:refactor-targets` and `/ponytail-debt` turn complexity hot spots and deferred `shortcut:` comments into tech-debt slices.

### Phase 3: Work on issues

**How:** `/flow:work-on-issue #42` (an issue URL or a slice name like `M2.3` also works). You can also just say "let's work on 42".

**What happens:**
1. **Orient:** it checks you're in the right repo on a clean default branch (if not, it asks; it never stashes or discards your work), reads `AGENTS.md` (or `CLAUDE.md`), and learns branch and PR conventions from recent merged PRs.
2. **Understand:** it reads the issue with all its comments, its milestone's acceptance criteria and the plan entry, and checks dependencies are closed. Open decisions come back to you with a recommendation.
3. **Already fixed?** It searches PRs, issues and commits, then verifies against the current code. If it's resolved, it drafts a plain-language **Product update** for the reporter; after you approve, it posts it and closes the issue. No branch.
4. **Build:** otherwise it creates a branch linked to the issue (`gh issue develop`, so it shows under the issue's Development section). For anything beyond a small fix it shows you a plan first. Then it makes the change with tests for the positive and negative cases, runs the repo's checks, updates `AGENTS.md` if needed, and reviews its own diff (with `/code-review` for bigger changes).
5. **PR:** a commit in the repo's style, then a PR with `Closes #42` and the standard sections: Choices made, What was wrong, What changed, Where to look, Tests, Follow-ups filed.
6. **Follow-ups:** real problems it finds outside the issue become new issues labelled `follow-up`, not extra changes in this PR.

**Your part:** answer decisions, approve the plan, read the diff, and run the app where the change is visible. On a UI change, look at it yourself.

**Working on several issues at once:** use one session per issue, each in its own git worktree, so they don't share a working tree:
```sh
git worktree add ../myrepo-42 -b <handle>/42-short-slug
```
Open that folder in a new VS Code window. Between unrelated issues in the same session, run `/clear` so old context doesn't leak into the new task.

**With pstack and ponytail:** pstack's session hook routes multi-file, design and debugging work to `poteto-mode`, which picks a playbook (Feature, Bug fix, Refactoring) for the build step. ponytail keeps the change to the smallest complete diff and ends each reply with what it skipped. Run `/pstack:deslop` before committing. They shape the code; `/flow:work-on-issue` still owns the branch, the PR and the gates.

### Phase 4: Review

Every PR gets a **human** approval. Claude's review is input that frees the reviewer for judgement calls; it is never the approval.

**Reviewing a teammate's PR:** `/flow:review-pr 57`. Claude:
1. Reads the PR, its linked issue, the milestone's acceptance criteria, `AGENTS.md` and existing comments.
2. Checks scope, each acceptance criterion (with evidence), the repo's invariants, tests (positive and negative; frontend changes must have tests), docs, and runs the checks (or cites the CI run).
3. Hunts bugs with the built-in `/code-review`, and `/security-review` when the diff touches auth, crypto, network exposure, untrusted input or permissions. It hands Rust changes to `rust-reviewer` and UI changes to `ui-ux-reviewer`.
4. **Verifies every finding** (reproduces it or names the exact lines and input) and drops what it can't substantiate. Problems that already existed on the base branch become follow-up issues, not blocking comments.
5. Drafts the review with Blocking / Should fix / Nits / Questions, and shows it to you. It posts only after you approve, and never merges.

**Addressing comments on your own PR:** `/flow:review-pr 57`, then say "address the review comments". It lists each comment with a proposed fix, disagreement or question, fixes the ones you agree with in small commits, runs the checks, and drafts replies for you to approve.

**Merging:** the human reviewer merges, using the repo's merge style. Then run status mode so the plan stays current.

**More review depth:** `/ponytail-review` asks whether every line is needed and safe. `/pstack:thermos` runs a deep bug, security and maintainability audit for a risky or large branch, and `/pstack:interrogate` sets several models against a contested design. `/pstack:babysit` drives an open PR's CI to green. Every finding still goes through `review-pr`'s verification before anyone sees it.

## 4. Day to day in VS Code

- **The Claude Code panel** is where you talk to Claude. The file you have open and any code you've selected are shared with it automatically. Mention other files with `@path`.
- **Permission modes:** by default Claude asks before editing files or running commands. Read what it wants to run before allowing it. Use plan mode for anything bigger than a small fix: Claude explores and proposes a plan without changing anything until you accept.
- **Checkpoints:** if a change goes the wrong way, `/rewind` takes the conversation and files back to an earlier point. It doesn't undo things outside the repo (pushes, issues), which is one more reason those need your OK.
- **Context:** `/clear` between unrelated tasks; `/compact` when a long session gets slow; `/context` shows what's using it.
- **Model and effort:** `/model` to switch. Use more capable settings for research, planning and review, and the default for routine implementation.
- **Paired plugins:** ponytail is always on; switch its level with `/ponytail lite`, `full` or `ultra`. Its band (cost, tokens, tests) shows in the terminal and desktop app, not in the VS Code panel. pstack spawns subagents and, for `arena`, `interrogate` and `swarm`, panels of several models, which costs more tokens. `/pstack:setup-pstack` picks its models per role or turns its session hook off.
- **Built-in skills worth knowing:** `/code-review` (bug hunt at a chosen level; `--comment` posts inline on a PR), `/security-review`, `/simplify` (cleanup of your diff), `/init` (first `CLAUDE.md`), `/memory` (edit memory files).
- **Agents in the plugin:** ask for them by name ("have the rust-reviewer look at this branch").
  - `rust-reviewer` (read-only): Rust best practice, backed by fmt, clippy and test output.
  - `ui-ux-reviewer` (read-only): looks at the running UI at phone, tablet and desktop widths in light and dark, and checks usability, WCAG 2.2 AA and the project's UI conventions. Give it test credentials in `UI_REVIEW_EMAIL`/`UI_REVIEW_PASSWORD` if pages need a sign-in.
  - `qa-automation-engineer`: test strategy and coverage.
  - `frontend-design-expert`: frontend implementation with a design eye.
  - `vscode-extension-expert`: VS Code extension work.

  The two reviewers never edit files. Their findings come back to your session, and you decide what to change.

## 5. Where knowledge lives

| What | Where | Shared? |
|---|---|---|
| Commands, architecture, invariants, conventions | `AGENTS.md` in the repo (`CLAUDE.md` imports it) | Yes, in git |
| Why we chose what we chose | `docs/research/*.md` | Yes, in git |
| Milestones, status, fixed decisions | GitHub milestones and issues, plus `docs/PLAN.md` | Yes |
| The reasoning behind a change | The PR body (Choices made) | Yes |
| Your personal preferences for one repo | `CLAUDE.local.md` (git-ignored) | No |
| Your defaults for every repo | `~/.claude/CLAUDE.md` | No |
| Claude's auto memory | Your user profile | No |

If something matters to the team, it goes in a shared place. Claude's memory and your personal files are invisible to everyone else.

## 6. Rules we follow

- **Review every diff before it's pushed.** You're the author of whatever you commit.
- **Never paste secrets, tokens or customer data into a prompt.** Shared settings deny reading `.env` files and keys; don't work around it.
- **Approve anything outward-facing yourself:** issues, comments, reviews, pushes, releases.
- **Claude never merges** and never approves a PR on someone's behalf.
- **Don't weaken an invariant** in `AGENTS.md` to make a change easier. If one is in the way, raise it.
- **Tests for the positive and the negative cases**, and every frontend change has tests.
- **One issue, one PR.** Out-of-scope findings become `follow-up` issues.
- **Mind CI cost.** macOS and Windows runners bill at a multiple of Linux. Run checks locally before pushing, and don't add expensive jobs to every PR without agreeing it first.
- **Check licences** of new dependencies, including their latest major version.
- **Leave ponytail's webhook empty** unless the team agrees on a shared board. Its task ledger stays on your machine, in `~/.config/ponytail/band/`.

## 7. Conventions

| Thing | Convention | Example |
|---|---|---|
| Milestone | `M<m>: <name>`; description has the goal and acceptance | `M2: Accounts and sign-in` |
| Issue (slice) | `M<m>.<n>: <outcome, imperative>` | `M2.3: Reset a forgotten password` |
| Follow-up issue | Symptom stated plainly, label `follow-up` | `Session expiry shows a blank page` |
| Branch | `<handle>/<issue>-<slug>`, created with `gh issue develop` | `wcolon/45-health-check` |
| Commit | `<Imperative summary> (#<issue>)` | `Add a health check endpoint (#45)` |
| PR title | Like the commit | `Show dates in the user's time zone (#125)` |
| PR body | `Closes #<issue>` + the template's sections | see `.github/pull_request_template.md` |
| Labels | `area:<part>`, `os:<name>`, `follow-up`, `phase-gate`, GitHub defaults | `area:api` |

A repo's own history wins: the skills read recent PRs and match what's there.

## 8. Troubleshooting

| Symptom | Fix |
|---|---|
| `/plugin isn't available in this environment` | The VS Code panel has no `/plugin`. Run `claude plugin ...` in a terminal. |
| `node: command not found` after each prompt | ponytail's hooks need Node on the PATH of a non-interactive shell (nvm users: check it). |
| `/flow:` shows nothing | `claude plugin list`; if missing, install it (section 1). Restart the session after installing or updating. |
| `gh` answers `404` for the repo | `gh auth status`; switch to the account with access: `gh auth switch -u <account>`. |
| Claude keeps asking to run read-only commands | The repo lacks `.claude/settings.json`; run `/flow:setup-project`. |
| The agent uses the wrong test command or breaks a rule | `AGENTS.md` is wrong or missing it. Fix it in a small PR. |
| Another model skips steps or ignores the skill | Check its context length (32K at least) and size; see [other-models.md](other-models.md). Small models aren't reliable for these skills. |
| Windows: commands fail in the Bash tool | Claude Code on Windows needs Git for Windows (it uses Git Bash). See [tools.md](tools.md). |
| The session is slow or drifts | `/compact`, or `/clear` and restate the task. |

## 9. Changing the kit

The kit is a repo like any other, with the same workflow: an issue, a branch, a PR, a review.
- Skills are in `plugins/flow/skills/<name>/SKILL.md`. The `description` decides when Claude uses it, so name the phrases people actually say.
- Agents are in `plugins/flow/agents/`. Quote a `description` that contains `: `, or YAML can't parse it.
- Bump `version` in `plugins/flow/.claude-plugin/plugin.json` with every change, so `claude plugin update` picks it up.
- Run `claude plugin validate .` and `claude plugin validate plugins/flow` before pushing (CI runs both).
- Try a change locally first: `claude plugin marketplace add <path to your clone>`, then install from it.
- The `skill-creator` skill (if your account has it) can help write and test a new skill.
