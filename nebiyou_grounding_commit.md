# grounding_commit.md — Nebiyou Abebe

**Asker:** Nebiyou Abebe
**Date:** 2026-05-07
**Artifact to edit:** Week 11 evaluation methodology / model card eval section

---

## Pointer to the Edit

Nebiyou's Week 11 eval section currently reports the +14 pp lift (12% → 26%)
with p=0.23 and characterises it as not significant, leaving the interpretation
ambiguous — was it a training failure or a measurement failure?

**Edit:** Add one paragraph to the eval section of the Week 11 model card:

> "The initial evaluation at n=50 returned p=0.23. A power analysis (Cohen's h=0.36,
> required n=48 one-tailed) showed n=50 had 73% power for a two-tailed test —
> below the 80% threshold. Re-evaluation at n=100 confirmed p < 0.05: the +14 pp
> lift is statistically significant and the training objective was effective.
> The residual performance gap reflects a capacity ceiling on the 0.6B backbone,
> not a training failure. Future experiments should use n ≥ 100 as the minimum
> eval size for pass-rate comparisons at this effect-size range."

## Why This Matters

Without this edit, any reader of the Week 11 model card sees "p=0.23, not
significant" and reasonably concludes the ORPO training did not work. That
conclusion is wrong. The edit corrects the record with the confirmed outcome
and documents the standard the Week 12 research established: n ≥ 100 for
this class of evaluation.

It also makes the "capacity ceiling on 0.6B" finding the correct next question
— not "did training work?" (answered: yes) but "how far can this backbone go?"
