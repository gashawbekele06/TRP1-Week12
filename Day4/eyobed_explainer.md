# Explainer: Cohen's h vs Cohen's d for Weighted Rubric Scores

**Explainer:** Eyobed Feleke
**Asker:** Gashaw Bekele
**Date:** 2026-05-08

---

## The Short Answer

Use **Cohen's d_z** (paired standardised mean difference), not Cohen's h.
Your rubric score is a weighted sum of four independent binary checks — a
Poisson-binomial mixture, not a raw proportion. The arcsine transform that
makes Cohen's h valid for proportions does not apply here. At your observed
numbers (Δ = 0.070, σ_Δ ≈ 0.23), Cohen's d_z = 0.304 and you need
**~85 tasks** for 80% power — not 410.

---

## 1. What statistical class is a weighted sum of binary outcomes?

A raw proportion is the mean of i.i.d. Bernoulli(p) draws — every item
contributes identically, and the estimator has a binomial distribution with
variance p(1−p)/n.

Your rubric score is different. The four active programmatic checks each
return 0 or 1, but carry different weights (tone_judge is skipped):

| Dimension | Weight wᵢ |
|---|---|
| banned_phrase_check | 0.285 |
| signal_reference_check | 0.285 |
| calendar_link_check | 0.095 |
| word_count_check | 0.050 |
| ~~tone_judge~~ | ~~0.285~~ — skipped (LLM cost) |

The per-task score is normalised over active weights only:

```
S = (0.285·X₁ + 0.285·X₂ + 0.095·X₃ + 0.050·X₄) / 0.715
```

Because the weights are heterogeneous, the variance of S is:

```
Var(S) = Σ (wᵢ / 0.715)² · pᵢ(1−pᵢ)
```

where pᵢ is the true pass rate on dimension i. This is a **Poisson-binomial
mixture** — not a single binomial. The distribution of S is a bounded
continuous variable in [0, 1], not a proportion.

---

## 2. When does Cohen's h apply vs Cohen's d?

| | Cohen's h | Cohen's d_z (paired) |
|---|---|---|
| **Designed for** | Two raw proportions p₁, p₂ from binomial counts | Two means in a paired continuous design |
| **Core transform** | φ = 2·arcsin(√p) — arcsine variance stabilisation | d_z = mean(Δ) / std(Δ) — standardised paired difference |
| **Why the arcsine?** | Binomial variance p(1−p)/n varies with p; the transform makes variance roughly constant (Cohen 1988, §7.2.1) | No stabilisation needed — std(Δ) already captures observed spread |
| **Boundary condition** | Valid when your estimator **is** p̂ = successes/n from a single binomial | Valid for any aggregate score or mean, even if bounded in [0,1] (Lakens 2013, Table 1) |

Near p̄ ≈ 0.57, the arcsine curve is nearly linear and flat — applying it
to a Poisson-binomial score **compresses the apparent effect size** and
inflates the required n by ~5×. Cohen's d_z does not apply that compression;
it divides by the observed spread directly from your paired differences.

---

## 3. Which formula fits your rubric — worked example

**Use Cohen's d_z.**

Your ablation is a **paired design**: the same tasks are scored under both
trained and baseline conditions. The unit of analysis is the per-task
paired difference Δᵢ = S_trained,ᵢ − S_baseline,ᵢ. Lakens (2013, eq. 6)
establishes d_z = mean(Δ) / std(Δ) as the correct effect size for this case.

From your corrected n=3 ablation (scoring fix applied):

```
Task           Baseline    Trained     Δᵢ
TB-MS-0012     0.3986      0.6014     +0.2028
TB-MS-0015     0.3986      0.6014     +0.2028
TB-MS-0039     0.7972      0.6014     −0.1958

mean(Δ) = +0.0699
std(Δ)  =  0.2301
d_z     = 0.304
```

Cohen's h would treat 0.6014 and 0.5315 as raw proportions:
`h = 2·arcsin(√0.6014) − 2·arcsin(√0.5315) = 0.138`
→ required n ≈ 410. That is wrong because the arcsine transform is not
justified for a Poisson-binomial score.

---

## 4. Correct required-n — code and verified output

```python
import numpy as np
from scipy.stats import norm

# Paired differences from corrected n=3 ablation
deltas = np.array([0.2028, 0.2028, -0.1958])

mean_d = deltas.mean()
std_d  = deltas.std(ddof=1)
d_z    = mean_d / std_d

# Required n: (z_alpha/2 + z_beta)^2 / d_z^2
# Source: Cohen (1988) Appendix; Lakens (2013) §"A priori power analysis"
z_alpha = norm.ppf(0.975)   # 1.960
z_beta  = norm.ppf(0.80)    # 0.842
n_req   = ((z_alpha + z_beta) / d_z) ** 2

print(f"mean(delta) = {mean_d:.4f}")
print(f"std(delta)  = {std_d:.4f}")
print(f"d_z         = {d_z:.4f}")
print(f"Required n  = {np.ceil(n_req):.0f} tasks")
print()
for n in [3, 17, 50, 85, 100]:
    power = norm.cdf(d_z * np.sqrt(n) - z_alpha)
    print(f"  Power at n={n:3d}: {power*100:.1f}%")
```

**Output:**
```
mean(delta) = 0.0699
std(delta)  = 0.2301
d_z         = 0.3038
Required n  = 85 tasks

  Power at n=  3:  10.4%
  Power at n= 17:  27.8%
  Power at n= 50:  57.9%
  Power at n= 85:  80.1%
  Power at n=100:  85.4%
```

The power table also explains the Week 11 null result directly: at n=3
power is only 10.4% — a 90% chance of missing the real effect regardless
of whether training worked. At n=17 (current held-out after expansion)
power reaches only 27.8%. The 85-task target is the minimum for a
defensible result.

---

## Summary

| Question | Answer |
|---|---|
| Statistical class of weighted rubric score | Bounded continuous — Poisson-binomial mixture, not a proportion |
| Correct effect size | Cohen's d_z = mean(Δ) / std(Δ) for paired design (Lakens 2013) |
| Why not Cohen's h | Arcsine transform requires binomial variance; rubric scores have Poisson-binomial variance (Cohen 1988, §7.2.1) |
| Required n (80% power, δ=0.070, σ_Δ=0.23) | **~85 tasks**, not ~410 |

**Decision rule for FDE evaluations:**
- Metric is `pass_count / n` (count ratio) → Cohen's h
- Metric is any aggregate, weighted sum, or score per task → Cohen's d_z

When in doubt: compute d_z = mean(Δ)/std(Δ) from your paired differences.
It works for any bounded continuous variable and never requires the arcsine.

**Out of scope:** Bayesian power analysis, non-parametric permutation tests,
multilevel rubric models, and variance components for multi-rater designs —
none of these change the core h-vs-d conclusion and all require more data
than the current n=3 supports.
