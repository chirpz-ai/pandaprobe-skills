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
agents and LLM apps. This skill helps you use it across common workflows: instrumenting
applications with the SDK, reading and evaluating data via the CLI, and looking up
PandaProbe documentation.

## Core Principles

Follow these principles for ALL PandaProbe work:

1. **Documentation First**: NEVER implement instrumentation based on memory. The SDK
   evolves — always fetch current docs before writing code. See the section below on how
   to access documentation.
2. **CLI for Data Access**: Use the `pandaprobe-cli` when reading or evaluating PandaProbe
   data (traces, sessions, spans, scores, eval runs) — never hand-build API calls. It is
   agent-first: JSON on stdout, errors on stderr, no interactive prompts. See the section
   below on how to use the CLI.
3. **Best Practices by Use Case**: Check the relevant reference file below for
   use-case-specific guidance before implementing.
4. **Use latest PandaProbe versions**: Unless the user specifies otherwise or there's a
   good reason, always use the latest version of the PandaProbe SDK and CLI.
5. **Never echo the API key**: Don't print, log, or paste keys into chat. Let the CLI and
   SDK manage credentials; use `pandaprobe config show` (key is masked) to inspect config.

## Use case specific references

- instrumenting an AI agent or LLM app (provider wrappers, agent-framework integrations,
  manual decorators): [references/instrumentation.md](references/instrumentation.md)
- reading and evaluating data with the CLI, plus install/auth/config (traces, sessions,
  spans, scores, eval runs): [references/cli.md](references/cli.md)

## 1. PandaProbe CLI (data access)

Use the `pandaprobe-cli` to read and evaluate data from the terminal. Install it
(macOS/Linux) and verify:

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

### Credentials

- **Cloud (recommended):** `pandaprobe auth login` opens the browser, mints a 90-day key,
  and writes `~/.pandaprobe/config.yaml`. Add `--no-browser` for headless/SSH.
- **Self-hosted / manual:** set values yourself — `pandaprobe config set endpoint <url>`,
  `config set project_name <name>`, `config set api_key <key>` — or via `PANDAPROBE_*`
  env vars. If credentials are missing, ask the user to run `auth login` or set them in
  their shell; never ask them to paste a key into chat.

### Read vs. Write commands

`list`/`get`/`spans`/`metrics` are read-only. `evals runs create`,
`evals runs batch`, and `evals scores submit` **create evaluation runs** — run them only when the
user explicitly asks. Parse JSON output with `jq` and branch on exit codes (`0` ok ·
`1` general/network · `2` auth · `3` not found · `4` validation · `5` other API error).

### Detailed CLI reference

For install paths, full command tree, exit codes, output contract, and `jq` recipes, see
[references/cli.md](references/cli.md).

## 2. PandaProbe Documentation

All product docs live at **docs.pandaprobe.com**. **Always prefer your application's native
fetch tools** (e.g. `WebFetch`) over `curl` when available — the URLs below work with any
method.

### 2a. Documentation Index (llms.txt)

Fetch the full index of every documentation page (titles, URLs, and one-line summaries):

```bash
curl -s https://langfuse.com/llms.txt
```

Use it to discover the right page for a topic, then fetch that page directly. You can also
start at `https://docs.pandaprobe.com` and browse.

### 2b. Fetch Individual Pages as Markdown

Any docs page can be fetched as clean markdown (code samples and config included) by
appending `.md` to its path, or via an `Accept: text/markdown` header:

```bash
curl -s "https://docs.pandaprobe.com/tracing/overview.md"
curl -s "https://docs.pandaprobe.com/tracing/overview" -H "Accept: text/markdown"
```

Key pages for instrumentation: `tracing/overview`, `tracing/concepts`,
`tracing/wrappers/overview`, `tracing/integrations/overview`, `tracing/manual/decorators`,
`tracing/configuration/environment-variables`. CLI: `tools/cli`.

### Documentation Workflow

1. Start with **llms.txt** to orient — scan for relevant page titles.
2. **Fetch the specific page(s)** as `.md` once you've identified the right one.
3. Verify SDK names and signatures against the page before writing code, and adopt the
   latest SDK version.
