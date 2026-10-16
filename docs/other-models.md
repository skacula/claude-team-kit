# Using Qwen and other models

The kit doesn't depend on Claude models. Skills are plain `SKILL.md` files in the open [Agent Skills](https://agentskills.io/) format, the agents are Markdown, and every project's rules live in `AGENTS.md`, which most agent tools read. What changes from model to model is **how well** the workflow runs, not **whether** it loads.

There are three ways to run it, from most to least complete.

| Setup | Skills | Reviewer agents | Permission rules | Missing |
|---|---|---|---|---|
| **A. Claude Code with another model** | All, unchanged | All | Enforced, as in `.claude/settings.json` | Hosted web search, claude.ai account skills (browser, `deep-research`, documents), prompt caching on most providers |
| **B. Qwen Code** | All, converted on install | All, converted on install | Qwen Code's own approval modes | `/code-review` and `/security-review` (the skills fall back to their own pass), the Claude browser tools |
| **C. Other agent tools** (Codex, OpenCode, Copilot, Cursor, Gemini CLI…) | All, via `npx skills` | Not installed; see below | That tool's own | Anything Claude Code-specific |

## A. Claude Code with another model (recommended)

The whole kit runs as it does on Claude: the plugin, all skills and agents, and the permission rules in `.claude/settings.json`. Claude Code talks to any server that speaks Anthropic's Messages API, and Ollama, Alibaba Model Studio and OpenRouter all do.

Put the settings in the repo's **`.claude/settings.local.json`**. It's git-ignored, applies to the CLI and the VS Code extension alike, works the same on every OS, and affects only that repo. To use the model everywhere, put the same `env` block in `~/.claude/settings.json` instead.

The reviewer agents use `model: inherit`, so they run on whatever model the session uses. Claude Code's background tasks use the `haiku` alias, which is why each example below maps all three aliases.

### Local, free and private: Ollama

Nothing leaves your machine. This needs Ollama 0.14 or later and a machine that can run a capable coding model. `qwen3-coder` (30B) is about a 20 GB download; plan on 32 GB of memory, or a GPU with 24 GB.

1. Pull the model and start Ollama with a long context. The kit's skills plus a large `AGENTS.md` don't fit in Ollama's default; use at least 32K tokens, and 64K is better:
   ```sh
   ollama pull qwen3-coder
   OLLAMA_CONTEXT_LENGTH=65536 ollama serve
   ```
   On Windows PowerShell: `$env:OLLAMA_CONTEXT_LENGTH=65536; ollama serve`. On macOS and Windows you can set the context length in the Ollama app's settings instead.
2. `.claude/settings.local.json`:
   ```json
   {
     "env": {
       "ANTHROPIC_BASE_URL": "http://localhost:11434",
       "ANTHROPIC_AUTH_TOKEN": "ollama",
       "ANTHROPIC_MODEL": "qwen3-coder",
       "ANTHROPIC_DEFAULT_OPUS_MODEL": "qwen3-coder",
       "ANTHROPIC_DEFAULT_SONNET_MODEL": "qwen3-coder",
       "ANTHROPIC_DEFAULT_HAIKU_MODEL": "qwen3-coder"
     }
   }
   ```
3. Start a new session and check that `/model` shows `qwen3-coder`.

Ollama's Anthropic API doesn't support everything ([its list](https://docs.ollama.com/api/anthropic-compatibility)). The gaps that matter here are no hosted web search, no prompt caching, no PDF documents and no forced tool choice.

### Hosted Qwen: Alibaba Model Studio (Qwen Cloud)

Alibaba offers an Anthropic-compatible endpoint for Claude Code. The URL depends on your region and on whether you use pay-as-you-go or a Coding Plan, so copy it from Model Studio's Claude Code guide rather than from a blog. Then:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "<the Anthropic-compatible URL from Model Studio's Claude Code guide>",
    "ANTHROPIC_AUTH_TOKEN": "<your Model Studio API key>",
    "ANTHROPIC_MODEL": "<model name from the same guide>",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "<same>",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "<same>",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "<a smaller Qwen model, or the same>"
  }
}
```

### Many models, including free ones: OpenRouter

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://openrouter.ai/api",
    "ANTHROPIC_AUTH_TOKEN": "<your OpenRouter API key>",
    "ANTHROPIC_API_KEY": "",
    "ANTHROPIC_MODEL": "<an OpenRouter model id, e.g. a qwen/… coder model>",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "<same>",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "<same>",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "<same or smaller>"
  }
}
```

