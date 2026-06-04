---
name: pandaprobe-setup
description: Guided, interactive onboarding for PandaProbe — build and run a small working example from scratch so a new user sees their first traces fast. Use when the user asks to "set up PandaProbe", "add PandaProbe", "get started with PandaProbe", or "onboard", especially with an empty or new project. Asks which provider and what to build, implements the example doc-first, runs it, then verifies traces with the CLI.
---

# Set up PandaProbe (guided onboarding)

Goal: get a new user — often with an **empty or unfamiliar project** — from zero to their
**first traces** by building and running a small, working PandaProbe example *for* them,
then pointing them to next steps. This is the ground-up path; it favors a runnable example
over theory.

This is an orchestrator: it makes the choices interactive, implements the example **doc-first**
(from [instrumentation.md](instrumentation.md) + the live docs), and verifies with the CLI
([cli.md](cli.md)).

**Ground rules**

- **Be interactive.** Ask the user the questions in Step 1 and wait for answers — don't
  assume. Keep it friendly and concrete.
- **Confirm before side effects.** Get explicit confirmation before installing packages,
  creating files, or authenticating.
- **Doc-first — never from memory.** Fetch the latest docs for the chosen provider/framework
  and implement the example from them (exact imports, wrapper/handler wiring). The SDK
  changes often.
- **Never echo the API key.** Have the user set credentials themselves; don't print or paste keys.

## Step 0 — Orient & branch

Figure out which path the user needs:

- **They already have an app and want to trace it** → this is the instrument flow, not
  onboarding. Use [instrumentation.md](instrumentation.md) (SKILL.md section 1) instead.
- **Empty/new project, exploring, or wants a guided example** → continue here.

Confirm Python 3.10+ is available (PandaProbe's SDK is Python; there is no TS SDK yet), and
pick/confirm a working directory (create one and a virtualenv if helpful).

## Step 1 — Ask what to build (interactive)

Ask the user two questions and wait for answers:

1. **Which model provider** do they want to use — OpenAI, Anthropic, Gemini, Mistral, or
   Bedrock? (They'll need that provider's API key.)
2. **What should the first example be?**
   - **A) Simple LLM call** — one traced call via a provider wrapper (e.g. `wrap_openai`).
     Best for a first look: minimal code, a single trace.
   - **B) Agent with a framework** — a small agent with one tool, traced via a framework
     integration, so they see a multi-span trace (LLM + tool + agent). For this, ask which
     framework they'd like; recommend one that pairs well with their provider (e.g.
     LangGraph or OpenAI Agents) and confirm.

Offer to build just one now and the other later. Summarize the plan (provider + example +
packages you'll install) and confirm before proceeding.

## Step 2 — Credentials & install

Confirm before installing.

- **PandaProbe:** have the user set these (key copied from the dashboard at
  https://app.pandaprobe.com — **don't paste it yourself**):
  ```bash
  export PANDAPROBE_API_KEY="<from the PandaProbe dashboard>"
  export PANDAPROBE_PROJECT_NAME="my-first-project"
  ```
- **Provider:** ensure the chosen provider's key is set (e.g. `OPENAI_API_KEY`).
- **Install** the SDK with the extra for the example, plus any provider/framework library
  the example imports (see the extras table in [instrumentation.md](instrumentation.md)):
  ```bash
  pip install "pandaprobe[<extra>]"    # e.g. pandaprobe[openai] (Simple) or pandaprobe[langgraph] (Agent)
  ```

## Step 3 — Fetch docs & implement the example (doc-first)

Fetch the latest relevant docs (SKILL.md section 3), then write a minimal, runnable example
file from them — **not from memory**. Confirm the file with the user before creating it.

- **Simple LLM call** → `tracing/wrappers/overview` + the chosen provider's wrapper page;
  wrap the client and make one call.
- **Agent with a framework** → `tracing/integrations/overview` + the chosen framework's
  page (and `get-started/quickstart`); wire the integration (callback handler or adapter
  `.instrument()`) around a small agent with one tool.

Use [instrumentation.md](instrumentation.md) for the layer overview, but follow the docs for
exact names and wiring.

## Step 4 — Run the example

Run it so it emits a trace; debug logging confirms the SDK is sending data:

```bash
PANDAPROBE_DEBUG=true python example.py
```

If nothing is sent: check `PANDAPROBE_API_KEY` / `PANDAPROBE_PROJECT_NAME` (and the provider
key) are set, `PANDAPROBE_ENABLED` is not `false`, and the endpoint is reachable
(https://docs.pandaprobe.com/tracing/configuration/troubleshooting).

## Step 5 — Install the CLI & verify the trace

Install the CLI to read the data back and confirm the trace landed (details:
[cli.md](cli.md)):

```bash
curl -fsSL https://cli.pandaprobe.com/install.sh | sh   # Windows: irm https://cli.pandaprobe.com/install.ps1 | iex
pandaprobe version
pandaprobe auth login            # Cloud login (add --no-browser on headless/SSH)
pandaprobe traces list --limit 5
pandaprobe traces get <trace_id> # inspect spans, token usage, status
```

Then point the user to the dashboard at **https://app.pandaprobe.com** to see the trace
visually.

## Step 6 — Next steps

Offer where to go from here:

- **Build the other example** (the one not chosen in Step 1).
- **Instrument their real app** — switch to [instrumentation.md](instrumentation.md).
- **Read data & run evaluations** over captured traces — see [cli.md](cli.md).
