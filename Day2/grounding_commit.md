# Grounding Commit — Gashaw Bekele
## Week 12 | Agent & Tool-Use Internals

---

## Artifact Modified

**File:** `src/agents/query_agent.py`
**Project:** Week 3 — Document Intelligence Refinery
**Section:** Tool descriptions passed to the LLM in the query planning prompt

---

## What Changed

Rewrote all three tool descriptions in the query agent using the three-component pattern
(trigger + examples + anti-trigger) from Charlie's explainer.

**Before — `pageindex_navigate`:**
```
Navigate the PageIndex tree to find relevant sections.
Returns top-3 sections + their LDU IDs for focused retrieval.
```

**After — `pageindex_navigate`:**
```
Navigate the document's section tree to locate the right part of the document.
Use when the query asks about a named section, chapter, or topic area —
e.g., "What does the executive summary say about X?", "Find the methodology section."
NOT for numeric lookups, financial figures, or questions with exact values.
Prefer this over semantic_search when the question names a document section explicitly.
```

---

**Before — `semantic_search`:**
```
Vector similarity search over all ingested LDUs.
```

**After — `semantic_search`:**
```
Retrieve document chunks by meaning similarity to the query.
Use when the query is conceptual or narrative — e.g., "What are the main risks
mentioned?", "Summarize the approach to X."
NOT when the query contains specific numeric values, dates, or labeled financial
figures (e.g., "total revenue", "FY2024 profit"). Use structured_query for those.
Prefer this over structured_query when the answer is a paragraph, not a number.
```

---

**Before — `structured_query`:**
```
SQL queries over FactTable for numeric/financial data.
```

**After — `structured_query`:**
```
Query the structured fact table with SQL for precise numeric or financial answers.
Use when the query asks for a specific figure, label, or value —
e.g., "What is total comprehensive income for FY2024?", "Revenue in birr."
NOT for conceptual or narrative questions. Use semantic_search for those.
Prefer this over semantic_search whenever the query contains a financial label,
a year, or an explicit numeric request.
```

---

## Why This Change Improves the Artifact

Before this rewrite, `semantic_search` and `structured_query` had descriptions that
described the *mechanism* of each tool but gave the model no signal for *when* to prefer
one over the other. The model consistently routed numeric queries to `semantic_search`
because the description was broader and appeared first in the list.

After the rewrite, each description contains an explicit anti-trigger that names the
other tool as the alternative. The model now has a disambiguation signal at decision time,
not just a format signal. This directly addresses the routing failure observed in the
Week 3 pipeline for queries like "What is total comprehensive income for FY2024?" — which
previously went to `semantic_search` and returned a paragraph instead of a number.

---

## Verification

Added a 6-query smoke test covering the `semantic_search` / `structured_query` boundary:

| Query | Expected tool | Result after rewrite |
|---|---|---|
| "What is total comprehensive income FY2024?" | structured_query | ✓ |
| "What are the main risks in the report?" | semantic_search | ✓ |
| "Revenue in ETB for FY2024" | structured_query | ✓ |
| "Summarize the executive summary" | pageindex_navigate | ✓ |
| "Total assets as of March 2024" | structured_query | ✓ |
| "What does the report say about branch expansion?" | semantic_search | ✓ |

All 6 queries routed correctly after applying the three-component description pattern.
