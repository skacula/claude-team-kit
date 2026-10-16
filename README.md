# claude-team-kit

How our team works with Claude Code, packaged so every engineer gets the same skills, agents, templates and conventions on macOS, Linux or Windows.

## Install

You need VS Code, Git, the GitHub CLI, Node.js LTS and Claude Code ([tools.md](docs/tools.md)). Then:

```sh
claude plugin marketplace add skacula/claude-team-kit
claude plugin install flow@claude-team-kit
```

Then the plugins we pair with it:

```sh
claude plugin marketplace add "DietrichGebert/ponytail#v5.3.1"
claude plugin marketplace add "michael-denyer/pstack-claude#v0.9.86"
claude plugin install ponytail@ponytail
claude plugin install pstack@pstack-claude
claude plugin install code-metrics-complexity-and-duplication-report@anthropic-plugin-directory
```

Restart your Claude session and type `/flow:` to see the skills. Run these in a terminal: the VS Code chat panel has no `/plugin` command. The `#v…` tags pin a release; to upgrade, bump them here, in `docs/tools.md` and in `plugins/flow/skills/setup-project/templates/settings.json` in one PR. In a repo that's been set up with `/flow:setup-project`, Claude Code offers all four plugins when you trust the folder, so you can skip these steps there.

**Using Qwen or another model?** The kit isn't tied to Claude models: run Claude Code on Ollama, Qwen or OpenRouter, install it into Qwen Code with `qwen extensions install skacula/claude-team-kit:flow`, or into other agent tools with `npx skills add skacula/claude-team-kit`. See [docs/other-models.md](docs/other-models.md).

## What's in it

| Skill | Phase | What it does |
|---|---|---|
| `/flow:setup-project` | Once per repo | `AGENTS.md` (imported by `CLAUDE.md`), shared `.claude/settings.json`, VS Code recommendations, PR/issue templates, labels |
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

## The plugins around flow

`flow` owns the process: phases, GitHub conventions and human gates. Three paired plugins shape the work inside each step: `ponytail`, `pstack` and code-metrics. [docs/how-to-guide.md](docs/how-to-guide.md#the-plugins-around-flow) says what each does and which helper to reach for in each phase.

## Read next

- [docs/how-to-guide.md](docs/how-to-guide.md): the workflow, step by step
- [docs/tools.md](docs/tools.md): every tool and how to install it on each OS
- [docs/other-models.md](docs/other-models.md): running the kit with Qwen, local and other models
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
