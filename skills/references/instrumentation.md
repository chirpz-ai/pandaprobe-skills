# PandaProbe SDK Instrumentation

Add PandaProbe tracing to a developer's Python application. PandaProbe captures execution
as **traces** (one logical run) composed of **spans** (steps within a run).

## Fetch current docs first

**The SDK evolves — never instrument from memory.** Before writing any code, fetch the
latest official docs and adopt the latest SDK version. Treat every snippet below as a
starting point and **verify it against the current docs**:

- Index (discover all pages): https://docs.pandaprobe.com/llms.txt
- Overview: https://docs.pandaprobe.com/tracing/overview
- Concepts (data model): https://docs.pandaprobe.com/tracing/concepts
- Wrappers (LLM providers): https://docs.pandaprobe.com/tracing/wrappers/overview
- Integrations (agent frameworks): https://docs.pandaprobe.com/tracing/integrations/overview
- Manual decorators: https://docs.pandaprobe.com/tracing/manual/decorators

Append `.md` to any docs URL to fetch clean markdown (e.g. `…/tracing/overview.md`).

## Install

```bash
pip install pandaprobe        # Python 3.10+ (or: uv add pandaprobe)
```

Install only the extras for the providers/frameworks in use:

| Extra                                  | Covers                            |
| -------------------------------------- | --------------------------------- |
| `pandaprobe[openai]`                   | OpenAI wrapper                    |
| `pandaprobe[anthropic]`                | Anthropic wrapper                 |
| `pandaprobe[gemini]`                   | Google Gemini wrapper            |
| `pandaprobe[mistral]`                  | Mistral wrapper                  |
| `pandaprobe[bedrock]` *(beta)*         | AWS Bedrock wrapper              |
| `pandaprobe[langgraph]` / `[langchain]` / `[deepagents]` | LangChain-family integrations |
| `pandaprobe[google-adk]`               | Google ADK integration            |
| `pandaprobe[claude-agent-sdk]`         | Claude Agent SDK integration      |
| `pandaprobe[crewai]`                   | CrewAI integration                |
| `pandaprobe[openai-agents]`            | OpenAI Agents SDK integration     |

Combine extras: `pip install "pandaprobe[openai,anthropic,langgraph]"`.

## Credentials

Set via environment variables (read at client init). **Never hardcode or echo the key.**

```bash
export PANDAPROBE_API_KEY="your-api-key"
export PANDAPROBE_PROJECT_NAME="my-project"
# Optional:
export PANDAPROBE_ENDPOINT="https://api.pandaprobe.com"   # change for self-hosted
export PANDAPROBE_ENVIRONMENT="production"                # tag traces
export PANDAPROBE_ENABLED="true"                          # set false to no-op the SDK
```

The SDK auto-initializes from the environment. You can also configure programmatically with
`pandaprobe.init()` — see https://docs.pandaprobe.com/tracing/configuration/project-configuration.

## Three layers of instrumentation

Pick the lightest layer that fits. Verify all names against current docs.

### Layer 1 — LLM provider wrappers (zero-code)

Wrap the client once; every call is traced. Returns the same client type.

```python
# verify against current docs
from pandaprobe.wrappers import wrap_openai
from openai import OpenAI

client = wrap_openai(OpenAI())   # use exactly as before — all calls now traced
```

Available wrappers: `wrap_openai`, `wrap_anthropic`, `wrap_gemini`, `wrap_mistral`,
`wrap_bedrock` (beta). Details: https://docs.pandaprobe.com/tracing/wrappers/overview

### Layer 2 — Agent framework integrations

Hook into a framework to trace the full lifecycle (LLM calls, tools, sub-agent handoffs).
All share constructor params `session_id`, `user_id`, `tags`, `metadata`, and produce
properly nested spans automatically.

| Framework        | Class                       |
| ---------------- | --------------------------- |
| LangGraph        | `LangGraphCallbackHandler`  |
| LangChain        | `LangChainCallbackHandler`  |
| DeepAgents       | `DeepAgentsCallbackHandler` |
| Google ADK       | `GoogleADKAdapter`          |
| Claude Agent SDK | `ClaudeAgentSDKAdapter`     |
| CrewAI           | `CrewAIAdapter`             |
| OpenAI Agents    | `OpenAIAgentsAdapter`       |

```python
# verify against current docs — wiring differs per framework
from pandaprobe.integrations import LangGraphCallbackHandler

handler = LangGraphCallbackHandler(session_id="s-1")
graph.invoke({"messages": [...]}, config={"callbacks": [handler]})
```

Per-framework setup: https://docs.pandaprobe.com/tracing/integrations/overview

### Layer 3 — Manual instrumentation (full control)

Use decorators on functions, or context managers for fine-grained control.

```python
# verify against current docs
import pandaprobe

@pandaprobe.trace(name="support-agent")          # top-level entry point -> a trace
def handle_request(query: str) -> str:
    docs = retrieve_docs(query)
    return generate_answer(query, docs)

@pandaprobe.span(name="retrieve", kind="RETRIEVER")   # inner step -> a span
def retrieve_docs(query: str) -> list[str]:
    ...

@pandaprobe.span(name="generate", kind="LLM", model="gpt-5.4")
def generate_answer(query: str, docs: list[str]) -> str:
    ...
```

Both decorators auto-detect sync/async and capture inputs/outputs. `@pandaprobe.span`
requires an active trace context. Context managers `pandaprobe.start_trace()` and
`t.span()` cover cases decorators don't fit. Span kinds: `LLM`, `TOOL`, `AGENT`, `CHAIN`,
`RETRIEVER`, `EMBEDDING`, `OTHER`. Details:
https://docs.pandaprobe.com/tracing/manual/decorators

Layers compose: a wrapper call inside an `@pandaprobe.trace` (or integration) automatically
nests as a child span with token usage and model metadata.

## Verify instrumentation worked

1. Confirm the SDK is importable and check the version:
   ```bash
   python -c "import pandaprobe; print(pandaprobe.__version__)"
   ```
2. Enable debug logging to see traces being sent (no key printed):
   ```bash
   PANDAPROBE_DEBUG=true python your_app.py
   ```
3. Run the app, then confirm the trace landed using the CLI (see `cli.md`):
   ```bash
   pandaprobe traces list --limit 5
   pandaprobe traces get <trace_id>     # inspect spans, token usage, status
   ```

If no trace appears, check `PANDAPROBE_API_KEY` / `PANDAPROBE_PROJECT_NAME` are set,
`PANDAPROBE_ENABLED` is not `false`, and the endpoint is reachable. See
https://docs.pandaprobe.com/tracing/configuration/troubleshooting.
