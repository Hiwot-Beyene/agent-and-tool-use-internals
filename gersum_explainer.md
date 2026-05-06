# Explainer — How To Trace Responsibility in a Hybrid Agent Funnel

**Written by:** Gersum Asfaw
**For:** Hiwot Beyene  
**Topic:** Agent and tool-use internals — separating model, tool-schema, and scaffold responsibility  
**Date:** Day 2, Week 12

## The Question

Your Week 10 conversion engine already logs strong operational traces: stages, decisions, tool calls, latency, tokens, and cost. But the funnel itself is still mostly code-scheduled:

`enrich -> compose -> send -> reply -> book`

That means many "tool calls" in the trace are really summaries of orchestrated substeps, not evidence that a single model completion was shown several tools and chose one of them.

Your gap is whether the current traces let you correctly attribute behavior when the system drifts:

- what the model chose
- what the tool schema encouraged
- what the scaffold forced

You want to know how to design tool calls and traces so those sources of behavior stay separable as the system evolves toward real MCP or function-calling surfaces, while logging only safe, attributable fields and not hidden reasoning.

## Short Answer

The core design rule is simple:

**Log decisions at the same layer they are made.**

If the scaffold chose the next stage, log it as a scaffold decision. If the model emitted a tool call, log it as a model decision. If a tool description or schema constrained what the model could choose, log that schema as part of the decision context. Do not collapse those into one generic `tool_call` event.

That is the heart of your gap: not missing traces, but missing attribution boundaries.

## Why Current Pipeline Logs Are Not Enough

In a sequential code-scheduled funnel, the orchestrator usually decides what runs next. The model may still do useful work inside a stage, but it is not globally deciding among all possible actions at every turn.

So a trace like:

```json
{
  "stage": "reply",
  "tool_calls": ["compose_llm", "hubspot_update"]
}
```

is ambiguous. It does not tell you:

- whether the model selected `compose_llm` from a tool menu
- whether Python always ran `compose_llm` in that stage
- whether HubSpot was exposed as a callable tool to the model
- whether the model requested HubSpot or the scaffold ran it afterward

That is the attribution problem. The trace records execution, but not responsibility.

## The Three Responsibilities You Need To Separate

### 1. Model responsibility

This is behavior that came from the model's own output distribution:

- the model emitted a `tool_call`
- the model chose free text instead of a tool
- the model selected one tool over another after seeing both

Evidence for model responsibility must come from the model response itself.

### 2. Tool-schema responsibility

This is behavior shaped by how tools were described and constrained:

- vague tool names
- misleading descriptions
- hard-to-fill parameter schemas
- missing tools in the visible list

A model cannot choose a tool it was never shown, and it will choose badly if the schema communicates the wrong affordances.

### 3. Scaffold responsibility

This is behavior forced by orchestration logic outside the model:

- state-machine transitions
- precondition blocks
- fixed stage ordering
- retries and fallbacks

Scaffold responsibility is often the dominant source of behavior in early-stage agent systems. That is not bad. It is only a problem if the trace pretends deterministic scaffold choices were model intelligence.

## What Changes With Real Tool Calling

In today's sequential funnel, most decisions are made by code and only some are made by the model. In a real tool-calling surface, the model sees a menu of tools and may emit a structured call.

Without tool calling:

- the model is called inside a stage
- the scaffold determines which stage exists
- downstream systems often run after the model responds

With tool calling:

- the model receives serialized tool definitions in context
- the model can emit a structured tool call instead of plain text
- the client executes the requested tool and returns the result

At the token level, that means the model is no longer choosing only between language continuations. It is choosing between:

- ordinary assistant text
- one of several structured tool-call continuations

This richer action surface only helps if traces improve with it. Otherwise you gain more behavior but not more explainability.

## The Trace Design You Want

The safest design is to log one trace event per decision layer, not one blended event.

For each meaningful step, log four conceptual records:

### A. Decision context

- `trace_id`
- `turn_id`
- `stage`
- `available_tools`
- `tool_schema_version`
- `state_machine_state`
- `preconditions_satisfied`
- `model_name`

### B. Scaffold routing event

- `router_type: scaffold`
- `selected_stage`
- `reason_code`
- `blocked_tools`
- `fallback_path`

### C. Model action event

- `router_type: model`
- `assistant_mode: text | tool_call | no_tool`
- `selected_tool_name`
- `tool_arguments_valid`
- `response_format`
- `finish_reason`

You do not need raw hidden reasoning to know whether the model emitted a tool call.

### D. Tool execution event

- `executed_tool_name`
- `execution_status`
- `latency_ms`
- `input_schema_version`
- `result_summary`
- `error_class`

This separation makes attribution mechanical instead of interpretive.

## What Not To Log

Do **not** log:

- raw hidden reasoning
- unnecessary prospect PII
- full sensitive tool outputs
- private prompts when a derived summary is enough

Instead, log attributable summaries:

- hashed lead IDs instead of raw contact fields
- schema versions instead of full schemas every turn
- argument-shape summaries instead of sensitive values
- reason codes instead of hidden reasoning text

The principle is:

**log observable decisions, not private intermediate cognition**

That gives you high attribution with low leakage.

## A Concrete Failure Example

Suppose the system fails to offer booking after an engaged reply.

Bad trace:

```json
{
  "stage": "reply",
  "decision": "no booking",
  "tool_calls": []
}
```

This is nearly useless. It does not tell you who decided "no booking."

Better trace:

```json
{
  "trace_id": "t_482",
  "stage": "reply",
  "available_tools": ["draft_reply"],
  "blocked_tools": ["get_booking_link"],
  "block_reason": "state_machine_precondition_not_met",
  "assistant_mode": "text",
  "selected_tool_name": null
}
```

Now the cause is clear:

- booking was not missed by the model
- booking was blocked by scaffold preconditions
- prompt tuning would not fix this

That is the attribution power you are asking for.

## The Best Practical Design for Your Engine

The right next step is not "replace everything with MCP." It is:

1. keep the sequential funnel where it is operationally useful
2. mark scaffold decisions explicitly in traces
3. expose a small number of true tools where model choice is valuable
4. version tool descriptions and log the version shown to the model
5. log model-emitted action choice separately from tool execution

Without this separation, every failure looks like "bad agent." With it, the system becomes debuggable.

## What This Means for Your Week 10 Artifact

Your traces should distinguish at least:

- `decision_layer = scaffold | model | tool_runtime`
- `available_tools`
- `selected_tool`
- `tool_schema_version`
- `blocked_by_precondition`
- `fallback_triggered`

That is a better engineering surface than a single blended `tool_calls` field.

## Bottom Line

Your gap is real because your current traces show what executed but not always who was responsible for the decision. The right design is to separate what the scaffold offered, what the model emitted, and what the runtime executed. If those three are visible independently, attribution becomes mechanical instead of interpretive, and the system stays debuggable as it moves toward real MCP or function-calling surfaces.

## Sources

1. Yao, S. et al. "ReAct: Synergizing Reasoning and Acting in Language Models."  
   https://arxiv.org/abs/2210.03629  
   Load-bearing point: language-model systems become more interpretable and debuggable when action steps are externalized rather than treated as opaque internal behavior, which supports separating model-emitted actions from environment/tool execution.

2. Langfuse Data Model and Tracing Docs  
   https://langfuse.com/docs/observability/data-model/overview  
   Load-bearing point: traces, spans, events, and metadata should be structured so attribution remains legible across application layers.
