# evening_call_summary.md

**Written by:** Gashaw Bekele  
**Confirmed by:** Meseret  
**Call duration:** ~30 minutes  
**Date:** 2026-05-04  

---

## Feedback Gashaw Gave on the Explainer

Gashaw read Meseret's explainer draft independently before the call and brought
three specific pieces of feedback:

1. **Section 2c (dropout failure) needed a clearer failure signal.**
   The draft said "dropout left on causes issues" but did not show what the
   observable symptom looks like. Gashaw asked: *"If I ship this and the dropout
   is on, what do I actually see? Does the output change every call? Do scores
   drop? How would I know?"* This was the part of the gap Gashaw was most
   confused about and the draft passed over it too quickly.

2. **The code example in Section 3 was missing the `model.eval()` call** in the
   unmerged path — exactly the bug the explainer was warning against. Gashaw
   flagged it: *"You're demonstrating the fix but the code doesn't show the fix."*
   This was a critical correction before the artifact could ship publicly.

3. **Tweet 5 was too long** to read as a standalone tweet. The FDE rule and the
   honest null-delta answer were both correct but competed for space.
   Gashaw suggested keeping the FDE rule in Tweet 5 and moving the null-delta
   acknowledgment to a shorter note inside Tweet 6.

## What Meseret Revised

1. Section 2c was expanded with one sentence describing the observable symptom:
   non-deterministic outputs on identical prompts in unmerged mode, with token
   probabilities shifting between calls — the signal that dropout is active.

2. The code example in Section 3 was corrected to include `model_unmerged.eval()`
   explicitly before the forward pass, with an inline comment marking it as critical.

3. Tweet 5 was trimmed to the FDE rule only; the null-delta note was compressed
   to one line appended to Tweet 6.

## Asker Judgment

After revisions: **gap closed**. See `signoff.md`.
