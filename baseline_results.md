# TakeMeter — Baseline Results (Milestone 4)

**Model**: `llama-3.3-70b-versatile` (Groq), zero-shot, no fine-tuning
**Test set**: 31 examples (15% of 205-example dataset, stratified split)
**Prompt**: System prompt included full label definitions from `planning.md` (analysis / hot_take / reaction), one example per label, explicit instruction to output only the label name. Full prompt text in `groq_prompt.txt`.

---

## Overall Results

| Metric              | Value        |
| ------------------- | ------------ |
| Accuracy            | 0.581        |
| Macro F1            | 0.49         |
| Weighted F1         | 0.59         |
| Parseable responses | 31/31 (100%) |

No unparseable responses — the prompt's instruction to output only the label name worked cleanly.

---

## Per-Class Metrics

| Label    | Precision | Recall | F1   | Support |
| -------- | --------- | ------ | ---- | ------- |
| analysis | 1.00      | 0.20   | 0.33 | 5       |
| hot_take | 0.76      | 0.65   | 0.70 | 20      |
| reaction | 0.31      | 0.67   | 0.42 | 6       |

---

## Observations

- **`analysis`**: Perfect precision (1.00) but very low recall (0.20) — when the model predicts `analysis`, it's always correct, but it only catches 1 of 5 true `analysis` examples. The model is highly conservative about this label, likely because the bar I set in `planning.md` (a specific, checkable number or named comparison, not just a referenced metric) is strict, and the model defaults to a different label when evidence is present but doesn't clearly meet that bar.
- **`reaction`**: Low precision (0.31), higher recall (0.67) — the inverse problem. The model over-predicts `reaction`, suggesting it may be using surface-level cues (short length, casual tone, lack of explicit stats) as a proxy for "no evaluable claim," even when the post is making an implicit assertion that should count as `hot_take`.
- **`hot_take`**: The most balanced performance (0.76 precision, 0.65 recall), consistent with it being the majority class in the dataset (132/205 examples, ~64%).

## Hypothesis (to test after fine-tuning)

The baseline struggles most at the `analysis` / `hot_take` boundary and the `hot_take` / `reaction` boundary — exactly the two boundaries flagged as hardest in `planning.md`'s edge case analysis (Section 3). Specifically:

1. The model appears to lack a reliable way to detect when a post _references_ evidence without truly _providing_ it (the "named metric, no number" edge case), defaulting to under-predicting `analysis` as a result.
2. The model appears to use emotional/casual tone as a proxy signal for `reaction`, which causes it to misclassify some `hot_take` posts that are confidently asserted but stylistically casual.

I'll check after fine-tuning whether a model trained directly on my annotated examples learns a sharper distinction here, particularly whether it picks up on the "specific number vs. named-but-unquantified metric" distinction that the zero-shot model seems to miss.
