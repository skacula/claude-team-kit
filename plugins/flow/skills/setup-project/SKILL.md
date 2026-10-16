---
name: setup-project
description: Prepares a repository for the team's AI coding workflow, whatever the agent tool or model. Writes or refreshes AGENTS.md (with a CLAUDE.md that imports it), adds the shared .claude/settings.json that enables this plugin for everyone, the recommended VS Code extensions, PR and issue templates and the standard labels, and keeps personal files out of git. Works on a new or an existing repo and merges into files that already exist. Use when the user says "set up this repo", "set up this repo for Claude", "onboard this project", "add the team kit to this repo", "bootstrap the project", "create an AGENTS.md" or "create a CLAUDE.md", or starts a new project.
---

# Set up a project

After this, anyone who clones the repo gets the same rules, the same templates and, in Claude Code, the same plugin, whichever agent tool and model they use. Templates are in [templates/](templates/).

Never overwrite a file that exists: read it and merge, keeping what's there. Show the user what will change before writing, and put the change on a branch with a PR, like any other change.

## 0. Look first

- `git status` and the current branch. With uncommitted changes, ask before going on.
- `gh repo view --json nameWithOwner,defaultBranchRef,visibility` (if it isn't on GitHub yet, skip the GitHub steps and say so).
- What exists already: `AGENTS.md`, `CLAUDE.md`, `QWEN.md`, `CLAUDE.local.md`, `.claude/settings.json`, `.vscode/extensions.json`, `.github/pull_request_template.md`, `.github/ISSUE_TEMPLATE/`, `.gitignore`, `gh label list`.
- The stack: build files (`package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `*.csproj`, `Dockerfile`, CI workflows). Read the CI workflow to learn the real commands.
- Where issues are tracked: GitHub issues and milestones (the default), or Jira. Ask the user if you can't tell. For Jira, get the project key and site, and check `acli jira auth status` shows that site (if it doesn't, ask the user to run `acli jira auth login --web`). The code and PRs stay on GitHub either way.
- In Claude Code, `claude plugin marketplace list` shows where the `claude-team-kit` marketplace came from. You need it for the settings file. In other tools, use the default, `skacula/claude-team-kit`.

Show the user a short list of what you'll add or change, and get an OK.

## 1. The instructions file: `AGENTS.md`, imported by `CLAUDE.md`

This is the contract every session reads first, so it must be accurate and short enough to keep current. It lives in `AGENTS.md` because most agent tools read that file (Qwen Code, Codex, OpenCode and GitHub Copilot among them). Claude Code reads `CLAUDE.md` instead, so `CLAUDE.md` holds one line, `@AGENTS.md`, which imports it. Don't use a symlink: symlinks break on Windows checkouts.

- **No instructions file:** write `AGENTS.md` from [templates/AGENTS.md](templates/AGENTS.md) and `CLAUDE.md` from [templates/CLAUDE.md](templates/CLAUDE.md). Fill each section of `AGENTS.md` from the repo itself:
  - commands from the build files and CI (run each one to confirm it works, and note how long the slow ones take)
  - the architecture from the directory layout and entry points
  - conventions from recent PRs and commits

  Delete sections that don't apply. Leave `<!-- TODO -->` only where you couldn't find the answer, and list those for the user.
- **Only `CLAUDE.md` exists:** propose moving its content to `AGENTS.md` with `git mv CLAUDE.md AGENTS.md`, which keeps its history, and then writing the one-line `CLAUDE.md`. Show the user what that does, and do it only if they agree. If they'd rather not, leave it, and tell them that tools other than Claude Code won't see the rules.
- **`AGENTS.md` exists:** keep it. Make sure `CLAUDE.md` imports it rather than duplicating it.
- **In every case:** add only what's missing (the workflow section, commands that don't work as written, a missing invariants section), and show the diff.
- For a Jira repo, swap the first two **Workflow** bullets for the Jira version in the template's comment, with the real project key and site. The other skills read that line to know where issues live.
- If the repo has a plan or research docs, link them. Invariants agreed in research belong in the **Invariants** section.

## 2. Shared settings: `.claude/settings.json`

Start from [templates/settings.json](templates/settings.json). It points at `skacula/claude-team-kit`; if step 0 showed the marketplace coming from somewhere else (a fork), use that source instead. If the file exists, merge the keys: add the marketplace and the plugin, and add the permission rules that aren't already there. Don't remove the user's rules.

Explain what it does:
- When a teammate trusts this folder in Claude Code, it offers to add the marketplaces and install `flow` and the plugins the team pairs with it: `ponytail`, `pstack` and code-metrics. If the project doesn't want one of them, remove it from `enabledPlugins` and say so in the PR. ponytail and pstack are pinned to a release tag (`ref`); keep the pin.
- The `allow` list only covers read-only commands, so nothing that changes state runs without asking.
- The `deny` list stops Claude's Read tool from opening env files, keys and `secrets/` folders anywhere in the repo. It doesn't cover shell commands such as `cat`, which still ask first.

Claude Code enforces these rules whichever model it runs. Other tools have their own permission settings. Tell the user to keep those in the mode that asks before editing files or running commands, especially with smaller models.

## 3. VS Code: `.vscode/extensions.json`

Start from [templates/extensions.json](templates/extensions.json). Keep the general recommendations and add only the ones for the stacks this repo uses (the template lists them by stack in `_byStack`; move the matching ones into `recommendations` and drop `_byStack` from the written file). Merge with an existing file. VS Code then offers the extensions to everyone who opens the folder.

## 4. GitHub templates

If absent, add:
- [templates/pull_request_template.md](templates/pull_request_template.md) → `.github/pull_request_template.md`
- [templates/ISSUE_TEMPLATE/slice.md](templates/ISSUE_TEMPLATE/slice.md) and [templates/ISSUE_TEMPLATE/bug.md](templates/ISSUE_TEMPLATE/bug.md) → `.github/ISSUE_TEMPLATE/`

If the repo has its own templates, keep them, and tell the user how they differ from the team's. For a Jira repo, add only the PR template: the issue templates don't reach Jira.

## 5. Labels

For a Jira repo, skip this step: Jira creates a label the first time it's used, and `plan-milestones` uses the same names there.

Ask before creating labels, since they're visible to everyone. Propose:
- `follow-up` ("Found while delivering a slice"), `phase-gate` ("A gate or end-to-end verification")
- `area:<part>` per major component you found in step 0 (3–10, named after directories or layers)
- `os:macos`, `os:linux`, `os:windows` only if the project ships on several OSes

After approval: `gh label create "<name>" --description "<…>" --color <hex>` for each one that doesn't exist.

## 6. Personal files stay out of git

Make sure `.gitignore` has:
```
CLAUDE.local.md
.claude/settings.local.json
.qwen/QWEN.local.md
```

## 7. Commit

On a branch (e.g. `<handle>/setup-agent-workflow`), commit with a message like `Set up the team's agent workflow`, push, and open a PR whose body lists what was added and the TODOs left in `AGENTS.md`. Give the user the PR link.

Then suggest the next step: `/flow:research` for a new project, or `/flow:plan-milestones` if the scope is already known.
