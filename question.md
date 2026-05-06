# Week 12 Day 2 - Knowledge Gap (Agent and Tool-Use Internals)

**Name:** Hiwot Beyene  
**Submitting to Explainers:** Meserete Bolled, Gersum Asfaw  
**Week 10 implementation context:** `conversion-engine`

## Gap I identified in my Week 10 implementation

My Week 10 Conversion Engine has strong operational traces (`stage`, `decision`, `tool_calls`, token/cost metadata), but today the funnel is mostly **sequential and code-scheduled** (`enrich → compose → send → reply → book`): each stage runs when the API/orchestrator invokes it, and trace `tool_calls` are often **summaries of substeps** (enrichment providers, compose LLM), not a single model completion choosing among all funnel tools at once.

Even so, I **cannot** yet attribute failures cleanly when behavior drifts across turns:

- for **LLM-driven substeps** (e.g. compose, reply intent), what the model actually decided vs what the **scaffold** forced,
- how **MCP-style tool descriptions** would steer routing if I expose more actions as real tools,
- and what to **log** for debugging vs **withhold** (hidden reasoning, PII).

So my core gap is: **I cannot separate model vs tool-schema vs scaffold responsibility in traces** in a way that stays correct as I move from today’s pipeline logs toward **real MCP / function-calling** surfaces.

## Final research question (precise and concise)

For my Conversion Engine funnel `enrich → compose → send → reply → book` (code-scheduled stages today; more MCP-exposed tools over time), how should I **design and trace** tool calls so I can separate **(1)** model-emitted tool choice where it exists, **(2)** routing effects of **tool descriptions/schemas**, and **(3)** **scaffold** (state machine, preconditions)—while logging only **safe, attributable** fields and not raw hidden reasoning?

## Why this gap matters for my implementation

If I close this gap, I can redesign my tool schemas and orchestration to improve both reliability and debuggability of conversion decisions, rather than only tuning prompts and hoping pass rates move.
