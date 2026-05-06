# Morning Call Summary
## Week 12 | Agent & Tool-Use Internals | Pair: Gashaw Bekele & Charlie Lijalem

*Written by: Gashaw Bekele | Confirmed by: Charlie Lijalem*

---

## What Was Ambiguous in the Original Drafts

**Gashaw's original draft question** asked broadly: "What is the mechanism by which an
LLM selects a tool — does it reason over descriptions, does the framework constrain the
output, or both?" Charlie interrogated this immediately: "You're asking three separate
questions. Is your real gap about the token-level mechanics, the schema serialization,
or the description-writing heuristics? Pick one — because a 600-word explainer cannot
close all three."

**Charlie's original draft question** asked about "the mathematical or architectural
mechanics of how GPT-4o-mini processes a document image." Gashaw interrogated: "Are you
asking about the ViT patch encoding, the OpenAI tiling algorithm, or how image and text
tokens merge in the transformer? These are three separate mechanisms — which one is
actually causing your pipeline to fail?"

---

## How Each Question Was Sharpened

**Gashaw's question** was narrowed from three sub-questions to one specific gap:
the routing failure in `query_agent.py` where numeric questions consistently go to
`semantic_search` instead of `structured_query`. The sharpened version names the
specific artifact, the specific observed failure, and asks what description properties
drive the dispatch decision — making it resolvable in one explainer.

**Charlie's question** was narrowed to the tiling → token → attention pipeline: what
does `detail: high` actually do to the image before it reaches the model, and how do
those image tokens combine with text tokens. The CLIP pre-training story and fine-tuning
questions were explicitly scoped out.

---

## Final Attestation

Both questions are unambiguous to the other partner. Each partner confirms the final
committed version above reflects the post-call sharpening, not the original draft.
