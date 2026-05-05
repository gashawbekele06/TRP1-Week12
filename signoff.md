# signoff.md

**Asker:** Gashaw Bekele  
**Explainer:** Meseret  
**Date:** 2026-05-04  

---

## Gap Closure Judgment

**Status: CLOSED**

---

## What I Understand Now That I Did Not Before

Before Meseret's explainer I could not state whether merged and unmerged LoRA
serving produced the same output distribution or not. I had a vague assumption
that they were equivalent but I had no mechanism to defend that assumption, and
I could not have told you under what conditions it breaks.

After the explainer I can now state the following with confidence:

Merged and unmerged LoRA are algebraically identical — `W'·x = W·x + (α/r)·B·A·x`
by the distributive law, as Hu et al. (2021) §4.2 establishes. In fp16/bf16 the
numerical gap is below 1e-3 in logit space and does not change the top-1 token
under standard conditions. The three conditions that break equivalence are: (1) a
scaling-factor bug where `α/r` is applied inconsistently across the two code paths,
(2) floating-point accumulation differences that are real but sub-threshold for
token selection, and (3) dropout left active in unmerged mode because `model.eval()`
was not called — the only condition that produces a materially wrong output, and
the most likely silent failure in deployment.

Running the logit comparison from Section 3 of the explainer against my published
adapter (`gashawbekele/tenacious-bench-lora-path-a`) confirmed: `max_diff < 1e-3`
and `top1_same = True`. Serving mode is ruled out as a contributor to the null
delta (Delta A = 0.00). The backbone capacity diagnosis in `methodology_rationale.md`
stands as the sole documented root cause.

I can now defend the serving decision in any FDE deployment and I know exactly
what to check first if a deployed adapter appears to have no effect.
