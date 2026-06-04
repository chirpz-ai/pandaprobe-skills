---
name: pandaprobe-instrumentation
description: Add PandaProbe tracing to an AI agent or LLM application. Use when instrumenting an agent framework (LangGraph, LangChain, OpenAI Agents, CrewAI, Google ADK, Claude Agent SDK, DeepAgents), wrapping an LLM provider client, adding manual trace/span instrumentation, or verifying that traces are captured.
---

# PandaProbe SDK Instrumentation

Add PandaProbe tracing to a developer's AI agent or LLM application. PandaProbe captures
execution as **traces** (one logical run) composed of **spans** (steps within a run).

Instrumentation has three layers, in priority order:

1. **Agent framework integrations** — the primary path. If the app runs on a supported
   framework, this gives end-to-end traces (LLM + tools + sub-agents) with almost no code.
2. **LLM provider wrappers** — when there's no agent framework, or you only need LLM-call
   visibility.
3. **Manual instrumentation** — decorators/context managers for custom runtimes or extra
   spans around your own logic.

Prefer the highest layer that fits; they compose (a wrapper call inside an integration or
`@pandaprobe.trace` nests automatically as a child span).

## Fetch current docs first

**The SDK evolves — never instrument from memory.** Before writing code, fetch the latest
docs and adopt the latest SDK version. Treat every snippet below as a starting point and
**verify it against the current docs** — especially exact import paths and class names,
which are framework-specific.

- Index (discover all pages): https://docs.pandaprobe.com/llms.txt
- Overview & concepts: https://docs.pandaprobe.com/tracing/overview · https://docs.pandaprobe.com/tracing/concepts
- Integrations (agent frameworks): https://docs.pandaprobe.com/tracing/integrations/overview
- Wrappers (LLM providers): https://docs.pandaprobe.com/tracing/wrappers/overview
- Manual decorators / context managers: https://docs.pandaprobe.com/tracing/manual/decorators
- Env vars: https://docs.pandaprobe.com/tracing/configuration/environment-variables

Append `.md` to any docs URL for clean markdown (e.g. `…/tracing/integrations/langgraph.md`).
**Always open the specific framework's integration page before wiring it up.**

## Install

```bash
pip install pandaprobe        # Python 3.10+ (or: uv add pandaprobe)
```

Install only the extras for the providers/frameworks in use (combine with commas, e.g.
`pip install "pandaprobe[langgraph,openai]"`):

| Extra                          | Covers                                    |
| ------------------------------ | ----------------------------------------- |
| `pandaprobe[langgraph]`        | LangGraph integration                     |
| `pandaprobe[langchain]`        | LangChain integration                     |
| `pandaprobe[deepagents]`       | DeepAgents integration                    |
| `pandaprobe[openai-agents]`    | OpenAI Agents SDK integration             |
| `pandaprobe[crewai]`           | CrewAI integration                        |
| `pandaprobe[google-adk]`       | Google ADK integration                    |
| `pandaprobe[claude-agent-sdk]` | Claude Agent SDK integration              |
| `pandaprobe[openai]`           | OpenAI wrapper                            |
| `pandaprobe[anthropic]`        | Anthropic wrapper                        |
| `pandaprobe[gemini]`           | Google Gemini wrapper                    |
| `pandaprobe[mistral]`          | Mistral wrapper                          |
| `pandaprobe[bedrock]`          | AWS Bedrock wrapper (beta)               |

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

The SDK auto-initializes from the environment. You can also configure programmatically via
`pandaprobe.init()` — see https://docs.pandaprobe.com/tracing/configuration/project-configuration.
If keys are missing, ask the user to set them in their shell or `.env`; don't ask them to
paste a key into chat.

## Layer 1 — Agent framework integrations (recommended)

Hook into a framework to automatically trace the full agent lifecycle — LLM calls, tool
invocations, sub-agent handoffs, guardrails — as a properly nested span tree. You don't
create traces or spans yourself.

All integrations share the same constructor params: `session_id`, `user_id`, `tags`,
`metadata` (all optional). They differ only in **how you wire them in**, via one of two
patterns:

- **Callback handler** (LangChain-family: LangGraph, LangChain, DeepAgents): construct a
  handler and pass it per invocation via `config={"callbacks": [handler]}`.
- **Adapter `.instrument()`** (OpenAI Agents, CrewAI, Google ADK, Claude Agent SDK):
  construct an adapter and call `adapter.instrument()` **once at startup**, before creating
  agents/runners — then use the framework normally.

