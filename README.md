# Knowledge Gap Formulation (Week 12 — Day 2)

This **`submission/`** folder bundles one full Day 2 exercise for **Hiwot Beyene** ([@hiwot-beyene](https://huggingface.co/hiwot-beyene)): **Agent and tool-use internals** — from a sharpened question on the **Conversion Engine**, to peer explainers, packaged comms (thread, summaries, signoff, sources), and a **grounding commit** tied to real trace code.

## What this is for

- Document how the Day 2 question was negotiated and answered (**scaffold vs model vs tool schema**, **MCP / function calling**, **safe traces**).
- Ship public-style artifacts (`thread.md`) and cohort deliverables (`signoff.md`, `grounding_commit.md`, `sources.md`).
- Record **morning / evening** call summaries and the **bidirectional** explainer flow (what i received vs what i wrote for peers).

## Lait (this directory)

- `README.md`  
  Short guide to the submission bundle and file purposes.

- `question.md`  
  My Day 2 research question and gap (**`conversion-engine`** funnel, trace attribution, MCP-ready design).

- `thread.md`  
  Six-post thread distilling the answer to r question (layers, logging recipe, pilot MCP, portfolio).

- `sources.md`  
  Canonical links (OpenAI function calling, MCP spec, Anthropic tool use), Week 12 challenge pointer, **`conversion-engine`** artifact paths, and patterns i will run.

- `morning_call_summary.md`  
  How the question was sharpened after the morning call (topics → Week 10 grounding).

- `evening_call_summary.md`  
  Slack + clarification flow, draft explainers, evening call feedback and revisions.

- `signoff.md`  
  ir gap-closure judgment after internalizing layered attribution.

- `grounding_commit.md`  
  Planned portfolio edit: **`StructuredLogger.emit`**, **`leads_router` / orchestrator`** call sites, attribution metadata on **`eval/agent_trace_log.jsonl`**.

- `explainer.md`  
  Long-form explainer **i wrote for Gersum**, in the cohort “Asker’s Question / My Answer” shape (tool attribution, traces, MCP).

- `gersum_explainer.md`  
  Long-form explainer **Gersum wrote for Me** (trace design: decision context, scaffold vs model events, what not to log).

## Sister copy (parent folder)

Authoritative working drafts and peer clarification docs may also live in [`../`](../) (e.g. [`../question.md`](../question.md)); **`submission/`** is what i hand in as the **Day 2 package**.

## My External links

- **Tenacious-Bench (Week 11):** [github.com/Hiwot-Beyene/tenacious-bench](https://github.com/Hiwot-Beyene/tenacious-bench)  
- **Conversion Engine (Week 10):** implementation in this workspace under [`conversion-engine/`]((https://github.com/Hiwot-Beyene/conversion-engine) — mirror or submodule on GitHub as i publish.  
- **Writing:** [hiwotbeyene.substack.com](https://hiwotbeyene.substack.com/)  
- **Hugging Face:** [huggingface.co/hiwot-beyene](https://huggingface.co/hiwot-beyene)  
