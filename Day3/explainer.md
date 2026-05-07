# explainer.md

**Explainer:** Gashaw Bekele
**Asker:** Nebiyou Abebe
**Topic:** Evaluation statistics — statistical power for proportion tests
**Date:** 2026-05-07

---

## The Short Answer

Your n=50 is already sufficient to detect a genuine +14 pp lift with 80% power
— if you are running a one-tailed test. The p=0.23 most likely means either
(a) you used a two-tailed test where n=60 is the threshold, or (b) the true
effect size is smaller than the 14 pp point estimate suggests. Running 100
examples clears both doubts. If p > 0.05 at n=100, the training setup deserves
scrutiny — not the eval size.

---

## 1. The Load-Bearing Mechanism: Statistical Power

Statistical power is the probability that a test correctly detects an effect
that is real. When power is low, a genuine improvement can produce a large
p-value — not because the training failed, but because the experiment was
too small to see the signal.

Power depends on three things:

- **Effect size** — how large the true difference is
- **Sample size n** — how many evaluation examples you ran
- **Significance threshold α** — how strict your pass/fail line is (usually 0.05)

The relationship is: bigger effect or bigger n → higher power. If power is
below 80%, a p-value above 0.05 is uninformative. It does not mean "no effect."
It means "experiment too small to tell."

---

## 2. Translating Your Numbers

Your observed effect: baseline pass rate p₁ = 0.12, trained p₂ = 0.26, Δ = 0.14.

**Cohen's h** is the standard effect-size measure for proportion differences:

```
h = 2 × arcsin(√p₂) − 2 × arcsin(√p₁)
  = 2 × arcsin(√0.26) − 2 × arcsin(√0.12)
  = 2 × 0.5326 − 2 × 0.3523
  = 1.0652 − 0.7047
  = 0.3605   (medium effect by Cohen's 1988 benchmarks)
```

**Required n for 80% power** at α = 0.05:

```
n = ((z_α + z_β) / h)²

One-tailed (we expect trained > baseline):
  z_α = 1.645,  z_β = 0.842
  n = ((1.645 + 0.842) / 0.3605)² = (2.487 / 0.3605)² = 6.9² ≈ 48

Two-tailed (no direction assumed):
  z_α = 1.960,  z_β = 0.842
  n = ((1.960 + 0.842) / 0.3605)² = (2.802 / 0.3605)² = 7.8² ≈ 60
```

**Power at your current n=50:**

```
One-tailed:  power = Φ(h√n − z_α) = Φ(0.3605×7.07 − 1.645) = Φ(0.904) ≈ 82%
Two-tailed:  power = Φ(h√n − z_α) = Φ(0.3605×7.07 − 1.960) = Φ(0.589) ≈ 72%
```

**What this tells you:** At n=50, a one-tailed test already has 82% power.
If p=0.23 with a one-tailed test, the true effect is probably smaller than 14 pp.
If you used a two-tailed test, power was only 72% — just below the 80% threshold.
The minimum fix is to clarify which test you ran and expand to n=100.

---

## 3. Worked Demonstration

```python
import math
from scipy.stats import norm

p1, p2 = 0.12, 0.26

# Cohen's h
h = 2 * (math.asin(math.sqrt(p2)) - math.asin(math.sqrt(p1)))

# Required n (one-tailed, 80% power)
z_alpha, z_beta = 1.645, 0.842
n_required = ((z_alpha + z_beta) / h) ** 2

# Power at different n values
def power_at_n(n, h, z_alpha):
    return norm.cdf(h * math.sqrt(n) - z_alpha)

print(f"Cohen h        : {h:.4f}")
print(f"Required n     : {math.ceil(n_required)}")
print(f"Power at n=50  : {power_at_n(50,  h, 1.645)*100:.1f}%  (one-tailed)")
print(f"Power at n=50  : {power_at_n(50,  h, 1.960)*100:.1f}%  (two-tailed)")
print(f"Power at n=100 : {power_at_n(100, h, 1.645)*100:.1f}%  (one-tailed)")
print(f"Power at n=100 : {power_at_n(100, h, 1.960)*100:.1f}%  (two-tailed)")
```

**Output:**
```
Cohen h        : 0.3627
Required n     : 48
Power at n=50  : 82.1%  (one-tailed)
Power at n=50  : 72.7%  (two-tailed)
Power at n=100 : 97.6%  (one-tailed)
Power at n=100 : 95.2%  (two-tailed)
```

At n=100, both test variants clear 95% power. There is no ambiguity left.

---

## 4. The Decision Rule

**Step 1 — Run 100 examples before doing anything else.**
This costs one inference pass over 50 additional examples. It eliminates
the sample-size question entirely for both one-tailed and two-tailed tests.

**Step 2 — Interpret the result at n=100:**

| Result at n=100 | Interpretation | Next action |
|---|---|---|
| p < 0.05 | Original result was sample-size limited. The lift is real. | Ship or widen eval to confirm. |
| p > 0.05, Δ still ~14 pp | Effect is too noisy to be reliable. High variance in pass/fail. | Investigate eval rubric stability first. |
| p > 0.05, Δ shrinks to < 8 pp | Original 14 pp was a lucky sample. Small or no true lift. | Investigate training: pair quality, epochs, ORPO objective fit. |

**Step 3 — When to blame the training setup:**
Only when n ≥ 100 and the lift disappears or shrinks to < 8 pp. Before
that point, the sample size is a legitimate alternative explanation and
pursuing training-side fixes is premature.

---

## 5. Two Adjacent Concepts Worth Knowing

**Why "not significant" ≠ "no effect"**
A p-value above 0.05 is not evidence that the effect is zero. It is evidence
that you cannot distinguish the observed difference from chance at the chosen
threshold. With n=50 and 72% power, you have a 28% chance of missing a real
14 pp lift. Reporting p=0.23 as "the adapter did not improve performance" is
a Type II error claim that the data do not support.

**Effect size uncertainty**
The 14 pp estimate comes from 50 binary observations (6 → 13 passes). The
95% confidence interval around a 14 pp difference at n=50 spans roughly
−2 pp to +30 pp. The true effect could be anywhere in that range. This is
why expanding n is the only principled first step — it narrows the interval
before you interpret the direction.

---

## Summary

| Question | Answer |
|---|---|
| Is n=50 enough to detect +14 pp at 80% power? | Yes (one-tailed: 82%). No (two-tailed: 73%). |
| Minimum diagnostic experiment | Run 100 examples. Cost: 50 additional inferences. |
| When to investigate training side | Only if Δ < 8 pp or p > 0.05 at n=100. |
| What p=0.23 most likely means | Two-tailed test used, OR true effect < 14 pp. Not "no effect." |
