1/6
My Week 10 **Conversion Engine** already logs **`stage`**, **`decision`**, **`tool_calls`**, and token/cost — so why is it still hard to say *why* something went wrong when **`enrich → compose → send → reply → book`** misbehaves?

I needed one clear story: **who** decided what — **model**, **tool schema**, or **scaffold**?

2/6
One flat “agent failed” label hides three different failure classes.

Today the funnel is mostly **code-scheduled**: each stage runs when **Python/API** invokes it. Many trace **`tool_calls`** lines are **summaries** (“we ran enrichment,” “we composed”) — not a single completion where the model ranked every funnel tool at once.

If you don’t separate layers, you **tune prompts** instead of fixing the right lever.

3/6
Three layers to keep apart:

- **Scaffold:** state machine, preconditions, what context each LLM step sees, any **override** or forced path.
- **Tool / prompt surface:** MCP-style **`tools[]`** defs, descriptions, JSON Schema, validation errors — what steers *routing* when function calling is real.
- **Model:** the stochastic policy **inside** whatever the scaffold allowed.

**Model “tool choice”** only means something literal when the provider returns **`tool_calls`** — not when you post-hoc label a step.

4/6
How to trace it (senior recipe):

- Log **`eligible_tools`** (or explicit “none — scaffold-only”) + **registry/version hash** per LLM turn.
- Log **parsed `tool_calls`** from the API when they exist; log **`scaffold_action`**: accepted, blocked, retried, replaced.
- Keep **HubSpot / side effects** as **executed** facts — you already need those rows.

Per turn: **what was allowed → what the model returned → what Python ran.** That’s attribution without guessing.

5/6
**Reasoning vs output:** I’m not asking to hoard chain-of-thought.

Debuggable traces need **decisions + structured outputs** (intents, args, validation). **Not** raw hidden reasoning or full prompts in prod logs — **hashes**, redaction, and policy-safe fields only.

Same idea as separating **prefill vs decode**: separate **attributable signal** from **noise / unsafe text**.

6/6
What I ship toward:

**Phase honest:** label scaffold-only steps so traces don’t *imply* model tool routing where there is none.

**Phase pilot:** one stage gets real **`get_*` / MCP-projected** tools + **tool result** messages; same trace schema.

**Portfolio:** conversion traces + bench exports that show **layer**, not vibes — so **`cost_pareto`-style** reliability work and **agent** work compound instead of blaming “the LLM.”
