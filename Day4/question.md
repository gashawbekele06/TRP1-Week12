# day4_question.md

**Asker:** Gashaw Bekele
**Topic area:** Evaluation and statistics — Cohen's h vs Cohen's d for rubric scores
**Date:** 2026-05-08

---

## The Question

My Week 11 ablation corrected Delta A = +0.070 (trained vs baseline, n=3).
To plan the next eval run I need a power analysis — but I hit a fork:

- **Nebiyou's case** (pass rate 12%→26%) → used Cohen's h = 0.36 → needed n=48
- **My case** (overall score 0.531→0.601) → scores are weighted averages of
  4 binary checks (weights 0.285 / 0.285 / 0.095 / 0.05), not raw proportions

The two formulas give different answers at my observed delta:

| Formula | Effect size | n for 80% power |
|---|---|---|
| Cohen's h | 0.14 | ~410 tasks |
| Cohen's d | 0.30 | ~85 tasks |

**Specific gap:** Is a weighted average of binary rubric checks still a
"proportion" (use h) or a bounded continuous variable (use d)? Using the
wrong formula either wastes 325 inference calls or leaves the eval underpowered.

---

## Explainer Scope

1. What statistical class is a weighted sum of binary outcomes — proportion or continuous?
2. When does Cohen's h apply vs Cohen's d — what is the boundary condition?
3. Which formula fits my rubric (4 binary checks, weights above) and why?
4. The correct required-n for my case (Delta=0.070, std of paired differences ≈ 0.23).

**Out of scope:** Python code, Bayesian methods, CI construction — conceptual answer only.
