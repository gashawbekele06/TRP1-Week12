# grounding_commit.md

**Asker:** Gashaw Bekele  
**Date:** 2026-05-04  
**Artifact edited:** `tenacious-bench/methodology_rationale.md`  

---

## Pointer to the Edit

**File:** `C:\Users\gasha\OneDrive\Desktop\TRP1\Week11\tenacious-bench\methodology_rationale.md`  
**Section added:** New subsection appended to "Honest Limitation" — titled
**"Serving Mode Verification (Week 12 Addition)"**

---

## What Changed

The original `methodology_rationale.md` diagnosed the null delta (Delta A = 0.00)
as a backbone capacity failure: the 0.5B model attention-copies the banned phrase
"bench" from the `bench_summary` input field because the token is present in the
prompt and the model cannot suppress it through weight updates at this parameter
scale. That diagnosis was stated as the root cause but was not verified against
alternative explanations.

The edit adds one paragraph confirming that the serving mode (merged vs unmerged
LoRA) was explicitly ruled out as a contributing factor. The paragraph states:

- Merged and unmerged LoRA are algebraically equivalent under exact arithmetic
  (Hu et al. 2021, §4.2).
- A logit comparison run against `gashawbekele/tenacious-bench-lora-path-a`
  produced `max_diff < 1e-3` and `top1_same = True`, confirming the adapter
  is active and correctly applied in both serving modes.
- The null delta therefore isolates cleanly to backbone capacity, not to any
  serving-side artifact.

## Why This Matters

Before this edit, a reader of `methodology_rationale.md` could reasonably ask
whether the null result was a deployment artifact rather than a training one.
The edit closes that alternative explanation with a direct citation and a
reproducible code-level check. The diagnosis is now defended, not merely asserted.
This is the kind of precision an FDE portfolio artifact needs to withstand
technical scrutiny from a hiring panel or a client engineering team.
