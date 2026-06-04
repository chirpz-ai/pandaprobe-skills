---
name: pandaprobe-cli
description: Install, authenticate, configure, and operate the pandaprobe CLI to read and evaluate data. Use when inspecting traces, sessions, spans, scores, or evaluation runs from the terminal, parsing JSON output with jq and branching on exit codes, or creating eval runs and submitting scores.
---

# PandaProbe CLI Reference

Operate the `pandaprobe` CLI to read traces, sessions, spans, scores, and evaluation runs.
The CLI is agent-first: **JSON on stdout, errors on stderr, no interactive prompts,
meaningful exit codes.** Parse output with `jq` and branch on exit codes.

Official docs: https://docs.pandaprobe.com/tools/cli

## Install

```bash
# macOS / Linux
curl -fsSL https://cli.pandaprobe.com/install.sh | sh

# Windows (PowerShell)
irm https://cli.pandaprobe.com/install.ps1 | iex

# From source (requires Go)
go install github.com/chirpz-ai/pandaprobe-cli@latest
```

The binary is `pandaprobe`. Verify:

```bash
pandaprobe version
```

## Discover

```bash
pandaprobe --help              # top-level command groups
pandaprobe traces --help       # actions in a group
pandaprobe traces list --help  # flags for one action
```

## Authenticate

Two paths:

**Cloud (recommended)** — `auth login` opens the browser, mints a 90-day API key, and
writes `api_key` + `project_name` to `~/.pandaprobe/config.yaml`:

```bash
pandaprobe auth login              # browser-based login (Cloud only)
pandaprobe auth login --no-browser # headless/SSH: prints the URL instead
pandaprobe auth status             # confirm login (key masked)
pandaprobe auth logout             # remove stored credentials
```

**Local / self-hosted** (auth disabled or manual key) — set values yourself. **Never
print the key.**

```bash
pandaprobe config set endpoint http://localhost:8000
pandaprobe config set project_name my-project
pandaprobe config set api_key <key>   # only if the deployment requires one
```

## Configuration & precedence

Values resolve highest → lowest: **flags > `PANDAPROBE_*` env > `~/.pandaprobe/config.yaml`
> defaults.**

| Setting       | Flag         | Env var                   | Config key     | Default                      |
| ------------- | ------------ | ------------------------- | -------------- | ---------------------------- |
| API key       | `--api-key`  | `PANDAPROBE_API_KEY`      | `api_key`      | —                            |
| Project name  | `--project`  | `PANDAPROBE_PROJECT_NAME` | `project_name` | —                            |
| Endpoint      | `--endpoint` | `PANDAPROBE_ENDPOINT`     | `endpoint`     | `https://api.pandaprobe.com` |
| Web app URL   | `--auth-url` | `PANDAPROBE_AUTH_URL`     | `auth_url`     | `https://app.pandaprobe.com` |
| Output format | `--format`   | `PANDAPROBE_FORMAT`       | `format`       | `json`                       |
| Timeout (sec) | —            | `PANDAPROBE_TIMEOUT`      | `timeout`      | `30`                         |

Inspect the effective config (API key masked):

```bash
pandaprobe config show
pandaprobe config get endpoint
pandaprobe config path
```

## Output contract

JSON by default → **data to stdout, errors to stderr**, so output pipes cleanly into `jq`.
Use `--format table` only for human display.

List responses wrap items with pagination:

```json
{ "items": [ /* ... */ ], "pagination": { "total": 150, "limit": 20, "offset": 0 } }
```

Errors are JSON on stderr:

```json
{ "error": { "code": "validation_error", "message": "...", "status": 422, "request_id": "...", "details": {} } }
```

Global flags: `--verbose`, `--debug` (logs HTTP to stderr, key masked), `--no-color`,
`--config <path>`.

## Exit codes (branch on these)

| Code | Meaning                                       |
| ---- | --------------------------------------------- |
| `0`  | Success                                       |
| `1`  | General error (network, decode, unexpected)   |
| `2`  | Authentication / authorization (401, 403)     |
| `3`  | Not found (404)                               |
| `4`  | Validation (bad flags, 400, 422)             |
| `5`  | Other API error (other 4xx, 5xx)             |

Pagination everywhere: `--limit` (1–200) and `--offset`. Filtering is server-side.

## Command reference

### Traces (read-only)

```bash
pandaprobe traces list --status ERROR --sort-by started_at --sort-order desc --limit 20
pandaprobe traces get <trace_id>
pandaprobe traces spans <trace_id> --kind LLM --status ERROR
```

- `traces list` — filters: `--status` (`PENDING|RUNNING|COMPLETED|ERROR`), `--session-id`,
  `--user-id`, `--name`, `--tags`, `--started-after`, `--started-before`, `--sort-by`
  (`started_at|ended_at|name|latency|status`), `--sort-order` (`asc|desc`), `--limit`
  (1–200), `--offset`.
