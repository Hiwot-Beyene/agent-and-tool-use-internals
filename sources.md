## Canonical sources (Day 2 — agent & tool-use internals)

Aligned with the Week 12 topic bank **“Agent and tool-use internals”** (function-calling mechanics, MCP exposure, scaffold vs model, multi-turn planning). These are primary-style references for **tools in the API**, not second-hand summaries.

1. **OpenAI Platform — Function calling (tools in chat completions)**  
   https://platform.openai.com/docs/guides/function-calling  
   *Load-bearing for:* how **`tools` / `tool_choice`** surface in the request, and how **`tool_calls`** vs normal **`content`** show up in the assistant message (same pattern OpenRouter and other OpenAI-compatible stacks mirror).

2. **Model Context Protocol — Specification**  
   https://spec.modelcontextprotocol.io/specification/  
   *Load-bearing for:* how an MCP **host** advertises **tools** to a client/model, and why “MCP” is a **capability projection** problem—not magic the weights execute.

3. **Anthropic — Tool use (Messages API)**  
   https://docs.anthropic.com/en/docs/build-with-claude/tool-use  
   *Load-bearing for:* an alternate but equivalent mental model for **tool_use** blocks, **tool_result** messages, and multi-step loops—useful when comparing providers.

*(Challenge brief reminder: each day’s `sources.md` should reflect **≥2 canonical sources** plus the **tool/pattern** you actually used to ground the explainer—see the **Deliverables** / research-before-you-write section in the same Week 12 doc.)*

## Portfolio artifacts (my Day 2 question + Week 10 grounding)

Pointers from [`question_and_gap.md`](../question_and_gap.md) and the Conversion Engine layout where traces mix **scaffold-scheduled** stages with **LLM substeps**:

- [`conversion-engine/agent/agent/structured_logger.py`](../../../conversion-engine/agent/agent/structured_logger.py) — **`StructuredLogger.emit`**: `stage`, `decision`, `tool_calls`, `metadata` (append-only **`eval/agent_trace_log.jsonl`**).
- [`conversion-engine/agent/api/leads_router.py`](../../../conversion-engine/agent/api/leads_router.py) — enrich path: post-hoc **`tool_calls`** labels (e.g. `enrichment.crunchbase`, `compose.email_llm`) plus **`llm_usage`** in metadata.
- [`conversion-engine/agent/agent/orchestrator.py`](../../../conversion-engine/agent/agent/orchestrator.py) — **deterministic** lifecycle steps (`load → enrich → classify → decision matrix → CRM`); illustrates **scaffold vs model** separation.
- [`conversion-engine/eval/tau2/runner.py`](../../../conversion-engine/eval/tau2/runner.py) — sequential **`enrich → send → reply → book`** fixture driver (funnel shape for “multi-step” without a single joint tool menu).

## Tools and patterns (what I apply from the explainers)

- **Attribution trace shape** — per LLM turn log **`eligible_tools`** (or explicit scaffold-only), **`tool_registry_version` / prompt hash**, **parsed provider `tool_calls`** when present, **`scaffold_action`** (none / block / override / retry), then **executed side effects** — so post-hoc `tool_calls` rows are not mistaken for model-emitted routing (see peer explainers / `research_clarifications.md` in the parent folder).
- **MCP / function-calling pilot** — introduce real **`tools[]`** on one stage first; return **tool result** messages before final user-visible assistant text; keep **CRM writes deterministic post-LLM** where policy requires it (pattern discussed for booking / HubSpot flows).
- **Safe logging** — persist **decisions and structured outputs**, not raw chain-of-thought; redact PII; prefer **version hashes** over full prompt dumps in durable logs (Week 12 “observability” adjacent topic in the challenge doc).
