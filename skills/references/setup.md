---
name: pandaprobe-setup
description: Guided end-to-end onboarding for PandaProbe — take a project from zero to a verified trace. Use when the user asks to "set up PandaProbe", "add PandaProbe to my project", "get started with PandaProbe", or "onboard PandaProbe". Sequences SDK install/config and app instrumentation first, then CLI install/auth and trace verification.
---

# Set up PandaProbe (guided onboarding)

Goal: get the user from zero to a **verified trace from their own app**, then point them to
the dashboard. This is an orchestrator — it sequences the instrumentation
([instrumentation.md](instrumentation.md)) and CLI ([cli.md](cli.md)) references and the
live docs: instrument and trace the app first, then bring in the CLI to read the data back.

**Ground rules**

- **Confirm before side effects.** Detect first and propose a short plan. Get explicit
  confirmation before installing packages, authenticating, or editing the user's code.
- **Doc-first instrumentation.** Don't write tracing code from memory — fetch the latest
  docs for the user's exact framework/provider (see SKILL.md section 3).
- **Never echo the API key.** Have the user set credentials themselves; don't print, log,
  or paste keys.

## Step 0 — Detect & plan

Inspect the project and summarize what you find, then propose the plan and ask to proceed.

- **Language / runtime** — is this a Python app? (PandaProbe's SDK is Python - we do not have TS SDK at this point.)
- **Stack** — which agent framework or LLM provider is in use? Check dependency files
  (`pyproject.toml`, `requirements.txt`, `uv.lock`) and imports for: LangGraph, LangChain,
  OpenAI Agents, CrewAI, Google ADK, Claude Agent SDK, DeepAgents; or OpenAI, Anthropic,
  Gemini, Mistral, Bedrock.
- **Entry point** — where the agent/LLM run starts (the function to wrap or the
  graph/runner to attach to).
- **Already set up?** — is `pandaprobe` already installed, or `PANDAPROBE_*` env vars set?

State the detected stack and the proposed layer (integration → wrapper → manual, highest
that fits), then confirm before continuing.

## Step 1 — Install the SDK

Install the SDK with the extra for the detected stack (confirm before running). See the
extras table in [instrumentation.md](instrumentation.md):

```bash
pip install "pandaprobe[<extra>]"    # e.g. pandaprobe[langgraph] or pandaprobe[openai]
```

## Step 2 — Configure SDK credentials

The SDK reads credentials from environment variables. Ask the user to set these in their
shell or `.env` — **do not paste the key yourself** (they copy it from the PandaProbe
dashboard at https://app.pandaprobe.com):

```bash
export PANDAPROBE_API_KEY="<from the PandaProbe dashboard>"
export PANDAPROBE_PROJECT_NAME="my-project"
```

Confirm they're set (the app can read them) before instrumenting.

## Step 3 — Instrument the app

This is the core step. Follow [instrumentation.md](instrumentation.md) for the chosen layer,
and **fetch the exact framework/provider page** (SKILL.md section 3) to confirm import paths,
class names, and wiring **before** editing code. Apply the minimal change at the entry
point — attach the integration handler/adapter, or wrap the provider client — and confirm
the diff with the user before writing it.

## Step 4 — Run the app to produce a trace

Run the instrumented app so it emits a trace. Enable debug logging to confirm the SDK is
sending data:

```bash
PANDAPROBE_DEBUG=true python your_app.py   # debug logs show traces being sent
```

If nothing is sent: check `PANDAPROBE_API_KEY` / `PANDAPROBE_PROJECT_NAME` are set,
`PANDAPROBE_ENABLED` is not `false`, and the endpoint is reachable
(https://docs.pandaprobe.com/tracing/configuration/troubleshooting).

## Step 5 — Install the CLI & verify the trace

Now install the CLI to read the data back and confirm the trace landed (details:
[cli.md](cli.md)):

```bash
curl -fsSL https://cli.pandaprobe.com/install.sh | sh   # Windows: irm https://cli.pandaprobe.com/install.ps1 | iex
pandaprobe version
pandaprobe auth login            # Cloud login (add --no-browser on headless/SSH)
pandaprobe traces list --limit 5
pandaprobe traces get <trace_id> # inspect spans, token usage, status
```

For self-hosted, skip `auth login` and set `endpoint` + `api_key` via `pandaprobe config set`.
The CLI is also the tool for ongoing work — reading traces/sessions/scores and running
evaluations (see [cli.md](cli.md)).

## Step 6 — Done

Confirm success and point the user to the dashboard at **https://app.pandaprobe.com** to
view their traces. Suggest next steps: grouping runs into sessions, and running evaluations
over captured traces (see [cli.md](cli.md) → Evaluations).
