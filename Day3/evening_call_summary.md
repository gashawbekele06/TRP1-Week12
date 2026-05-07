# evening_call_summary.md — Day 3

**Written by:** Gashaw Bekele
**Confirmed by:** Nebiyou Abebe
**Call duration:** ~35 minutes
**Date:** 2026-05-07

---

## Feedback Gashaw Gave on Nebiyou's Explainer (LoRA rank question)

Gashaw read Nebiyou's explainer and experiment.py before the call and brought
three specific pieces of feedback:

1. **The framing of "objective mismatch vs rank" needed to come earlier.**
   The short answer section stated "objective mismatch, not a hard rank limit"
   but the reasoning behind it did not appear until three sections later. Gashaw
   asked: *"Can you move the ORPO argument — that SFT has no negative signal —
   earlier so the reader knows where you are going before the linear algebra?"*
   This was the biggest structural gap in the first draft.

2. **The experiment was the strongest part but was buried.**
   The five-trial override test (Approach A: 5/5 produced the banned word;
   Approach B: 0/5 despite the same instruction) is the clearest proof of
   the mechanism. Gashaw flagged it should be referenced in the summary table
   rather than appearing only as a long code block at the end.

3. **"Diagnostic experiments Gashaw should run next" was out of scope.**
   Gashaw said: *"This section is useful but it answers a different question —
   what to do next. The question was about why the asymmetry happened, not
   what experiments to run. It risks diluting the central answer."*
   Nebiyou agreed to keep it brief and clearly labelled as an extension.

## Feedback Nebiyou Gave on Gashaw's Explainer (power analysis question)

1. **The decision table was the most useful part** — Nebiyou said it directly
   answered the question of when to investigate training vs eval size.
   No revision requested on the table.

2. **"Effect size uncertainty" section needed a concrete example.**
   Nebiyou asked: *"The 95% CI of −2pp to +30pp — where does that come from?
   Show the calculation or cite it."* Gashaw added the Wilson interval formula
   reference to the final version.

3. **Tweet 3 was slightly over 280 characters** on Nebiyou's phone preview.
   Gashaw trimmed "already sufficient" to "✅ sufficient" to stay clean.

## Asker Judgments

- Gashaw on Nebiyou's explainer: **gap closed** — see day3_signoff.md
- Nebiyou on Gashaw's explainer: **gap closed** (confirmed verbally on call)
