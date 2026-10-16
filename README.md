# claude-team-kit

How our team works with Claude Code, packaged so every engineer gets the same skills, agents, templates and conventions on macOS, Linux or Windows.

## Install

You need VS Code, Git, the GitHub CLI, Node.js LTS and Claude Code ([tools.md](docs/tools.md)). Then:

```sh
claude plugin marketplace add skacula/claude-team-kit
claude plugin install flow@claude-team-kit
```

Restart your Claude session and type `/flow:` to see the skills. In a repo that's been set up with `/flow:setup-project`, Claude Code offers the plugin when you trust the folder, so you can skip this step there.

## What's in it

| Skill | Phase | What it does |
|---|---|---|
| `/flow:setup-project` | Once per repo | `CLAUDE.md`, shared `.claude/settings.json`, VS Code recommendations, PR/issue templates, labels |
| `/flow:research` | 1. Research | A sourced report: options compared, recommendation, risks, decisions for a human |
| `/flow:plan-milestones` | 2. Milestones | Plan with milestones and acceptance criteria, then GitHub milestones, labels and `M<m>.<n>:` issues; `status` keeps it in sync |
| `/flow:work-on-issue` | 3. Issues | Checks whether it's already fixed; otherwise a linked branch, the change with tests, and a PR; out-of-scope findings become follow-up issues |
| `/flow:review-pr` | 4. Review | A verified review against the issue's acceptance and the repo's invariants, posted only with your OK; also works through review comments on your own PR |

| Agent | What it does |
|---|---|
| `rust-reviewer` | Read-only. Reviews Rust changes for best practice (errors, API design, async, unsafe, security, tests) and backs findings with fmt/clippy/test output |
| `ui-ux-reviewer` | Read-only. Looks at the running UI at several viewports and both themes; reviews layout, usability, WCAG 2.2 AA and the project's UI conventions |
| `qa-automation-engineer` | Test strategy and test automation across web, mobile and backend |
| `frontend-design-expert` | Frontend implementation with a design eye |
| `vscode-extension-expert` | Building and debugging VS Code extensions |

## Read next

- [docs/how-to-guide.md](docs/how-to-guide.md): the workflow, step by step
- [docs/tools.md](docs/tools.md): every tool and how to install it on each OS
- [templates/user-CLAUDE.md](templates/user-CLAUDE.md): optional personal defaults for `~/.claude/CLAUDE.md`

## Layout

```
.claude-plugin/marketplace.json     the marketplace (one plugin: flow)
plugins/flow/.claude-plugin/plugin.json
plugins/flow/skills/<name>/SKILL.md  one skill per phase, with its templates beside it
plugins/flow/agents/*.md             shared agents
docs/                                guide and tools list
templates/                           personal files you copy by hand
```

Changes go through a PR like any other repo. Bump the plugin's `version` and run `claude plugin validate .` and `claude plugin validate plugins/flow` first; CI runs both.
