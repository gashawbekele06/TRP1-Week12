# grounding_commit.md — Day 3

**Asker:** Gashaw Bekele
**Date:** 2026-05-07
**Artifact edited:** `tenacious-bench/methodology_rationale.md`

---

## Pointer to the Edit

**File:** `C:\Users\gasha\OneDrive\Desktop\TRP1\Week11\tenacious-bench\methodology_rationale.md`
**Section revised:** "Honest Limitation" — existing paragraph updated and a new
subsection added titled **"Training Objective Diagnosis (Week 12 Day 3 Addition)"**

---

## What Changed and Why

**Before this edit**, the Honest Limitation section said:

> *"This is a capacity limitation of the backbone, not a pipeline failure."*

That diagnosis was correct in ruling out a pipeline failure but imprecise about
which dimension of capacity was the bottleneck. It pointed to backbone size (0.5B)
as the explanation without distinguishing between:
- **Rank capacity** — whether a rank-16 subspace can represent token suppression
- **Objective capacity** — whether SFT's positive-only loss can enforce a hard
  negative constraint regardless of rank or model size

A reader of the original text would conclude: "use a bigger model." That is
partially right but misses the more actionable fix.

**After this edit**, the section now states:

The primary bottleneck is the **training objective**, not backbone size alone.
Standard SFT maximises the likelihood of chosen outputs without any gradient
signal that explicitly penalises the rejected token ("bench"). ORPO (Hong et al.,
2024) demonstrates that SFT on chosen responses increases log-probability of both
chosen and rejected responses simultaneously — meaning the banned phrase remains
likely even after training on 221 examples that exclude it.

The recommended fix for v0.2 is therefore:
1. **First priority:** Add decoding-layer enforcement via `bad_words_ids` or
   `NoBadWordsLogitsProcessor` — requires no retraining, directly blocks the
   token at generation time regardless of what the adapter learned.
2. **Second priority:** Retrain with ORPO using rejected responses that contain
   the banned phrase, giving an explicit negative gradient signal.
3. **Third priority:** Increase backbone size (Qwen2.5-1.5B) only if (1) and (2)
   do not fully resolve the constraint — larger models are more capable but do not
   change the fundamental objective mismatch.

This edit makes the diagnosis in methodology_rationale.md defensible against the
most obvious engineering pushback: "why not just use a bigger model?" The answer
is now: because the problem is not model size, it is training signal class.
