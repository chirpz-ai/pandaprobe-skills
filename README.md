# pandaprobe-skills

An [Agent Skill](https://docs.claude.com/en/docs/claude-code/skills) that teaches AI coding
agents (Claude Code, Cursor, etc.) how to work with
[**PandaProbe**](https://pandaprobe.com) — the agent engineering platform for tracing,
evaluating, and monitoring AI agents.

The skill covers two things:

- **CLI** — install, authenticate, and drive the agent-first `pandaprobe` CLI to inspect
  traces, sessions, spans, scores, and evaluation runs.
- **SDK instrumentation** — add PandaProbe tracing to a Python app via provider wrappers,
  agent-framework integrations, or manual decorators.

It uses progressive disclosure: the agent sees only the skill's name and description until a
task matches, then reads `skills/SKILL.md`, and only pulls in
`skills/references/cli.md` or `skills/references/instrumentation.md` when that specific
topic is needed.

## Install

Uses [`npx skills`](https://github.com/vercel-labs/skills), the open agent-skills CLI.

```bash
# Project (current repo):
npx skills add chirpz-ai/pandaprobe-skills --yes

# Global (all projects), Claude Code:
npx skills add chirpz-ai/pandaprobe-skills --agent claude-code --global --yes
```

Install from a local checkout (e.g. to test changes):

```bash
npx skills add ./ --agent claude-code
```

## Prerequisites

- The **`pandaprobe` CLI** for data/eval access — install with
  `curl -fsSL https://cli.pandaprobe.com/install.sh | sh` (see `skills/references/cli.md`).
- A **PandaProbe account and API key** for PandaProbe Cloud — sign up at
  [app.pandaprobe.com](https://app.pandaprobe.com), then run `pandaprobe auth login`.
  Self-hosted deployments can configure an endpoint and key manually instead.
- For SDK instrumentation: **Python 3.10+** and `pip install pandaprobe`.

## Usage

Once installed, the agent picks the skill up automatically for prompts like:

- "Use pandaprobe to list my failed traces."
- "Drill into the error in trace `tr_…`."
- "Add PandaProbe tracing to my LangGraph agent."
- "Run a coherence eval over yesterday's completed traces."

## Docs

Full documentation lives at [docs.pandaprobe.com](https://docs.pandaprobe.com).

## License

MIT — see [LICENSE](LICENSE).
