# TakeMeter

A fine-tuned text classifier that evaluates discourse quality in r/nba. Given a post or comment from the subreddit, TakeMeter classifies it as `analysis` (evidence-backed argument), `hot_take` (confident opinion without evidence), or `reaction` (no evaluable claim — pure emotion or open question).

## Community

r/nba is one of the largest NBA discussion communities on Reddit. Discourse quality varies dramatically post to post: the same thread can contain a detailed statistical breakdown, an all-caps reaction to a dunk, and a confidently-asserted opinion with no backing. The community itself already distinguishes between these — members call out "hot takes" and upvote "film breakdowns" — making it a natural fit for a discourse quality classifier.

## Labels

Three mutually exclusive labels, assigned by asking in order: (1) does this provide specific, checkable evidence? → `analysis`. (2) Does this assert a bold claim without checkable evidence? → `hot_take`. (3) Otherwise → `reaction`.

**`analysis`** — A post that makes a structured argument backed by specific, checkable evidence: a real statistic with a number, a named historical comparison, or a concrete tactical observation where the evidence is part of the reasoning, not decoration. Naming a metric without providing the actual value does not count.

- "SGA's true shooting percentage in clutch situations this season is 63.2%, which ranks top-3 all-time among players with 200+ clutch possessions."
- "Even though it was a 4-1 series, it was still 5 very good games. 10, 1, 4, 1, 4 point victories. That is a close series."

**`hot_take`** — A bold, confident opinion stated without specific, checkable evidence. Includes posts that reference a stat by name without supplying the number, and posts that generalize beyond what a cited event supports.

- "LeBron was never clutch. Anyone who watched the 2011 Finals knows what he is."
- "The threshold for the last time that 'real basketball' was played keeps moving forward every year."

**`reaction`** — A post that makes no evaluable claim at all. Covers immediate emotional responses to live events and genuine open-ended questions where the poster doesn't signal a preferred answer.

- "HE CROSSED HIM UP AND THEN DUNKED ON HIM ARE YOU KIDDING ME"
- "Who do you think wins Finals MVP if Haliburton stays healthy through Game 7?"

**Design note**: The original taxonomy had four labels, including a separate `discussion` category for open questions. During data collection, `discussion` consistently came in under 10% across multiple source types. Rather than artificially inflate it, I merged it into `reaction`, since both share the property of making no evaluable claim. See `planning.md` Section 2 for the full rationale.

## Dataset

**Source**: 205 labeled posts and comments from r/nba, collected manually (copy-paste) from game threads, post-game celebration threads, and play-in tournament discussion threads. Manual collection was used because Reddit's Data API currently requires an application review process for new script apps.

**Labeling process**: I used Claude as a pre-labeling assistant — I pasted batches of raw, unlabeled text, and Claude suggested a label and reasoning for each example based on the decision rules in `planning.md`. I reviewed every suggestion individually and made my own judgment call on ambiguous cases. Final labels reflect my review, not the raw suggestions.

**Label distribution**:

| Label    | Count | Percentage |
| -------- | ----- | ---------- |
| hot_take | 132   | 64.4%      |
| reaction | 43    | 21.0%      |
| analysis | 30    | 14.6%      |

**Train/val/test split**: 70% / 15% / 15% (stratified), handled automatically by the Colab notebook with `random_state=42`. Test set: 31 examples.

**Three difficult labeling decisions**:

