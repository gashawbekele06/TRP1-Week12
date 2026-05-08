# grounding_commit.md — Day 4

**Asker:** Gashaw Bekele
**Date:** 2026-05-08
**Artifact edited:** `tenacious-bench/improvement_report.md` and
`tenacious-bench/methodology_rationale.md`

---

## Pointer to the Edit

**File 1:** `improvement_report.md` — "What This Means for v0.2" section
**File 2:** `methodology_rationale.md` — "Evaluation Measurement Improvements" section

---

## What Changed and Why

**Before this edit**, the improvement report stated:

> *"Power analysis at Cohen's d ≈ 0.35 (estimated from the corrected n=3 delta)
> requires n ≈ 66 for 80% two-tailed power. At n=50, power ≈ 70%."*

That used an informal d estimate without specifying d_z (the paired variant)
or computing std(Δ) from the actual per-task differences. It also cited n≈66
without the paired-design correction.

**After this edit**, both files now state:

The correct effect size for the ablation is **Cohen's d_z = 0.304**, computed
from the paired differences (Lakens 2013):

```
deltas  = [+0.2028, +0.2028, −0.1958]
d_z     = mean(deltas) / std(deltas) = 0.0699 / 0.2301 = 0.304
```

Required n for 80% two-tailed power: **~85 tasks** (not 66, not 410).

| n | Power |
|---|---|
| 3 | 10.4% |
| 17 | 27.8% |
| 50 | 57.9% |
| 85 | 80.1% |
| 100 | 85.4% |

The previous estimate used Cohen's h, which applies to raw pass-rate
proportions (binomial variance p(1−p)/n). Rubric scores are weighted sums
of binary checks — Poisson-binomial mixtures — for which the arcsine
variance stabilisation in Cohen's h is not valid. Using h at δ=0.070
near p̄≈0.57 inflates the required n from 85 to ~410.

**Why this matters for the artifact:**
The improvement_report.md is the document that tells a future reader how
many held-out tasks to collect for a conclusive ablation. Citing the wrong
formula could cause either of two errors:

1. Stop at 85 thinking it is underpowered (h says 85/410 = 21% power) when
   it is actually at 80% power (d_z says 85 = target).
2. Run 410 tasks when 85 would have been enough — wasting ~325 inference calls.

The edit makes the target concrete and defensible: **expand held_out to 85
tasks from train_filtered/ to reach 80% power at the observed effect size.**
