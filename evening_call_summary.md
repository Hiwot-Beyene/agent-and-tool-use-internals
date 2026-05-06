# Evening call summary — Day 2 (agent and tool-use internals)

Earlier on Slack we had already shared **gaps and sharpened questions**, then went **peer-by-peer** with **clarification prompts** so nobody had to guess architecture (sequential funnel vs unified tool menu, what `tool_calls` in JSONL actually mean, booking keyword gate vs function calling, etc.). That front-loaded ambiguity before the long research write.

During the day each of us **researched our partner’s gap** and circulated **draft explainers** (sources + concrete trace / API shapes); we read asynchronously where we could so the call could focus on **what still felt vague**.

On the evening call we walked those drafts live: peers asked for **tighter grounding in Week 10 `conversion-engine`**—e.g. naming **`StructuredLogger.emit`**, **honest post-hoc `tool_calls`**, and **scaffold vs provider `tool_calls`**—and for **actionable fields** (`decision_layer`, `available_tools`, `tool_schema_version`, block reasons) instead of generic “improve observability.” The writers revised toward **attribution-by-layer**, **safe logging** (no raw reasoning, minimal PII), and a **grounding commit** that points at real files rather than a hypothetical agent rewrite. By sign-off, the askers could say **gap closed** on **how to trace responsibility** as the stack moves toward **MCP / function calling**.