1. _"I don't think this is actually the answer. The amount of fouls per game is generally pretty constant (it's actually kinda a low average right now)..."_ — References a real, checkable trend but gives no exact number. I labeled it `analysis` because the claim is falsifiable (the trend either exists or doesn't), but this sits at the softest edge of the `analysis` boundary.

2. _"He's good, BUT are you better than him?"_ — A rhetorical jab that's technically a question but functionally a joke. Not a genuine open question, not an emotional reaction to a live event, not an asserted claim. I kept it as `reaction` because it fails the "evaluable claim" test, but this exposed a gap in my taxonomy for low-effort rhetorical asides.

3. _"Do the warriors always set this many moving screens for Steph? If so, then his mobility is a little less impressive..."_ — Starts as a question but the second half commits to an evaluative conclusion. I kept `reaction` because the core claim is conditional ("if so"), but a different annotator could reasonably call this `hot_take`.

## Fine-Tuning Pipeline

**Base model**: `distilbert-base-uncased` (HuggingFace), a 66M-parameter distilled version of BERT.

**Training approach**: Standard sequence classification fine-tuning using HuggingFace `Trainer` on Google Colab's free T4 GPU. Training took approximately 23 seconds (27 steps across 3 epochs).

**Hyperparameters** (defaults from the starter notebook):

| Parameter           | Value |
| ------------------- | ----- |
| Epochs              | 3     |
| Learning rate       | 2e-5  |
| Batch size          | 16    |
| Max sequence length | 256   |

**Key hyperparameter observation**: I used the default hyperparameters without modification. The training loss decreased steadily across epochs (1.075 → 1.028) and validation loss also decreased (1.057 → 0.939), indicating no overfitting. However, validation accuracy stayed frozen at 0.645 across all three epochs — the model grew more confident in its predictions without changing which predictions it made. In hindsight, this was an early signal of the majority-class collapse described in the evaluation below: the model was converging toward always predicting `hot_take` and simply becoming more certain about that single strategy.

## Baseline Comparison

**Baseline model**: Groq `llama-3.3-70b-versatile`, zero-shot, with a system prompt containing the full label definitions from `planning.md` and one example per label.

### Overall Accuracy

| Model                     | Accuracy |
| ------------------------- | -------- |
| Zero-shot baseline (Groq) | 0.581    |
| Fine-tuned DistilBERT     | 0.645    |

The fine-tuned model's accuracy is higher, but this number is misleading — see the evaluation report below.

## Evaluation Report

### Per-Class Metrics

**Zero-shot baseline (Groq)**:

| Label         | Precision | Recall   | F1       | Support |
| ------------- | --------- | -------- | -------- | ------- |
| analysis      | 1.00      | 0.20     | 0.33     | 5       |
| hot_take      | 0.76      | 0.65     | 0.70     | 20      |
| reaction      | 0.31      | 0.67     | 0.42     | 6       |
| **macro avg** | **0.69**  | **0.51** | **0.49** | **31**  |

**Fine-tuned DistilBERT**:

| Label         | Precision | Recall   | F1       | Support |
| ------------- | --------- | -------- | -------- | ------- |
| analysis      | 0.00      | 0.00     | 0.00     | 5       |
| hot_take      | 0.65      | 1.00     | 0.78     | 20      |
| reaction      | 0.00      | 0.00     | 0.00     | 6       |
| **macro avg** | **0.22**  | **0.33** | **0.26** | **31**  |

### Confusion Matrix (Fine-Tuned Model)

|                    | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
| ------------------ | ------------------- | ------------------- | ------------------- |
| **True: analysis** | 0                   | 5                   | 0                   |
| **True: hot_take** | 0                   | 20                  | 0                   |
| **True: reaction** | 0                   | 6                   | 0                   |

The fine-tuned model predicted every single test example as `hot_take`. It learned no label boundaries at all — it collapsed into a majority-class predictor. The `analysis` and `reaction` columns are entirely zeros.

### Why the Fine-Tuned Model Failed

The fine-tuned model's 0.645 accuracy looks like it beats the baseline's 0.581, but this is an artifact of the data distribution (hot_take = 64.4% of the dataset), not evidence of learning. The model achieved a macro F1 of 0.26 — far below the baseline's macro F1 of 0.49, and far below my success threshold of 0.65.

The root cause is class imbalance at small scale. With only 205 total examples and `hot_take` making up 64.4% of them, the training set contained roughly 92 hot_take examples, 30 reaction examples, and 21 analysis examples. DistilBERT found it easier to minimize loss by predicting `hot_take` every time than by learning the actual label boundaries — especially since the boundaries between these labels are genuinely subtle (many `analysis` and `reaction` posts share surface-level vocabulary with `hot_take`). This is the classic majority-class collapse problem that the project hints explicitly warned about.

The zero-shot baseline, by contrast, never saw the training data distribution. It classified based purely on the label definitions in the prompt, so it wasn't biased toward any label by frequency. This gave it worse overall accuracy but meaningfully better discrimination — it actually attempted to identify all three classes.

### Wrong Prediction Analysis

I used Claude to scan the 11 misclassified examples for patterns. The AI-surfaced observation was that every single error went in one direction (`analysis` → `hot_take` or `reaction` → `hot_take`), and confidence scores were nearly identical (0.36–0.38) across all predictions, confirming the model was not genuinely discriminating but applying a near-uniform default. I verified this by checking that no test example received a prediction of `analysis` or `reaction` — confirmed via the confusion matrix.

Three specific wrong predictions, analyzed in depth:

**Wrong prediction 1**: _"A bunch of times this series he would use whoever was guarding him as a screen on wemby. really funny and clever"_
True: `analysis`. Predicted: `hot_take` (confidence: 0.36).
This post describes a specific, checkable tactical pattern (using your own defender as a screen on Wembanyama) — it's informal film analysis. The model missed it because it never learned to distinguish `analysis` at all, but even if it had, the casual tone ("really funny and clever") and lack of explicit numbers would make this a hard case. It's the kind of `analysis` that relies on a described mechanism rather than a cited statistic, and the model would need more examples of this subtype to learn the pattern.

**Wrong prediction 2**: _"These comments let you know that most people who watch sports have no idea what they're watching. People are framing it as 'Kawhi disappeared' instead of giving Kerr his props for that amazing defensive game plan..."_
True: `analysis`. Predicted: `hot_take` (confidence: 0.38).
This post explains a specific tactical mechanism (traps forcing Kawhi to fatigue by end of Q3) as evidence for a broader argument. The opening line ("most people have no idea what they're watching") reads like a classic hot_take dismissal, which likely makes this hard for any text classifier — the rhetorical framing says `hot_take` but the content says `analysis`. A model would need to weigh the argumentative structure over the tone, which is a lot to ask of 21 training examples.

**Wrong prediction 3**: _"That looked like a end of a era embrace"_
True: `reaction`. Predicted: `hot_take` (confidence: 0.36).
This is a brief, in-the-moment observation about a specific visual during the game. It contains no generalization, prediction, or judgment beyond the moment. The model's failure here isn't about a subtle boundary — this is a textbook `reaction`, and any model that had genuinely learned what `reaction` means should catch it. The failure is purely due to majority-class collapse, not a difficult edge case.

### Sample Classifications

| Post                                                                                                 | Predicted | Confidence | Correct?            |
| ---------------------------------------------------------------------------------------------------- | --------- | ---------- | ------------------- |
| "The way he plays pointguard is really fun to watch. He plays it so close to the center line..."     | hot_take  | 0.36       | ❌ (true: analysis) |
| "LeBron was never clutch. Anyone who watched the 2011 Finals knows what he is."                      | hot_take  | 0.37       | ✅                  |
| "That looked like a end of a era embrace"                                                            | hot_take  | 0.36       | ❌ (true: reaction) |
| "The threshold for the last time that 'real basketball' was played keeps moving forward every year." | hot_take  | 0.37       | ✅                  |
| "He was standing out of bounds with the game on the line and then got his pocket picked."            | hot_take  | 0.38       | ❌ (true: reaction) |

The one correct prediction — "LeBron was never clutch" — is correctly labeled `hot_take` because it asserts a strong, generalized claim about a player's entire career with no specific evidence (the reference to "the 2011 Finals" is decorative context, not a cited stat). But the prediction is not genuinely "correct" in the sense that the model identified it as a hot take — the model predicted everything as `hot_take`, so this hit is coincidental, not diagnostic.

## Reflection: What the Model Captured vs. What I Intended

I intended the model to learn three distinct discourse modes: evidence-backed reasoning, unsupported assertion, and non-argumentative expression. What it actually learned was a single signal: the prior probability of `hot_take` in the training data.

The gap between intention and outcome has two root causes, both of which I could diagnose only after seeing the results:

First, the label distribution was too skewed for the dataset size. At 64.4% hot_take, the majority class was dominant enough that DistilBERT's loss function found the "always predict hot_take" strategy more efficient than learning boundaries. This wouldn't necessarily be fatal with 2,000 examples (the minority classes would have 300–400 examples each, enough to learn from), but at 205 total examples, the minority classes had too few examples to create a meaningful gradient signal.

Second, the label boundaries I defined are genuinely hard for a small model to learn from text alone. The difference between `analysis` and `hot_take` often comes down to whether a number is "part of the reasoning" or "decorative" — a distinction that requires understanding argumentative structure, not just detecting the presence of numbers. The difference between `reaction` and `hot_take` sometimes comes down to whether a post generalizes beyond a single moment — a distinction that requires understanding temporal scope. These are hard even for humans (see the difficult annotation cases in `planning.md` Section 3), and 21 analysis training examples is nowhere near enough to teach them.

The zero-shot baseline outperformed the fine-tuned model on macro F1 (0.49 vs 0.26) precisely because it bypassed the training data entirely and applied the label definitions directly. This is one of the valid outcomes my `planning.md` success criteria anticipated: "If fine-tuning doesn't clear the zero-shot baseline by a meaningful margin, the honest conclusion is that the labels either aren't learnable from text alone at this dataset size, or the zero-shot model with a well-written prompt is already sufficient."

What would need to change: more data (especially for `analysis` and `reaction`), more balanced distribution (closer to 33/33/33), and possibly class-weighted loss during training to counteract the remaining imbalance.

## Spec Reflection

**One way the spec helped**: The spec's requirement to define success criteria before training (Milestone 2, Section 6) forced me to commit to specific numbers (macro F1 ≥ 0.65, no class below 0.50) and to explicitly state that a fine-tuned model failing to beat the baseline would be a valid finding. When the fine-tuned model collapsed to majority-class prediction, I didn't have to retroactively decide whether this was a failure — I already had a framework that said "this is a reportable outcome, not a project failure." Without that pre-commitment, I might have wasted time trying to fix the model instead of analyzing why it failed, which is where the real learning happened.

**One way the implementation diverged**: The spec suggested 2–4 labels. I started with 4, but during data collection I discovered that `discussion` was naturally too rare in r/nba to sustain as an independent label (under 10% across multiple source types). I merged it into `reaction` mid-project, reducing to 3 labels. This wasn't anticipated in the spec but was the right call — forcing 200+ discussion examples would have required collecting from sources that don't represent how r/nba actually works, defeating the spec's own principle that labels should be "grounded in community norms."

## AI Usage

**Instance 1 — Label stress-testing (before annotation)**:
I gave Claude my four original label definitions and asked it to generate 8 posts designed to sit at the boundary between two labels. Three of the eight exposed real gaps in my decision rules: the original rules treated "event-triggered" as automatically meaning `reaction`, but many hot takes are also event-triggered; and the rules didn't address posts that name a metric without giving its value. I rewrote the edge case rules in `planning.md` Section 3 to fix both gaps before annotating any real data. The AI didn't design the fix — it generated the adversarial test cases, and I diagnosed and corrected the rules myself.

**Instance 2 — Pre-labeling during annotation**:
I pasted batches of 60–70 raw r/nba comments (no labels) to Claude and asked it to suggest a label and reasoning for each, using the decision rules from `planning.md`. Claude suggested labels for all 205 examples. I reviewed every suggestion — several were flagged by Claude itself as "borderline" or "flag for review," and on those I applied my own judgment rather than accepting the suggestion. The final dataset reflects my reviewed labels, not the raw AI suggestions. This workflow is disclosed in `planning.md` Section AI Tool Plan.

**Instance 3 — Error pattern analysis (after evaluation)**:
I pasted the 11 misclassified test examples to Claude and asked it to identify systematic patterns. The AI observed that every error was a one-directional collapse (everything → `hot_take`) with near-identical confidence scores, confirming the majority-class collapse diagnosis. I verified this by checking the confusion matrix (all-zero columns for `analysis` and `reaction`). The AI's observation matched what the confusion matrix already showed — it didn't surface a hidden pattern, but it confirmed the diagnosis efficiently.

## Repository Contents

| File                      | Description                                                             |
| ------------------------- | ----------------------------------------------------------------------- |
| `README.md`               | This evaluation report                                                  |
| `planning.md`             | Design document: label definitions, edge cases, data plan, AI tool plan |
| `takemeter_dataset.csv`   | 205 labeled examples (text, label, notes)                               |
| `evaluation_results.json` | Accuracy comparison (baseline vs fine-tuned)                            |
| `confusion_matrix.png`    | Confusion matrix visualization for fine-tuned model                     |
| `baseline_results.md`     | Detailed baseline performance breakdown                                 |

## Demo Video

[Demo Video](https://www.loom.com/share/0e3a90cd95ca4d64a29e7be7475ab1e2)
