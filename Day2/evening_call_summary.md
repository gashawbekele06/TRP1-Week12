# Evening Call Summary
## Week 12 | Agent & Tool-Use Internals | Pair: Gashaw Bekele & Charlie Lijalem

*Written by: Gashaw Bekele | Confirmed by: Charlie Lijalem*

---

## Feedback Gashaw Gave Charlie on the VLM Explainer

**What landed well:**
- The tiling algorithm with the Python token calculator was exactly the right level of
  concreteness — Gashaw could run it immediately and verify the numbers against the
  described pipeline behavior.
- The two-pass strategy implication was the highest-signal adjacent concept; it directly
  changes how the budget cap logic should be written.
- The "scoped out" section was appreciated — knowing CLIP pre-training was excluded helped
  Gashaw trust that the in-scope content was complete, not selective.

**What did not land initially:**
- The "unified token sequence" claim in Mechanism 1 felt asserted rather than shown.
  Gashaw asked: "How do I know image patches and text tokens actually share the same
  attention matrix? Can you show a concrete shape or diagram?" Charlie revised to add
  the concatenation step explicitly and cite the GPT-4V System Card directly.

**What Charlie revised:**
- Added the explicit concatenation step (Step 4 in Mechanism 1) with the language
  "vectors in the same embedding space — processed by one transformer with one
  attention mechanism."
- Clarified that the 85-token base overhead is a low-resolution thumbnail included at
  all detail levels, not only high.

---

## Feedback Charlie Gave Gashaw on the Tool Selection Explainer

**What landed well:**
- The two-layer distinction (Layer 1 = which tool, Layer 2 = valid format) resolved
  Gashaw's core confusion immediately. Gashaw had assumed the framework was doing the
  routing — understanding that Layer 2 is orthogonal to selection was the central unlock.
- The three-component description pattern (trigger + examples + anti-trigger) was
  immediately actionable. Gashaw rewrote the tool descriptions during the call.
- The quantitative claim from ToolScope (8.8–38.6% degradation) gave Gashaw a way to
  defend the rewrite as a correctness fix, not a stylistic preference.

**What did not land initially:**
- Gashaw asked: "You say anti-triggers are more powerful than triggers — but what is the
  mechanism? Is this an attention thing or a training data thing?" Charlie added the
  clarification that anti-trigger suppression is more reliable because the model was
  fine-tuned on examples where explicit negative examples produced correct avoidance,
  whereas trigger phrases rely on semantic similarity which can be weak with overlapping
  descriptions.

**What Gashaw revised after signoff:**
- Applied the three-component pattern to all three tool descriptions in `query_agent.py`.
- Added a 6-query smoke test covering the overlapping `semantic_search` /
  `structured_query` boundary.

---

## Signoff Status

- **Charlie's signoff on Gashaw's VLM explainer:** Gap closed.
- **Gashaw's signoff on Charlie's tool selection explainer:** Gap closed.
