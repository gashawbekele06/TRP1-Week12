# Question — Gashaw Bekele
## Week 12 | Agent & Tool-Use Internals | Pair: Charlie Lijalem

---

## Final Committed Question

**"My Week 3 Document Intelligence Refinery exposes three tools to the LLM inside
`query_agent.py` — `pageindex_navigate`, `semantic_search`, and `structured_query`.
For numeric fact questions such as 'What is total comprehensive income for FY2024?',
the correct routing path is `structured_query` against the SQL fact table. But I
observed the model consistently selecting `semantic_search` instead, even when the
question contains explicit numeric and temporal signals. I cannot make a principled
fix because I do not understand the mechanism: when a model with multiple tools chooses
one, is it reasoning over the tool descriptions in natural language, is the framework
serializing tool schemas as structured JSON into the context window, or both — and
which specific properties of a tool description (the function name, the docstring,
the parameter names and types) most strongly determine whether the model picks the
right tool when two tools have semantically overlapping purposes?"**

---

## Connection to Shipped Artifact

This gap directly affects `query_agent.py` in my Week 3 Document Intelligence Refinery
(the query planning and tool-dispatch layer). The pipeline has a three-tool routing
decision at every query. Right now the tool descriptions were written by intuition.
Closing this gap would let me rewrite those descriptions with a principled understanding
of what signals the model is actually using to make the dispatch decision — and explain
that choice in the model card.

---

## Why This Generalizes

Any FDE building a multi-tool agent faces this decision: how to write tool descriptions
so the model routes reliably. The mechanism — how function schemas are serialized,
what the model emits at the token level, what makes a description load-bearing — is
the same across OpenAI, Anthropic, and open-source models. Closing this gap improves
every agent with overlapping tools, not only this pipeline.

---

## What a Satisfying Answer Looks Like

An explainer that tells me: (1) what the model actually sees when tool schemas are sent —
is it raw JSON, a formatted prose block, or something else; (2) what the model emits to
select a tool — a constrained token, a special delimiter, or a JSON completion; and
(3) one concrete, actionable rule for writing tool descriptions that reduces overlap
confusion — supported by a runnable example showing the difference between a description
that causes wrong routing and one that does not.
