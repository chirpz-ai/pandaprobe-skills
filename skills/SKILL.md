---
name: pandaprobe
description: Trace and evaluate AI agents with PandaProbe. Use when the user wants to (1) set up, onboard, or get started with PandaProbe in a project ("set up PandaProbe", "add PandaProbe", "get started with PandaProbe") — a guided, interactive flow that builds and runs a first traced example from scratch and verifies the traces; (2) add tracing or observability to an AI agent or LLM app by instrumenting an agent framework (LangGraph, LangChain, OpenAI Agents, CrewAI, Google ADK, Claude Agent SDK, DeepAgents) or LLM provider (OpenAI, Anthropic, Gemini, Mistral, Bedrock) with the PandaProbe SDK; or (3) install, authenticate, and use the pandaprobe CLI to inspect, debug, or evaluate traces, sessions, spans, scores, and evaluation runs. Instrumentation is done doc-first against the live PandaProbe documentation.
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
agents and LLM apps. This skill supports three goals:

1. **Onboard a project** onto PandaProbe — a guided, interactive setup that builds and runs
   a first traced example from scratch, ideal for an empty or new project (see the next section).
2. **Instrument an application** with the PandaProbe SDK so its runs are traced (section 1);
   see [references/instrumentation.md](references/instrumentation.md).
3. **Read and evaluate data** — traces, sessions, spans, scores, eval runs — with the
   `pandaprobe` CLI (section 2); see [references/cli.md](references/cli.md).

Section 3 is a supporting **method**, not a goal: how to fetch the latest documentation,
which instrumentation always depends on (work doc-first, never from memory).

## Set up PandaProbe (guided onboarding)

When the user says "set up PandaProbe", "add PandaProbe", "get started", or similar —
especially in an empty or new project — run the guided onboarding: **build and run a small
working example** so they see their first traces. Ask which provider and what to build (a
simple traced LLM call, or a small framework agent), implement it doc-first, run it, then
install the CLI to verify the trace and point them to the dashboard. If they instead want to
trace an app they already have, use section 1 (instrument). **Confirm before installing
packages, creating files, or authenticating.**

Follow the step-by-step playbook in [references/setup.md](references/setup.md); it uses the
docs from section 3 and the references in sections 1 (instrument) and 2 (CLI).

## Core Principles

Follow these for ALL PandaProbe work:

1. **Documentation first.** NEVER instrument from memory. The SDK is under active
   development — always fetch the latest docs before writing instrumentation code (section 3).
2. **CLI for data access.** Use the `pandaprobe` CLI to read or evaluate data — never
   hand-build API calls. It is agent-first: JSON on stdout, errors on stderr, meaningful
   exit codes, no interactive prompts (section 2).
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
3. **Fetch the latest docs** for that exact framework/provider (section 3) and confirm the package
   extra, import path, class name, and wiring pattern **before** writing code.
4. **Install, set credentials, wire the chosen layer, then verify** the trace landed with
   the CLI (section 2).

The high-level playbook — layers, install extras, the integration wiring patterns, credentials, and verification — lives in
[references/instrumentation.md](references/instrumentation.md). Always reconcile exact names and signatures with the current docs.

## 2. Read & evaluate data (CLI)

Use the `pandaprobe` CLI to inspect traces, sessions, spans, scores, and evaluation runs.
Reads (`list`/`get`/`spans`/`metrics`) are safe to run freely; the three eval **write**
commands — `evals runs create`, `evals runs batch`, `evals scores submit` — create data, so
run them only when the user explicitly asks.

Install, authentication (Cloud and self-hosted), the full command tree, output contract,
exit codes, and `jq` recipes are in [references/cli.md](references/cli.md) — read it before
running commands. For anything beyond it, run `pandaprobe <command> --help` or see
https://docs.pandaprobe.com/tools/cli.

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