- `traces get <trace_id>` — full trace with inline `spans`. `--spans-only` returns just the
  spans array; `--kind` / `--status` filter spans client-side.
- `traces spans <trace_id>` — `--kind` (`AGENT|TOOL|LLM|RETRIEVER|CHAIN|EMBEDDING|OTHER`),
  `--status` (`OK|ERROR|UNSET`).

```json
{ "items": [ { "trace_id": "tr_123", "name": "support-agent", "status": "ERROR", "started_at": "..." } ],
  "pagination": { "total": 42, "limit": 20, "offset": 0 } }
```

### Sessions (read-only)

```bash
pandaprobe sessions list --has-error --sort-by recent --limit 20
pandaprobe sessions get <session_id> --include-traces
```

- `sessions list` — `--user-id`, `--has-error`, `--started-after`, `--started-before`,
  `--tags`, `--query`, `--sort-by` (`recent|trace_count|latency|cost`), `--sort-order`,
  `--limit`, `--offset`.
- `sessions get <session_id>` — `--include-traces` (default `true`), `--limit`, `--offset`.

### Evaluations

Target traces or sessions with `--target trace|session` (default `trace`). There are multiple read commands
along with three write commands that are flagged below.

**Read:**

```bash
pandaprobe evals metrics --target trace
pandaprobe evals runs list --target trace --status COMPLETED --limit 20
pandaprobe evals runs get <run_id> --target trace
pandaprobe evals runs scores <run_id> --target trace
pandaprobe evals scores list --target trace --name coherence --source AUTOMATED
pandaprobe evals scores get <trace_id> --target trace
```

- `evals metrics` — `--target trace|session` (default `trace`).
- `evals runs list` — `--target`, `--status` (`PENDING|RUNNING|COMPLETED|FAILED`),
  `--limit`, `--offset`.
- `evals runs get <run_id>` / `evals runs scores <run_id>` — `--target`.
- `evals scores list` — `--target`; trace filters `--trace-id`, `--name`, `--source`
  (`AUTOMATED|ANNOTATION|PROGRAMMATIC`), `--status` (`SUCCESS|FAILED|PENDING`),
  `--data-type` (`NUMERIC|BOOLEAN|CATEGORICAL`), `--eval-run-id`, `--environment`,
  `--date-from`, `--date-to`; session uses `--session-id`; `--limit`, `--offset`.
- `evals scores get <trace_id|session_id>` — `--target`.

**Write — run only when the user explicitly asks or you need to evaluate an agent:**

```bash
# Run metrics over traces matching filters
pandaprobe evals runs create --target trace --metrics coherence,tool_correctness --status COMPLETED

# Run metrics over an explicit set of traces/sessions
pandaprobe evals runs batch --target trace --trace-ids <id1>,<id2> --metrics coherence

# Submit a manual score for a trace (trace target only)
pandaprobe evals scores submit --trace-id <trace_id> --name accuracy --value 0.92
```

- `evals runs create` — `--target`, `--metrics m1,m2` (required), `--sampling-rate`,
  `--model`, `--name`. Trace filters: `--date-from`, `--date-to`, `--status`,
  `--session-id`, `--user-id`, `--tags`, `--filter-name`. Session filters: `--user-id`,
  `--has-error`, `--tags`, `--min-trace-count`, `--signal-weights`. **(write op)**
- `evals runs batch` — `--target`, `--trace-ids`/`--session-ids` (required), `--metrics`
  (required), `--name`, `--model`. **(write op)**
- `evals scores submit` — `--trace-id`, `--name`, `--value` (required); optional
  `--data-type`, `--source`, `--reason`, `--metadata`. **Trace-only**; `--target session`
  errors. **(write op)**

### Utility

```bash
pandaprobe version
pandaprobe completion zsh   # also bash | fish | powershell
pandaprobe config show      # set | get | show | path
```

## Recipes

`list`/`get` are read-only and safe to run freely. The recipes below only read data.

```bash
# 1. Overview: count traces by status
pandaprobe traces list --limit 200 \
  | jq '[.items[].status] | group_by(.) | map({status: .[0], count: length})'

# 2. Find a failed trace, then read the failing span's error
ID=$(pandaprobe traces list --status ERROR --limit 1 | jq -r '.items[0].trace_id')
pandaprobe traces get "$ID" | jq '.spans[] | select(.status == "ERROR") | {name, kind, error}'

# 3. Eval-run drill-down: list -> get -> scores
RUN=$(pandaprobe evals runs list --status COMPLETED --limit 1 | jq -r '.items[0].id')
pandaprobe evals runs get "$RUN"
pandaprobe evals runs scores "$RUN" | jq '.items[] | {name, value, data_type}'

# 4. Per-trace scores
pandaprobe evals scores get "$ID" | jq '.[] | {name, value, data_type, source}'
```
