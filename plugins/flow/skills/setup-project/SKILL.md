---
name: setup-project
description: Prepares a repository for the team's Claude Code workflow. Writes or refreshes CLAUDE.md, adds the shared .claude/settings.json that enables this plugin for everyone, the recommended VS Code extensions, PR and issue templates and the standard labels, and keeps personal files out of git. Works on a new or an existing repo and merges into files that already exist. Use when the user says "set up this repo for Claude", "onboard this project", "add the team kit to this repo", "bootstrap the project", "create a CLAUDE.md", or starts a new project.
---

# Set up a project

After this, anyone who clones the repo and trusts the folder in Claude Code gets the same plugin, the same rules (`CLAUDE.md`) and the same templates. Templates are in [templates/](templates/).

Never overwrite a file that exists: read it and merge, keeping what's there. Show the user what will change before writing, and put the change on a branch with a PR, like any other change.

## 0. Look first

- `git status` and the current branch. With uncommitted changes, ask before going on.
- `gh repo view --json nameWithOwner,defaultBranchRef,visibility` (if it isn't on GitHub yet, skip the GitHub steps and say so).
- What exists already: `CLAUDE.md`, `CLAUDE.local.md`, `.claude/settings.json`, `.vscode/extensions.json`, `.github/pull_request_template.md`, `.github/ISSUE_TEMPLATE/`, `.gitignore`, `gh label list`.
- The stack: build files (`package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `*.csproj`, `Dockerfile`, CI workflows). Read the CI workflow to learn the real commands.
- Where this plugin was installed from: `claude plugin marketplace list` shows the `claude-team-kit` marketplace and its source (e.g. `org/claude-team-kit`). You need it for the settings file.

Show the user a short list of what you'll add or change, and get an OK.

## 1. CLAUDE.md

This is the contract every session reads first, so it must be accurate and short enough to keep current.
- **No CLAUDE.md:** write one from [templates/CLAUDE.md](templates/CLAUDE.md). Fill each section from the repo itself: commands from the build files and CI (run each one to confirm it works, and note how long the slow ones take), the architecture from the directory layout and entry points, conventions from recent PRs and commits. Delete sections that don't apply. Leave `<!-- TODO -->` only where you couldn't find the answer, and list those for the user.
- **CLAUDE.md exists:** keep it. Add only what's missing (the workflow section, commands that don't work as written, a missing invariants section), and show the diff.
- If the repo has a plan or research docs, link them. Invariants agreed in research belong in the **Invariants** section.

## 2. Shared settings: `.claude/settings.json`

Start from [templates/settings.json](templates/settings.json). It points at `skacula/claude-team-kit`; if step 0 showed the marketplace coming from somewhere else (a fork), use that source instead. If the file exists, merge the keys: add the marketplace and the plugin, and add the permission rules that aren't already there. Don't remove the user's rules.

Explain what it does: when a teammate trusts this folder, Claude Code offers to add the team marketplace and install the `flow` plugin. The `allow` list only covers read-only commands, so nothing that changes state runs without asking. The `deny` list keeps Claude from reading env files and keys.

## 3. VS Code: `.vscode/extensions.json`

Start from [templates/extensions.json](templates/extensions.json). Keep the general recommendations and add only the ones for the stacks this repo uses (the template lists them by stack in `_byStack`; move the matching ones into `recommendations` and drop `_byStack` from the written file). Merge with an existing file. VS Code then offers the extensions to everyone who opens the folder.

## 4. GitHub templates

If absent, add:
- [templates/pull_request_template.md](templates/pull_request_template.md) → `.github/pull_request_template.md`
- [templates/ISSUE_TEMPLATE/slice.md](templates/ISSUE_TEMPLATE/slice.md) and [templates/ISSUE_TEMPLATE/bug.md](templates/ISSUE_TEMPLATE/bug.md) → `.github/ISSUE_TEMPLATE/`

If the repo has its own templates, keep them, and tell the user how they differ from the team's.

## 5. Labels

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
```

## 7. Commit

On a branch (e.g. `<handle>/setup-claude-workflow`), commit with a message like `Set up the team's Claude Code workflow`, push, and open a PR whose body lists what was added and the TODOs left in `CLAUDE.md`. Give the user the PR link.

Then suggest the next step: `/flow:research` for a new project, or `/flow:plan-milestones` if the scope is already known.
