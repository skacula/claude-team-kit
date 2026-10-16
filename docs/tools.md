# Tools and packages

Everything we use, with an install command per OS. Prefer each tool's official installer or your OS package manager. When a command here is out of date, the linked official page wins.

Windows users can work natively or inside WSL2. If you use WSL2, follow the Linux column inside it and install VS Code's WSL extension.

## Required on every machine

| Tool | Why we need it | macOS | Windows | Linux |
|---|---|---|---|---|
| [VS Code](https://code.visualstudio.com/) | Editor; hosts the Claude Code extension | `brew install --cask visual-studio-code` | `winget install Microsoft.VisualStudioCode` | `.deb`/`.rpm` from the site, or `snap install code --classic` |
| [Git](https://git-scm.com/) | Version control. On Windows, Git for Windows also provides the Git Bash that Claude Code runs commands in | `xcode-select --install` or `brew install git` | `winget install Git.Git` | `sudo apt install git` / `sudo dnf install git` |
| [GitHub CLI](https://cli.github.com/) (`gh`) | Every skill uses it for issues, milestones, PRs and reviews | `brew install gh` | `winget install GitHub.cli` | See [cli.github.com](https://github.com/cli/cli/blob/trunk/docs/install_linux.md) (apt/dnf repos) |
| [Node.js LTS](https://nodejs.org/) | Runs `npx`-based tools (skills CLI, MCP servers) and JS projects | `brew install node` or a version manager (nvm, fnm, volta) | `winget install OpenJS.NodeJS.LTS` or fnm/volta | Your distro's package, or nvm/fnm |
| [Claude Code](https://docs.claude.com/en/docs/claude-code/setup) (CLI) | Plugin management, `claude doctor`, terminal sessions | `npm install -g @anthropic-ai/claude-code` (or the native installer from the setup page, or `brew install --cask claude-code`) | `npm install -g @anthropic-ai/claude-code` (or the native PowerShell installer from the setup page, or `winget install Anthropic.ClaudeCode`) | `npm install -g @anthropic-ai/claude-code` (or the native installer from the setup page) |
| Claude Code for VS Code (`anthropic.claude-code`) | The Claude panel in the editor | `code --install-extension anthropic.claude-code` | same | same |
| A Claude account | Sign in with `/login`; use the team plan | | | |

After installing: `gh auth login`, `claude` then `/login`, and `claude doctor`.

## Claude Code pieces we use

| Piece | What it is | How you get it |
|---|---|---|
| `flow` plugin | Our skills (`setup-project`, `research`, `plan-milestones`, `work-on-issue`, `review-pr`) and agents (`rust-reviewer`, `ui-ux-reviewer`, `qa-automation-engineer`, `frontend-design-expert`, `vscode-extension-expert`) | `claude plugin marketplace add skacula/claude-team-kit`, then `claude plugin install flow@claude-team-kit`; or automatically when you trust a repo set up with `/flow:setup-project` |
| `ponytail` plugin | Always-on "smallest complete change" mode, plus `/ponytail-review`, `/ponytail-audit` and `/ponytail-debt`. Its hooks need `node` on the PATH | `claude plugin marketplace add "DietrichGebert/ponytail#v5.3.1"`, then `claude plugin install ponytail@ponytail` |
| `pstack` plugin | `poteto-mode` playbooks and skills such as `how`, `why`, `architect`, `tdd`, `thermos`, `interrogate`, `babysit`. Its session hook is a shell script (Git Bash on Windows) | `claude plugin marketplace add "michael-denyer/pstack-claude#v0.9.86"`, then `claude plugin install pstack@pstack-claude` |
| code-metrics plugin | Local complexity and duplication reports: `/code-metrics-tracker:metrics`, `/code-metrics-tracker:refactor-targets`. Needs Python 3 | `claude plugin install code-metrics-complexity-and-duplication-report@anthropic-plugin-directory` (Anthropic's built-in directory; no marketplace to add) |
| Built-in skills | `/code-review`, `/security-review`, `/simplify`, `/init`, `/memory`, plan mode, `/rewind` | Ship with Claude Code |
| Claude account skills | `deep-research`, `docx`, `pdf`, `pptx`, `xlsx`, `skill-creator` | Available when enabled on your claude.ai account; optional |
| `find-skills` (optional) | Searches the open skills ecosystem for a skill that fits a task | `npx skills add vercel-labs/skills` and pick `find-skills` |
| `sentry-cli` skill (optional) | Using Sentry from the command line | Only for projects on Sentry; find it with `npx skills find sentry` |
| [Atlassian CLI](https://developer.atlassian.com/cloud/acli/guides/install-acli/) (`acli`, only for repos that track issues in Jira) | The skills use it for Jira work items, Epics, comments and transitions; code and PRs still use `gh`. Jira Cloud only | `brew tap atlassian/acli && brew install acli` on macOS; the install page has Windows and Linux. Then `acli jira auth login --web` |
| Other models (optional) | Ollama for local models; Qwen Code; other agent tools | See [other-models.md](other-models.md) |
| MCP servers (optional, per project) | Live docs or services inside Claude | Add to the repo's `.mcp.json` only when a project needs one. We use `gh` rather than a GitHub MCP server: it's already installed, works the same everywhere and costs fewer tokens |

## VS Code extensions

`/flow:setup-project` writes the right list into each repo's `.vscode/extensions.json`, and VS Code offers to install them when you open the folder (or run **Extensions: Show Recommended Extensions**). All of these work on macOS, Windows and Linux.

| For | Extensions |
|---|---|
| Everyone | `anthropic.claude-code`, `github.vscode-pull-request-github`, `github.vscode-github-actions`, `editorconfig.editorconfig`, `streetsidesoftware.code-spell-checker`, `redhat.vscode-yaml` |
| TypeScript / Vue / React | `dbaeumer.vscode-eslint`, `esbenp.prettier-vscode`, `vue.volar`, `vitest.explorer` |
| Rust | `rust-lang.rust-analyzer`, `tamasfe.even-better-toml`, `vadimcn.vscode-lldb` |
| Python | `ms-python.python`, `ms-python.vscode-pylance`, `ms-python.debugpy`, `charliermarsh.ruff` |
| Go | `golang.go` |
| Containers | `ms-azuretools.vscode-containers`, `ms-vscode-remote.remote-containers` |
| Terraform | `hashicorp.terraform` |
| C / C++ | `ms-vscode.cpptools-extension-pack` |

## Language toolchains (install what your project uses)

The project's `AGENTS.md` (or `CLAUDE.md`) lists the exact versions and commands. Common ones:

| Toolchain | macOS | Windows | Linux |
|---|---|---|---|
| Rust ([rustup](https://rustup.rs/)) | `brew install rustup` then `rustup-init`, or the rustup.rs script | `winget install Rustlang.Rustup` | the rustup.rs script |
| Python via [uv](https://docs.astral.sh/uv/) | `brew install uv` | `winget install astral-sh.uv` | the installer on the uv site, or `pipx install uv` |
| Docker | Docker Desktop, or `brew install colima docker` | Docker Desktop (WSL2 backend) | Docker Engine from your distro or docker.com |
| Go | `brew install go` | `winget install GoLang.Go` | go.dev/dl |
