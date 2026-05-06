# Question — Gashaw Bekele
## Week 12 | Agent & Tool-Use Internals | Pair: Charlie Lijalem

---

## Final Committed Question

**"When an LLM agent has multiple tools available, what is the mechanism by which it
selects one — does the model reason over tool descriptions, does the framework constrain
the output format, or both? Specifically, when two tools serve overlapping purposes
(like a semantic search tool and a structured query tool), what determines which one
the model picks, and what makes a tool description reliably steer that choice?"**

---

## Connection to Shipped Artifact

This gap directly affects the query agent in my Week 3 Document Intelligence Refinery
(`query_agent.py`), which exposes three tools to the LLM: `pageindex_navigate`,
`semantic_search`, and `structured_query`. For numeric fact questions such as
"What is total comprehensive income for FY2024?", the correct path is `structured_query`
against the SQL fact table — but the model consistently selected `semantic_search`
instead. I wrote those tool descriptions by intuition and cannot defend that choice
or diagnose the routing failure without understanding the selection mechanism.

Closing this gap would let me rewrite the tool descriptions with a principled
understanding of what signals the model actually uses at decision time, and explain
that choice in my pipeline's model card.

---

## Why This Generalizes

Any FDE building a multi-tool agent faces this exact decision: how to write tool
descriptions so the model routes reliably when tools overlap. The mechanism — how
schemas are serialized into context, what the model emits to select a tool, what
makes a description load-bearing vs. decorative — is the same across OpenAI,
Anthropic, and open-source models. Closing this gap improves every agent with
overlapping tools, not only this pipeline.

---

## What a Satisfying Answer Looks Like

An explainer that answers: (1) whether the model or the framework is responsible for
the selection decision; (2) what specific properties of a tool description most strongly
steer the choice when two tools overlap; and (3) one actionable rule — supported by a
concrete example — for writing descriptions that eliminate routing confusion.
