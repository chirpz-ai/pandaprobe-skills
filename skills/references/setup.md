---
name: pandaprobe-setup
description: Guided, interactive onboarding for PandaProbe — build and run a small working agent example from scratch so a new user sees their first traces fast. Use when the user asks to "set up PandaProbe", or "get started with PandaProbe", and is starting from an empty or new project. Asks which provider and agent framework, implements the example doc-first, runs it, then verifies traces with the CLI.
---

# Set up PandaProbe (guided onboarding)

Goal: take a new user **starting from an empty or new project** from zero to their **first
traces** by building and running a small, working agent example *for* them, then pointing
them to next steps. This path is strictly for onboarding from scratch — it favors a runnable
example over theory.

**If the user already has an application and wants to wire tracing into it, this is the
wrong path** — do not use this file. Switch to [instrumentation.md](instrumentation.md)
(SKILL.md section 2), which covers instrumenting an existing app.

This is an orchestrator: it makes the choices interactive, implements the example **doc-first**
(from [instrumentation.md](instrumentation.md) + the live docs), and verifies with the CLI
([cli.md](cli.md)).

**Ground rules**

- **Onboarding from scratch only.** Build a fresh example in a new/empty project. The moment
  the task becomes "instrument my existing code," hand off to [instrumentation.md](instrumentation.md).
- **Be interactive.** Ask the user the questions in Step 1 and wait for answers — don't
  assume. Keep it friendly and concrete.
- **Confirm before side effects.** Get explicit confirmation before installing packages,
  creating files, or authenticating.
- **Doc-first — never from memory.** Fetch the latest docs for the chosen provider/framework
  and implement the example from them (exact imports, integration wiring). The SDK changes often.

## Step 0 — Confirm the starting point

Confirm this is a from-scratch onboarding, not an existing app:

- **Empty/new project, or the user just wants a guided first example** → continue here.
- **They already have an app to trace** → stop and switch to [instrumentation.md](instrumentation.md).

Confirm Python 3.10+ is available (PandaProbe's SDK is Python; there is no TS SDK yet), and
pick/confirm a working directory (create one and a virtualenv if helpful).

## Step 1 — Ask what to build (interactive)

Ask the user two questions and wait for answers:

1. **Which model provider** do they want to use — OpenAI, Anthropic, Gemini, Mistral, or
   Bedrock? (They'll need that provider's API key.)
2. **Which agent framework** would they like to build with — LangGraph, LangChain, OpenAI
   Agents, CrewAI, Google ADK, or Claude Agent SDK? Recommend one that pairs well with their
   chosen provider and confirm.

You'll build a **small agent with one tool**, traced via that framework's integration, so
they see a multi-span trace (agent + LLM + tool). Summarize the plan (provider + framework +
packages you'll install) and confirm before proceeding.

## Step 2 — Install & authenticate the CLI, then set credentials

Confirm before installing. Credentials go in a `.env` file (not shell `export`).

**a. Install and authenticate the CLI.** This signs the user in and writes their `api_key`
and `project_name` to `~/.pandaprobe/config.yaml`:

```bash
curl -fsSL https://cli.pandaprobe.com/install.sh | sh   # Windows: irm https://cli.pandaprobe.com/install.ps1 | iex
pandaprobe version
pandaprobe auth login        # opens the browser; add --no-browser on headless/SSH
```

**b. Reuse those credentials for the SDK — copy the RAW values.** After login, the CLI
stores the **raw** `api_key` and `project_name` in `~/.pandaprobe/config.yaml` (find it with
`pandaprobe config path`). Open that file, read the raw values, and write them into a `.env`
file in the project root in exactly this format:

```dotenv
PANDAPROBE_API_KEY=<raw value>
PANDAPROBE_PROJECT_NAME=<raw value>
```

**Use the raw, unmasked key** exactly as stored in `config.yaml`. Do **not** use the masked
form (e.g. `sk_pp_****43dc`) printed by `pandaprobe config show` or `pandaprobe config get` —
a masked value will not authenticate. Read it from the config file, not from `config get`.

**c. Add the provider key.** The only credential the user must supply is their LLM provider
key. Ask which way they'd prefer to add it:

- **a) Paste in chat — you add it.** The user pastes their provider API key in the chat and
  you write it into `.env` yourself, using the correct variable name for the provider.
- **b) The user adds it manually.** prompt them to put the credential in `.env` (the right
  variable name for their provider) and wait for them to confirm it's set.

Either way `.env` ends up with the provider key, e.g.:

```dotenv
OPENAI_API_KEY=<provider key>   # variable name depends on the provider
# PANDAPROBE_DEBUG=true   # optional: log traces being sent
```

**d. Ignore secrets in git.** Create or update `.gitignore` in the project root and add
`.env` so the credentials are never committed:

```gitignore
.env
```

**e. Install the SDK** with the extra for the chosen framework, plus `python-dotenv` to load
`.env` (and any provider/framework library the example imports; see the extras table in
[instrumentation.md](instrumentation.md)):

```bash
pip install "pandaprobe[<framework-extra>]" python-dotenv   # e.g. "pandaprobe[langgraph]"
```

## Step 3 — Fetch docs & implement the example (doc-first)

Fetch the latest relevant docs (SKILL.md section 4), then write a minimal, runnable agent
example from them — **not from memory**. Confirm the file with the user before creating it.

- Fetch `tracing/integrations/overview` + the chosen framework's page (and
  `get-started/quickstart`), then wire the integration (callback handler or adapter
  `.instrument()`) around a small agent with one tool.
- Load the `.env` **before** the PandaProbe client initializes (the SDK reads env vars at
  init), e.g. call `load_dotenv()` at the very top of the file.

Use [instrumentation.md](instrumentation.md) for the layer overview, but follow the docs for
exact names and wiring.

## Step 4 — Run the example

Run it so it emits a trace (it loads credentials from `.env`):

```bash
python example.py
```

Set `PANDAPROBE_DEBUG=true` in `.env` to log traces being sent. If nothing is sent: check
`PANDAPROBE_API_KEY` / `PANDAPROBE_PROJECT_NAME` (and the provider key) are present in
`.env` and loaded, `PANDAPROBE_ENABLED` is not `false`, and the endpoint is reachable
(https://docs.pandaprobe.com/tracing/configuration/troubleshooting).

## Step 5 — Verify the trace

The CLI is already installed and authenticated (Step 2), so just confirm the trace landed
(details: [cli.md](cli.md)):

```bash
pandaprobe traces list --limit 5
pandaprobe traces get <trace_id> # inspect spans, token usage, status
```

Then point the user to the dashboard at **https://app.pandaprobe.com** to see the trace
visually.

## Step 6 — Next steps

Offer where to go from here:

- **Build another agent example** with a different framework or provider.
- **Instrument their real app** — switch to [instrumentation.md](instrumentation.md).
- **Read data & run evaluations** over captured traces — see [cli.md](cli.md).
