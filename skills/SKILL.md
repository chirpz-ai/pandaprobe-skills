---
name: pandaprobe
description: Interact with PandaProbe and access its documentation. Use when needing to (1) look up PandaProbe documentation, concepts, and integration guides for SDK usage, (2) instrument and trace an AI agent or LLM app, (3) inspect traces, sessions, spans, scores, or evaluation runs via the pandaprobe CLI, or (4) install, set up, and authenticate the pandaprobe CLI. Covers SDK instrumentation, CLI-based read/eval access, and documentation retrieval.
allowed-tools:
  - Bash(pandaprobe *)
  - Bash(curl -fsSL https://cli.pandaprobe.com/*)
  - Bash(irm https://cli.pandaprobe.com/*)
  - Bash(go install github.com/chirpz-ai/pandaprobe-cli*)
  - Bash(curl *docs.pandaprobe.com/*)
  - WebFetch(domain:docs.pandaprobe.com)
---

# PandaProbe

PandaProbe is an agent engineering platform for tracing, evaluating, and monitoring AI
agents and LLM apps. This skill supports two primary goals:

1. **Instrument an agent application** with the PandaProbe SDK so its runs are traced
   (§1) — work doc-first; see [references/instrumentation.md](references/instrumentation.md).
2. **Read and evaluate data** — traces, sessions, spans, scores, eval runs — with the
   `pandaprobe` CLI (§2); see [references/cli.md](references/cli.md).

§3 explains how to fetch the latest documentation, which §1 depends on.

## Core Principles

Follow these for ALL PandaProbe work:

1. **Documentation first.** NEVER instrument from memory. The SDK is under active
   development — always fetch the latest docs before writing instrumentation code (§3).
2. **CLI for data access.** Use the `pandaprobe` CLI to read or evaluate data — never
   hand-build API calls. It is agent-first: JSON on stdout, errors on stderr, meaningful
   exit codes, no interactive prompts (§2).
3. **Best practices by use case.** Read the relevant reference file before implementing —
   instrumentation → `references/instrumentation.md`; CLI → `references/cli.md`.
4. **Use the latest versions.** Unless the user specifies otherwise, use the latest
   PandaProbe SDK and CLI.
5. **Never echo the API key.** Don't print, log, or paste keys into chat; let the CLI and
   SDK manage credentials (`pandaprobe config show` masks the key).

## 1. Instrument an agent application (SDK)

Add PandaProbe tracing to the user's app. The SDK changes often and exact APIs differ per
framework, so **work doc-first — do not instrument from memory.** Keep your reasoning
high-level here and pull specifics from the docs.

Procedure:

1. **Identify the stack** — which agent framework (LangGraph, LangChain, OpenAI Agents,
   CrewAI, Google ADK, Claude Agent SDK, DeepAgents) or LLM provider (OpenAI, Anthropic,
   Gemini, Mistral, Bedrock) the app uses.
2. **Choose the layer** (priority order): agent-framework integration → provider wrapper →
   manual decorators. Prefer the highest that fits; layers compose.
3. **Fetch the latest docs** for that exact framework/provider (§3) and confirm the package
   extra, import path, class name, and wiring pattern **before** writing code.
4. **Install, set credentials, wire the chosen layer, then verify** the trace landed with
   the CLI (§2).

The high-level playbook — layers, install extras, the integration wiring patterns, credentials, and verification — lives in
[references/instrumentation.md](references/instrumentation.md). Always reconcile exact names and signatures with the current docs.

## 2. Read & evaluate data (CLI)

Use the `pandaprobe` CLI from the terminal. Install (macOS/Linux) and verify:

```bash
curl -fsSL https://cli.pandaprobe.com/install.sh | sh   # Windows: irm https://cli.pandaprobe.com/install.ps1 | iex
pandaprobe version
```

Discover commands and flags before running them:

```bash
pandaprobe --help              # top-level command groups
pandaprobe traces --help       # actions in a group
pandaprobe traces list --help  # flags for one action
```

**Authenticate:**

- **Cloud (recommended):** `pandaprobe auth login` opens the browser (add `--no-browser`
  for headless/SSH), mints a 90-day key, and writes `~/.pandaprobe/config.yaml`.
- **Self-hosted / manual:** `pandaprobe config set endpoint|project_name|api_key <value>`,
  or `PANDAPROBE_*` env vars. If credentials are missing, ask the user to run `auth login`
  or set them in their shell; never ask them to paste a key into chat.

**Read vs. write:** `list`/`get`/`spans`/`metrics` are read-only. `evals runs create`,
`evals runs batch`, and `evals scores submit` **create data** — run them only when the user
explicitly asks. Parse JSON output with `jq` and branch on exit codes (`0` ok ·
`1` general/network · `2` auth · `3` not found · `4` validation · `5` other API error).

The full command tree, flags, output contract, exit codes, and `jq` recipes are in
[references/cli.md](references/cli.md). For anything not covered there, run
`pandaprobe <command> --help` or see the docs: https://docs.pandaprobe.com/tools/cli.

## 3. Access PandaProbe documentation

All product docs live at **docs.pandaprobe.com**. **Prefer your application's native fetch
tools** (e.g. `WebFetch`) over `curl` when available — the URLs below work with any method.

**a. Documentation index (llms.txt)** — every page with title, URL, and one-line summary;
use it to find the right page for a topic:

```bash
curl -s https://docs.pandaprobe.com/llms.txt
```

**b. Fetch a page as markdown** — append `.md` to its path (or send `Accept: text/markdown`):

```bash
curl -s "https://docs.pandaprobe.com/tracing/integrations/langgraph.md"
```

Key instrumentation pages: `tracing/overview`, `tracing/concepts`,
`tracing/integrations/overview` (+ the per-framework page), `tracing/wrappers/overview`,
`tracing/manual/decorators`, `tracing/configuration/environment-variables`. CLI: `tools/cli`.

**Workflow:** start at llms.txt to orient → fetch the specific `.md` page → verify exact
names and signatures against it before writing code, and adopt the latest SDK version.
