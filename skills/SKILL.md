---
name: pandaprobe
description: Trace and evaluate AI agents with PandaProbe. Use when needing to (1) instrument and trace an AI agent or LLM app, (2) inspect traces, sessions, spans, scores, or evaluation runs via the pandaprobe CLI, or (3) install, setup and authenticate the pandaprobe CLI. Covers SDK instrumentation and CLI-based read/write access.
allowed-tools:
  - Bash(pandaprobe *)
  - Bash(curl -fsSL https://cli.pandaprobe.com/*)
  - WebFetch(domain:pandaprobe.com)
---

# PandaProbe

PandaProbe is an agent engineering platform for tracing, evaluating, and monitoring AI
agents and LLM apps. This skill helps you drive its **agent-first CLI** (read traces,
sessions, spans, scores, and evaluation runs) and add its **SDK instrumentation**
to a developer's application.

## Core principles

Follow these principles for ALL PandaProbe work:

1. **CLI for data access.** To read or evaluate PandaProbe data, use the `pandaprobe`
   CLI — never hand-build API calls. It is agent-first: JSON on stdout, errors on stderr,
   no interactive prompts.
2. **Parse JSON, branch on exit codes.** Output is JSON by default — pipe it through `jq`.
   Branch on exit codes (`0` ok · `1` general/network · `2` auth · `3` not found ·
   `4` validation · `5` other API error), not on string matching.
3. **Reads are safe; some eval commands write.** `list`/`get`/`spans`/`metrics` are
   read-only. `evals runs create`, `evals runs batch`, and `evals scores submit` **create
   data** — run them only when the user explicitly asks.
4. **Never echo the API key.** Don't print, log, or paste keys into chat. Let the CLI
   manage credentials; use `pandaprobe config show` (key is masked) to inspect config.
5. **Fetch current docs before writing SDK code.** The SDK evolves — always fetch the
   latest official docs and use the latest SDK version before instrumenting. Do not rely
   on memory for SDK APIs.

## Routing

- **To install, authenticate, or run CLI commands** (traces, sessions, spans, scores,
  eval runs) → read [`references/cli.md`](references/cli.md).
- **To add PandaProbe tracing to an application** (wrappers, framework integrations, or
  manual decorators) → read [`references/instrumentation.md`](references/instrumentation.md).
