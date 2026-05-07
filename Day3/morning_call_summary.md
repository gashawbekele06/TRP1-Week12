# morning_call_summary.md — Day 3

**Written by:** Nebiyou Abebe
**Confirmed by:** Gashaw Bekele
**Call duration:** ~25 minutes
**Date:** 2026-05-07

---

## What Was Ambiguous in the Original Drafts

**Gashaw's original draft question** was:
*"My LoRA adapter learned to shorten outputs but not to suppress banned phrases.
Is this because r=16 is too low a rank to represent token suppression?"*

Nebiyou's interrogation surfaced two problems:

1. **The question conflated two separate hypotheses without distinguishing them.**
   Nebiyou asked: *"You're assuming the failure is about rank capacity. But what if
   the problem is the training objective itself, not the subspace? What would be
   different in your experiment if it were objective mismatch instead of rank limit?"*
   Gashaw had not separated these two explanations in the draft.

2. **"Is r=16 too low?" is answerable with a number, not a mechanism.**
   Nebiyou pushed: *"What is the question behind the question? You want to know
   whether these two behaviors — length reduction and token suppression — are even
   the same class of transformation, right? Ask that."* This forced Gashaw to reframe
   from a hyperparameter question to a transformation-class question.

## How the Question Was Sharpened

The final question now separates the two explanations cleanly — rank capacity vs
objective mismatch — and asks whether token suppression is a fundamentally different
transformation from length reduction at the level of what a weight update can express.
This is answerable through research (LoRA paper, ORPO, unlikelihood training) without
requiring Nebiyou to run Gashaw's specific experiment.

Nebiyou confirmed the final question is unambiguous and resolvable in 600–1,000 words.
