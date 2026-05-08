# thread.md — Tweet Thread (Twitter-ready)

**Author:** Gashaw Bekele
**Topic:** Evaluation statistics — Cohen's h vs Cohen's d_z for LLM rubric scores
**Answers:** @EyobedFeleke's question on effect size for weighted rubric evaluation
**Date:** 2026-05-08
**Instructions:** Copy each block exactly. Post in order 1 → 6. Reply to each tweet to keep the thread connected.

---

## Tweet 1 / 6 — Hook
*(~268 chars)*

```
My pair @EyobedFeleke asked:
"Should I use Cohen's h or Cohen's d for my power analysis?"

His rubric score: weighted average of 4 binary checks.

Cohen's h said he needs 410 tasks.
Cohen's d said 85 tasks.

Only one is right — and the gap matters. 🧵
```

---

## Tweet 2 / 6 — The Mechanism
*(~262 chars)*

```
Cohen's h uses the arcsine transform φ = 2·arcsin(√p).

Why? Binomial variance p(1−p)/n varies with p.
The transform stabilises it so sample-size formulas stay valid.

But that only works when your metric IS a proportion: count/n.
If it's not, the transform breaks.
```

---

## Tweet 3 / 6 — The Key Distinction
*(~274 chars)*

```
A weighted rubric score is NOT a proportion.

S = (0.285·X₁ + 0.285·X₂ + 0.095·X₃ + 0.05·X₄) / 0.715

Each Xᵢ is binary, but weights differ → Poisson-binomial mixture.
Variance = Σ(wᵢ/W)²·pᵢ(1−pᵢ)

The arcsine transform is not calibrated for this. Cohen's h inflates n by 5×.
```

---

## Tweet 4 / 6 — The Correct Formula
*(~258 chars)*

```
Correct formula: Cohen's d_z for paired designs (Lakens 2013).

d_z = mean(Δ) / std(Δ)
    = 0.070 / 0.230 = 0.304

Power at n=3:   10.4%  ← explains the null result
Power at n=85:  80.1%  ← the real target
Power at n=100: 85.4%

Required n = 85, not 410.
```

---

## Tweet 5 / 6 — The Decision Rule
*(~244 chars)*

```
Decision rule for LLM evaluations:

Metric = passing_tasks / total_tasks
→ Raw proportion → Cohen's h ✅

Metric = aggregate score, weighted rubric, or per-task mean
→ Bounded continuous → Cohen's d_z ✅

When in doubt: compute d_z = mean(Δ)/std(Δ) from your paired differences.
```

---

## Tweet 6 / 6 — Confirmed + Link
*(~262 chars + URL)*

```
Update from @EyobedFeleke: confirmed.
Required n = 85 tasks. Target is achievable.

Collecting 410 would have been a 5× waste.
Stopping at 50 would have left the eval at 58% power.

Getting the formula right IS the evaluation design.

Full explainer 👇
[ADD BLOG URL HERE]

Sources: Cohen (1988) · Lakens (2013, Frontiers in Psychology)
```

---

## Pre-post checklist

- [ ] Replace `[ADD BLOG URL HERE]` in Tweet 6 with actual blog post URL
- [ ] Post Tweet 1 first, then reply to each in sequence (1 → 2 → 3 → 4 → 5 → 6)
- [ ] No markdown formatting — plain text + emojis only
- [ ] Tag @EyobedFeleke in Tweet 1 so he sees his question being answered publicly
