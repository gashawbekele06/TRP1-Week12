# signoff.md — Nebiyou's gap closure judgment

**Asker:** Nebiyou Abebe
**Explainer:** Gashaw Bekele
**Date:** 2026-05-07

---

## Gap Closure Judgment

**Status: CLOSED**

---

## What Nebiyou Confirmed (verbatim)

> "Sample size was the problem — 100 tasks resolved it and the result is now
> significant. Training worked. The remaining gap is a capacity ceiling on 0.6B,
> not a training failure."

---

## What This Means

Nebiyou followed the decision rule from Gashaw's explainer exactly:

| Step | Predicted outcome | Actual outcome |
|---|---|---|
| Run 100 examples | p < 0.05 if sample-size limited | ✅ p < 0.05 confirmed |
| Interpret Δ at n=100 | Lift is real if Δ holds | ✅ +14 pp held — training worked |
| Diagnose residual gap | If lift holds, investigate capacity ceiling | ✅ 0.6B capacity identified |

The explainer's central claim — *"not significant ≠ no effect; expand to n=100
before blaming the training setup"* — was confirmed by the actual experiment.

The remaining gap Nebiyou identified (0.6B capacity ceiling) is a separate,
correctly scoped finding: the adapter works, but the backbone limits how far
the improvement can go. This is now a training-side question for a future day,
not a statistical artifact.

---

## What Nebiyou Understands Now That He Did Not Before

1. A p-value above 0.05 at n=50 with 73% power (two-tailed) is not evidence
   of no effect — it is a measurement with a 27% chance of missing a real lift.
2. The minimum diagnostic for any LLM pass-rate experiment is n=100 before
   pursuing training-side explanations.
3. Cohen's h is the correct effect-size measure for proportion comparisons,
   and a 14 pp lift (12% → 26%) corresponds to h=0.36 — medium effect, detectable
   at n=48 one-tailed or n=60 two-tailed.
4. "Training worked" and "result not significant" can both be true simultaneously
   when the eval set is undersized.
