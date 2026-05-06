# morning_call_summary.md

**Written by:** Meseret  
**Confirmed by:** Gashaw Bekele  
**Call duration:** ~25 minutes  
**Date:** 2026-05-04  

---

## What Was Ambiguous in the Original Drafts

**Gashaw's original draft question** was: *"Why did my LoRA adapter produce a null delta
(Delta A = 0.00) when training loss dropped from 3.08 to 0.42? Could the serving mode
(merged vs unmerged) explain it?"*

Meseret's interrogation surfaced two problems with this draft:

1. **The question mixed a diagnosed cause with an unresolved hypothesis.**
   Gashaw's own `methodology_rationale.md` already documents the root cause —
   the 0.5B backbone attention-copies banned phrases from the input context
   regardless of adapter weights. Meseret pushed back: *"If you already know why
   the delta is zero, what is the actual gap? Are you asking about the mechanism
   you don't understand, or about a cause you haven't ruled out?"*

2. **"Could the serving mode explain it?" is a yes/no question, not a gap.**
   Meseret asked: *"What would you need to understand about merged vs unmerged
   serving to be able to answer that yourself? Name the mechanism you're missing."*
   This forced Gashaw to separate the diagnosed root cause from the second
   diagnostic layer — whether serving mode adds any additional divergence — and
   to name the forward-pass arithmetic as the specific thing he could not explain.

## How the Question Was Sharpened

The final question now explicitly accepts the backbone capacity diagnosis as
settled and frames the serving-mode question as a second, independent diagnostic
layer. The gap is precisely named: *are merged and unmerged LoRA weights
mathematically guaranteed to produce identical logits, and under what conditions
does that guarantee break?* This is answerable by mechanism, not by a yes/no,
and it connects directly to every FDE adapter deployment decision — not only
this experiment.

Meseret attests the final committed question is unambiguous and resolvable in
a 600–1,000 word explainer.
