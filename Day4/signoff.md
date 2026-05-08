# signoff.md — Day 4

**Asker:** Gashaw Bekele
**Explainer:** Eyobed Feleke
**Date:** 2026-05-08

---

## Gap Closure Judgment

**Status: CLOSED**

---

## What I Understand Now That I Did Not Before

My question was: should I use Cohen's h or Cohen's d to compute the required
sample size for my ablation, and why do the two formulas give such different
answers at the same observed delta?

Before Eyobed's explainer I was treating the two formulas as interchangeable
alternatives for a [0,1]-bounded score. I knew Nebiyou had used Cohen's h for
his 12%→26% pass-rate comparison, and I assumed the same formula applied to
my weighted rubric scores. The ~5× difference in required n (410 vs 85) seemed
like a rounding or implementation issue, not a fundamental statistical mistake.

After the explainer I can now state the following with confidence:

**The two formulas are not interchangeable — they apply to different data-
generating processes.**

Cohen's h is valid only when the estimator is p̂ = successes/n from a
single binomial. The arcsine transform φ = 2·arcsin(√p) is needed because
binomial variance p(1−p)/n varies with p — the transform stabilises it so
that sample-size formulas remain valid across different p values. This is the
entire purpose of Cohen's h, and it only works when the variance structure
really is p(1−p)/n.

My rubric score is a weighted sum of four heterogeneous Bernoulli(pᵢ) checks.
Its variance is Σ(wᵢ/W)²·pᵢ(1−pᵢ) — a Poisson-binomial mixture. The
arcsine transform is not calibrated for this variance structure. Applying
Cohen's h misidentifies the variance, compresses the apparent effect near
p̄ ≈ 0.57, and inflates the required n by ~5×. This is not a small correction —
it is the difference between stopping at 85 tasks (achievable) and needing
410 (expensive and probably never collected).

**The correct formula is Cohen's d_z for paired designs** (Lakens 2013):

```
d_z = mean(Δ) / std(Δ)
    = 0.0699 / 0.2301
    = 0.304
```

This treats paired task differences as the unit of analysis — which is
correct because my ablation scores the same tasks under both trained and
baseline conditions. At d_z = 0.304, 80% power requires ~85 tasks, and the
power at my current n=17 is only 27.8%.

**The decision rule I now use:**
- Metric = count_of_passing_tasks / n → Cohen's h
- Metric = aggregate score, weighted sum, or any per-task score → Cohen's d_z

This distinction directly changes my v0.2 evaluation plan. I now need 85
held-out tasks, not 410. Expanding from 17 to 85 means moving ~68 more
tasks from train_filtered/ — a concrete, achievable target.