`ANTHROPIC_API_KEY` must be explicitly empty ([OpenRouter's guide](https://openrouter.ai/docs/guides/guides/claude-code-integration)). OpenRouter warns that Claude Code "may not work correctly" with non-Anthropic models. Free variants have low rate limits, and long skills hit them fast.

## B. Qwen Code

Qwen Code installs plugins straight from a Claude Code marketplace and converts their skills and agents ([its docs](https://github.com/QwenLM/qwen-code/blob/main/docs/users/extension/introduction.md)):

```sh
npm install -g @qwen-code/qwen-code
qwen extensions install skacula/claude-team-kit:flow
```

- The skills are called the same way: `/flow:work-on-issue 42`.
- It reads `AGENTS.md`. `/flow:setup-project` writes one, so set up repos are covered.
- `.claude/settings.json` doesn't apply. Leave Qwen Code in its default approval mode, which asks before edits and commands. Don't use YOLO mode with this kit: the skills' "ask before posting" steps are instructions, and a model can skip an instruction.
- The skills use their own bug-hunting pass where Claude Code would run `/code-review`.
- To update the plugin: `qwen extensions update flow`.

## C. Other agent tools

Install the skills into any of 40+ tools with the open skills installer:

```sh
npx skills add skacula/claude-team-kit -a <agent>      # e.g. -a codex, -a opencode, -a github-copilot, -a qwen-code
```

Most of these tools read `AGENTS.md`. The reviewer agents aren't installed this way, because each tool has its own agent format. Ask the tool to "read `plugins/flow/agents/rust-reviewer.md` from the kit and review this branch following it", or convert the file to that tool's format (for OpenCode, `.opencode/agent/`).

## What to expect from the model

Every model can load the skills, but how well it follows them varies. The skills are long, multi-step procedures with many tool calls, and they need the model to stop and ask at the right moments. Roughly:

| Model class | Works well for | Watch out for |
|---|---|---|
| Claude Opus/Sonnet | Every phase | |
| Large hosted open models (Qwen3-Coder and similar, hosted) | Work on issues, plan status, a first review pass | Thinner research without web search; check reviews more closely |
| Local ~30B coding models (`qwen3-coder` on Ollama) | Small, well-specified issues; plan status; read-only checks | Skipped steps in long skills, weaker judgement in reviews, slow on a laptop |
| Under ~14B | Not recommended for these skills | Broken tool calls, lost instructions |

Test before relying on a model. In a scratch repo, run `/flow:work-on-issue` on a small issue, and check that the model:
- asked before pushing and before opening the PR
- wrote tests for the positive and the negative cases
- ran the checks and reported their real output
- kept to the issue's scope

## Rules that don't change with the model

- **The human gates are enforced by the tool, not the model.** In Claude Code, the shared settings pre-approve only read-only `git` and `gh` commands, so creating an issue, posting a review or pushing still asks you, whatever the model does. Keep that true: don't widen the allow list, and don't use bypass or auto modes with weaker models.
- **Your code goes to whoever runs the model.** A local Ollama model keeps it on your machine. A hosted model, free ones especially, sends your code and prompts to that provider under its terms, which may include logging or training. Check the provider's terms and the project's policy before using a hosted model on company or private code.
- **Keep keys out of git.** API keys go in `.claude/settings.local.json` (git-ignored) or your user settings, never in `.claude/settings.json`.
- **Write down the model.** Research reports name the tool and model that produced them; PRs should too when it isn't Claude. It tells the reviewer how closely to check.
