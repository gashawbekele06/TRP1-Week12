# thread.md — Tweet Thread (Twitter-ready)

**Author:** Gashaw Bekele
**Topic:** Evaluation statistics — statistical power for LLM pass-rate comparisons
**Answers:** @NebiyouAbebe's question on ORPO LoRA evaluation
**Date:** 2026-05-07
**Instructions:** Copy each block exactly. Post in order 1 → 6. Reply to each tweet to keep the thread connected.

---

## Tweet 1 / 6 — Hook
*(~248 chars)*

```
My pair @NebiyouAbebe trained an ORPO LoRA adapter.
Pass rate jumped 12% → 26%. A +14pp lift.
p-value: 0.23. "Not significant."

Before blaming the training setup — is n=50 even enough
to see a real +14pp lift?

The answer surprised me. 🧵
```

---

## Tweet 2 / 6 — The Mechanism
*(~245 chars)*

```
Statistical power = the probability your test detects
a real effect when it exists.

Low power → high p-value even when the lift IS real.
This is a Type II error: you miss the signal, not
because the training failed, but because the eval
was too small to see it.
```

---

## Tweet 3 / 6 — The Key Number
*(~272 chars)*

```
For a 12% → 26% lift, Cohen's h = 0.36 (medium effect).

Required n for 80% power:
→ One-tailed test: n = 48
→ Two-tailed test: n = 60

At n=50:
→ One-tailed power: 82% ✅ already sufficient
→ Two-tailed power: 73% ⚠️ just below threshold

The sample size question depends on which test you ran.
```

---

## Tweet 4 / 6 — The Calculation
*(~252 chars)*

```
import math
from scipy.stats import norm

p1, p2 = 0.12, 0.26
h = 2*(math.asin(math.sqrt(p2)) - math.asin(math.sqrt(p1)))

def power(n, z_a):
    return norm.cdf(h*math.sqrt(n) - z_a)

# n=50, one-tailed → 82%
# n=50, two-tailed → 73%
# n=100, both     → 95%+
```

---

## Tweet 5 / 6 — The Decision Rule
*(~268 chars)*

```
Minimum diagnostic experiment: run 100 examples.
Cost: 50 more inference calls.

At n=100, both test variants clear 95% power.
Then read the result:

p < 0.05 → original was sample-size limited. Lift is real.
Δ shrinks to < 8pp → investigate training: pairs, epochs, objective.
p > 0.05, Δ still ~14pp → check eval rubric stability.
```

---

## Tweet 6 / 6 — Honest Punchline + Link
*(~255 chars + URL)*

```
Key insight: "not significant" ≠ "no effect."

With n=50 and 73% power (two-tailed), there is a 27%
chance of missing a real +14pp lift.

Blame the training only after n=100 rules out
the sample-size explanation.

Full explainer 👇
[ADD BLOG URL HERE]

Source: Cohen (1988) · scipy.stats.norm
```

---

## Pre-post checklist

- [ ] Replace `[ADD BLOG URL HERE]` in Tweet 6 with actual blog post URL
- [ ] Post Tweet 1 first, then reply to each in sequence (1 → 2 → 3 → 4 → 5 → 6)
- [ ] No markdown formatting — plain text + emojis only
- [ ] Tag @NebiyouAbebe in Tweet 1 so he sees his question being answered publicly
