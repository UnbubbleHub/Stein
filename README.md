# Stein

**A language-agnostic news-article classifier — a single LLM prompt that labels any article by its editorial posture and format.**

Stein reads an article's text and assigns a two-level label: a **macro category** (its posture toward reality — does the piece *report*, *interpret*, or *take a position*?) and a **specific type** (its editorial format). It works in any language and classifies from the article body, not from keywords or the URL.

📄 Full write-up: [Stein on Unbubble](https://unbubblehub.substack.com/p/f97525ad-abb9-4c58-999f-4fb466146374)

## The taxonomy — 3 macros, 8 types

| Macro | Posture | Types |
|---|---|---|
| **FACT** | **Reports** — who / what / when / where, no interpretation | Breaking News · Quote · Liveblog |
| **FRAME** | **Interprets** — explains, contextualizes, quantifies, verifies | News Analysis · Explainer · Poll Report |
| **VIEW** | **Takes a position** — judgment or recommendation in the author's voice | Opinion · Review |

Plus `EXCLUDED` for betting / odds / prediction-market content.

It's a **closed set of 8** — interviews and institutional statements collapse into *Quote*, fact-checks and debunks into *News Analysis*, and so on. No other labels are emitted.

## How it works

- **Body-authoritative** — classifies from the article text; title and URL are only fast priors and tie-breakers. When they conflict with the body, the body wins.
- **Language-agnostic** — recognizes editorial *structure and intent*, not language-specific keywords; validated across Italian, English, German, Spanish, and French. Labels are always emitted in English.
- **Structural decision procedure** — precedence-ordered rules with explicit tie-breakers: a *remove-the-quotes* test for Breaking News vs Quote, a *lede test* for reporting vs interpreting, an *object test* for Review vs Opinion.
- **Structured output** — strict JSON: the macro, the type, two confidence scores (macro and type), and a one-line reason.

The entire classifier is the single prompt in [`prompt.md`](prompt.md) — drop it into any capable LLM.

## Results

Stein was validated against a **human-labeled gold set**: a stratified, multilingual sample hand-labeled by an editor, then compared against the model's automated labels (with an independent stronger-model cross-check).

- Running on **full article text**, the production classifier agrees with the human gold on about **89% of the macro posture** and about **83% of the exact type** (among the eight).
- **Reliable on essentially every case:** Quote, Poll Report, Liveblog, Review.
- **Where the residual divergence lives:** the Breaking News ↔ News Analysis ↔ Explainer boundary — i.e. *reporting* vs *interpreting* vs *teaching* around the same event. The current prompt encodes explicit tests (the lede test and a pedagogical-format rule) to resolve most of these.
- Applied to a multilingual corpus of ~1,700 news articles, the posture split was roughly **FACT 67% / FRAME 27% / VIEW 6%**.

A key methodological lesson: **model-vs-model agreement overstates accuracy** — two models tend to share the same biases and agree with each other. Only the human gold set surfaced the true boundary gaps and drove the prompt's final calibration.

## Files

- [`prompt.md`](prompt.md) — the complete classifier prompt: taxonomy, decision procedure, worked examples, and a language-agnostic signal appendix.

## Not included

The evaluation dataset and the per-article labels are not part of this repository — it contains the classifier and its aggregate results only.
