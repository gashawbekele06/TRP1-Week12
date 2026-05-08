# evening_call_summary.md — Day 4

**Written by:** Gashaw Bekele
**Confirmed by:** Eyobed Feleke
**Call duration:** ~30 minutes
**Date:** 2026-05-08

---

## Feedback Gashaw Gave on Eyobed's Explainer (h vs d_z question)

Gashaw read Eyobed's explainer and sources.md before the call and brought
three specific pieces of feedback:

1. **The short answer needed the decision rule up front.**
   The first draft opened with "use Cohen's d_z, not h" but did not give the
   one-sentence rule until the summary. Gashaw asked: *"Can you put the
   decision rule — 'count ratio → h, aggregate score → d_z' — right after the
   short answer? That is the thing I will actually remember."* Eyobed agreed
   and moved the rule to the end of Section 1.

2. **The score range [0, 0.715] needed explanation.**
   The first draft mentioned scores are bounded in [0, 0.715] without
   explaining why. Gashaw flagged: *"A reader who has not seen my rubric will
   not know why it is not [0, 1]. It is because tone_judge is skipped — say that
   explicitly."* Eyobed added the dimension table with the skipped row.

3. **The power table was the most useful part.**
   Gashaw said the column showing 10.4% power at n=3 was the clearest
   proof that the Week 11 null result was a measurement failure, not a model
   failure. No revision requested on the power table.

---

## Feedback Eyobed Gave on Gashaw's Explainer (Eyobed's question)

1. **Wilson CI calculation was the strongest section.**
   Eyobed confirmed the [80.6%, 97.5%] result matched what he got when he
   ran the formula himself. He asked for the two-proportion z-test p-value
   to be shown explicitly — Gashaw had computed p=0.047 but not printed it
   in the original draft. Added to Section 2.

2. **The calibration code block needed a cleaner example.**
   Eyobed said: *"The correctness proxy in the loop (`trained_score >=
   baseline_score`) is not the right check — it should compare against the
   ground truth label, not the other condition's score."* Gashaw revised the
   calibration section to note the proxy limitation and recommend running it
   only after the ground-truth labels are added to held_out_traces.jsonl.

3. **"Out of scope" section was appreciated.**
   Eyobed noted that explicitly ruling out ROC/AUC and Platt scaling was
   useful — it confirmed the explainer was scoped to what n=41 can support.
   No revision requested.

---

## Asker Judgments

- Gashaw on Eyobed's explainer: **gap closed** — see day4_signoff.md
- Eyobed on Gashaw's explainer: **gap closed** (confirmed verbally on call)
