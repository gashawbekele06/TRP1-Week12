# Tweet Thread — How GPT-4o-mini Actually Sees a Scanned Document Page

*By Gashaw Bekele — Week 12, Agent & Tool-Use Internals*

---

**Tweet 1**
You send a scanned PDF page to GPT-4o-mini with `detail: high`.

It "reads" your table. But what did the model actually receive?

Not magic. Here is the mechanism — in three steps, with real token math. 🧵

---

**Tweet 2**
Step 1: the image becomes tokens.

Vision Transformers (Dosovitskiy et al., 2021) divide the image into 16×16 pixel
patches. Each patch is linearly projected into the same vector space as word tokens.

Image patch embeddings + text token embeddings → one sequence → one attention layer.

The word "revenue" in your prompt attends directly to the patch covering that table
cell. Same attention. Same space.

---

**Tweet 3**
Step 2: `detail: high` controls how many patches you pay for.

OpenAI's algorithm (OpenAI, 2024):
1. Resize to fit within 2048×2048
2. Scale shortest side to 768px
3. Tile in 512×512 windows
4. Tokens = 85 + (170 × tiles)

A standard scanned A4 page at 150 DPI → 6 tiles → **1,105 tokens**

`detail: low` = flat 85 tokens. 13× cheaper.
`detail: auto` (the default) = non-deterministic. Always set it explicitly.

---

**Tweet 4**
What 1,105 tokens/page means for your budget cap:

GPT-4o-mini vision input: $0.150 per 1M tokens
→ $0.000166 per page
→ a $0.10/doc cap supports ~600 pages

The real bottleneck in Strategy C is **latency** (~1–3s per page), not token cost.

Use `detail: low` for triage (85 tokens), `detail: high` only on confirmed table
pages. Cuts cost ~60% on mixed documents.

---

**Tweet 5**
Why wide tables still break even at `detail: high`:

Each tile covers ~3.4 inches at 150 DPI.
A full-width table splits across 2–3 tiles with no shared row context.

The model cannot reconstruct a row that straddles a tile boundary.

Fix: detect table bounding boxes first, pre-crop before the API call.
Higher resolution does not solve a tiling problem.

---

**Tweet 6**
Three things to remember:

1. Image = patch tokens in the same space as text, unified by one attention layer
2. Tokens = 85 + 170×tiles — know your page dimensions before setting budget caps
3. Two-pass (low-detail triage → high-detail extraction) beats one-pass always

Full explainer with token calculator and budget table: https://medium.com/@gashawbekelek/how-tool-selection-actually-works-in-function-calling-agents-3e42bb32c6e8

Sources:
→ Dosovitskiy et al. 2021 — arXiv:2010.11929 (ViT patch embedding)
→ OpenAI Vision Docs — platform.openai.com/docs/guides/vision (tiling algorithm)
→ GPT-4V System Card — cdn.openai.com/papers/GPTV_System_Card.pdf (unified attention)
