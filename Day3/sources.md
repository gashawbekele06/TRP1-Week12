# sources.md

**Explainer:** Gashaw Bekele
**Topic:** Statistical power for proportion tests in LLM evaluation
**Date:** 2026-05-07

---

## Canonical Source 1

**Cohen, Jacob. (1988). Statistical Power Analysis for the Behavioral Sciences (2nd ed.).**
Lawrence Erlbaum Associates.

**Why this is load-bearing:**
Cohen (1988) is the original and definitive reference for effect size definitions
and power analysis. The Cohen's h formula used in the explainer —
`h = 2·arcsin(√p₂) − 2·arcsin(√p₁)` — is defined in Chapter 7
(Differences Between Proportions). The benchmarks for small (h=0.20),
medium (h=0.50), and large (h=0.80) effects come from Chapter 2.
The required-n formula `n = ((z_α + z_β) / h)²` is derived there.
No second-hand summary was used — the formula and benchmarks are
cited from the primary source.

**Relevance to Nebiyou's question:** The h=0.3627 computed for his 12%→26%
lift classifies as a medium-small effect by Cohen's scale, confirming that
the required n is in the range of 48–60, not hundreds.

---

## Canonical Source 2

**scipy.stats documentation — scipy.stats.norm and proportion power analysis.**
SciPy 1.x Official Documentation. https://docs.scipy.org/doc/scipy/reference/stats.html

**Why this is load-bearing:**
The `norm.cdf()` function from SciPy is the authoritative implementation of
the standard normal CDF used to compute power at a given n:
`power = Φ(h·√n − z_α)`. The code in Section 3 of the explainer runs against
this library and produces verifiable output. The results (82.1% power at n=50
one-tailed, 72.7% two-tailed, 97.6% at n=100) are reproducible by any reader
who runs the code block.

**Why not statsmodels:** statsmodels `NormalIndPower` is the more common
citation for this calculation but requires an additional install. SciPy is
part of the standard scientific Python stack and the formula is transparent
when implemented manually, making the mechanism visible rather than hidden
inside a function call.

---

## Tool / Calculation Used

**Tool:** Python 3.x + scipy.stats.norm (standard library in scientific Python)

**What was run:** The four-line power calculation in Section 3, verified against
the formula from Cohen (1988) Chapter 7. Output confirmed:
- Cohen h = 0.3627 for p₁=0.12, p₂=0.26
- n required (one-tailed, 80% power) = 48
- Power at n=50 (one-tailed) = 82.1%
- Power at n=50 (two-tailed) = 72.7%
- Power at n=100 = 95–98% across both test variants

**Reproducibility:** Any reader can run the code block in Section 3 with
`pip install scipy` and verify every number in the explainer.

---

## Attribution

All numeric claims in `nebiyou_explainer.md` trace to either Cohen (1988)
(formula derivation) or the SciPy calculation above (computed values).
The decision table in Section 4 is derived reasoning from these sources,
not from a separate citation. No hallucinated references.