| Framework        | Import (module · class)                                      | Wiring                |
| ---------------- | ------------------------------------------------------------ | --------------------- |
| LangGraph        | `pandaprobe.integrations.langgraph` · `LangGraphCallbackHandler`     | callback handler |
| LangChain        | `pandaprobe.integrations.langchain` · `LangChainCallbackHandler`     | callback handler |
| DeepAgents       | `pandaprobe.integrations.deepagents` · `DeepAgentsCallbackHandler`   | callback handler |
| OpenAI Agents    | `pandaprobe.integrations.openai_agents` · `OpenAIAgentsAdapter`      | `.instrument()`  |
| CrewAI           | `pandaprobe.integrations.crewai` · `CrewAIAdapter`                   | `.instrument()`  |
| Google ADK       | `pandaprobe.integrations.google_adk` · `GoogleADKAdapter`            | `.instrument()`  |
| Claude Agent SDK | `pandaprobe.integrations.claude_agent_sdk` · `ClaudeAgentSDKAdapter` | `.instrument()`  |

**Callback-handler pattern (LangGraph):**

```python
# verify against current docs
from pandaprobe.integrations.langgraph import LangGraphCallbackHandler

handler = LangGraphCallbackHandler(session_id="conversation-123", user_id="user-abc")
result = graph.invoke(
    {"messages": [{"role": "user", "content": "Hello!"}]},
    config={"callbacks": [handler]},   # pass on every invocation
)
```

**Adapter pattern (OpenAI Agents — same shape for CrewAI, Google ADK, Claude Agent SDK):**

```python
# verify against current docs
from pandaprobe.integrations.openai_agents import OpenAIAgentsAdapter

OpenAIAgentsAdapter(session_id="conversation-123", user_id="user-abc").instrument()
# ...then build and run agents as usual; runs are traced automatically.
```

Each framework's page documents exactly what gets traced and any framework-specific
caveats — **read it before wiring**: https://docs.pandaprobe.com/tracing/integrations/overview

## Layer 2 — LLM provider wrappers

When there's no agent framework, wrap the provider client once. The wrapper returns the
same client type, so existing call sites are unchanged, and every call is traced (input,
output, model, token usage, params, TTFT for streaming).

```python
# verify against current docs
from pandaprobe.wrappers import wrap_openai
from openai import OpenAI

client = wrap_openai(OpenAI())   # use exactly as before — all calls now traced
```

Available wrappers: `wrap_openai`, `wrap_anthropic`, `wrap_gemini`, `wrap_mistral`,
`wrap_bedrock` (beta). Details: https://docs.pandaprobe.com/tracing/wrappers/overview

## Layer 3 — Manual instrumentation

For custom runtimes or to add spans around your own logic. Use `@pandaprobe.trace` on a
top-level entry point and `@pandaprobe.span` on inner steps; both auto-detect sync/async
and capture inputs/outputs. `pandaprobe.start_trace()` / `t.span()` context managers cover
cases decorators don't fit.

```python
# verify against current docs
import pandaprobe

@pandaprobe.trace(name="support-agent")               # entry point -> a trace
def handle_request(query: str) -> str:
    docs = retrieve_docs(query)
    return generate_answer(query, docs)

@pandaprobe.span(name="retrieve", kind="RETRIEVER")   # inner step -> a span
def retrieve_docs(query: str) -> list[str]:
    ...
```

`@pandaprobe.span` requires an active trace context. Span kinds: `LLM`, `TOOL`, `AGENT`,
`CHAIN`, `RETRIEVER`, `EMBEDDING`, `OTHER`. Details:
https://docs.pandaprobe.com/tracing/manual/decorators

## Verify instrumentation worked

1. Confirm the SDK imports and check the version:
   ```bash
   python -c "import pandaprobe; print(pandaprobe.__version__)"
   ```
2. Run the app with debug logging to see traces being sent (key stays masked):
   ```bash
   PANDAPROBE_DEBUG=true python your_app.py
   ```
3. Confirm the trace landed using the CLI (see [cli.md](cli.md)):
   ```bash
   pandaprobe traces list --limit 5
   pandaprobe traces get <trace_id>     # inspect spans, token usage, status
   ```

If no trace appears, check `PANDAPROBE_API_KEY` / `PANDAPROBE_PROJECT_NAME` are set,
`PANDAPROBE_ENABLED` is not `false`, and the endpoint is reachable. See
https://docs.pandaprobe.com/tracing/configuration/troubleshooting.
