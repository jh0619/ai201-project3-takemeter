# TakeMeter — Planning Document

## 1. Community

**Community chosen**: r/nba (reddit.com/r/nba)

r/nba is one of the largest sports discussion communities on Reddit, with active game threads, weekly discussion threads, and standalone posts running constantly during the season. It's a good fit for a classification task because discourse quality genuinely varies post to post: the same thread can contain a detailed statistical breakdown, an all-caps emotional reaction to a dunk, a confidently-asserted opinion with no backing, and a genuine open question — often within minutes of each other. The community also already has informal vocabulary for these distinctions ("hot take," "actual analysis," "just here to vibe"), which means the labels reflect a real, recognized split rather than an artificial one imposed from outside.

---

## 2. Labels

**Note on taxonomy revision**: The original design (Milestones 1–2) used four labels, including a separate `discussion` category for open-ended questions. During data collection, `discussion` consistently came in under 10% of collected examples across multiple source types (game threads, celebration threads), well short of the ~20% target — r/nba simply doesn't generate much genuinely neutral, open-ended text relative to assertive commentary. Rather than force collection toward an artificially inflated `discussion` bucket, I merged it into `reaction`, since both categories share the same underlying property: neither makes an evaluable claim. This is a recorded design decision, not a silent fix — the original four-label boundary rules (including edge case 3 below) are kept because the underlying judgment call (is this post asserting something or not) still applies under the three-label system.

Three labels, mutually exclusive, assigned by asking in order: (1) does this provide specific, checkable evidence as part of a structured argument? (2) does this assert a bold claim without checkable evidence? (3) otherwise — no evaluable claim is being made at all (pure feeling, or a genuinely open question) — `reaction`.

### `analysis`

A post that makes a structured argument backed by specific, checkable evidence (a real statistic with a number attached, a named historical comparison, or a concrete tactical observation) where the evidence is presented as part of the reasoning, not as decoration.

- _Example 1_: "SGA's true shooting percentage in clutch situations this season is 63.2%, which ranks top-3 all-time among players with 200+ clutch possessions. The volume argument doesn't hold up when you look at efficiency under pressure."
- _Example 2_: "Jokić's assist-to-turnover ratio in playoff games against switching defenses is 4.8:1 over the past three seasons. Teams that switch on him actually give up more easy buckets than those that go under screens."

### `hot_take`

A bold, confident opinion or generalization stated without specific, checkable evidence. This includes posts that _reference_ a metric or comparison by name without supplying the actual number, and posts that generalize beyond what a cited event or fact actually supports.

- _Example 1_: "LeBron was never clutch. Anyone who watched the 2011 Finals knows what he is."
- _Example 2_: "The Thunder are a fraud. They beat a weak West all year and nearly choked to Indiana. SGA is not a top-5 player."

### `reaction`

A post that makes no evaluable claim at all. This covers two cases that share the same underlying property — nothing here can be judged true, false, well-supported, or unsupported: (a) an immediate emotional response to a specific live event, with no claim that extends beyond that moment, and (b) a genuine, open-ended question or prompt inviting community input, where the poster does not signal a preferred answer through loaded language.

- _Example 1 (emotional)_: "HE CROSSED HIM UP AND THEN DUNKED ON HIM ARE YOU KIDDING ME THIS MAN IS UNREAL"
- _Example 2 (emotional)_: "I cannot believe they blew that lead. 20 points. Gone. I'm done with this team until next season."
- _Example 3 (open question)_: "Who do you think wins Finals MVP if Haliburton stays healthy through Game 7?"
- _Example 4 (open question)_: "Best defensive player in the league right now who isn't getting talked about enough?"

---

## 3. Hard Edge Cases

**Edge case type 1 — event-triggered post that generalizes beyond the moment.**
Example: "this is the most disrespectful no-call I've ever seen, refs have been terrible all series, OKC gets every call."
This is triggered by a specific play (like a `reaction`), but "terrible all series" and "gets every call" are claims that extend beyond that single moment.
→ **Decision rule**: If the post stays confined to expressing a feeling about the single event, label `reaction`. The moment a post generalizes into a claim about a pattern, trend, or broader truth — even without numbers — label `hot_take`. This example → `hot_take`.

**Edge case type 2 — references a real metric without providing the number.**
Example: "OK but has anyone actually checked Luka's defensive box plus-minus since the trade? I feel like it's gotten worse but I don't have the numbers in front of me."
This names a legitimate, checkable statistic, which feels like `analysis`, but no actual value is given — the claim is asserted, not shown.
→ **Decision rule**: Naming a metric is not the same as providing evidence. If no specific number, named comparison, or concrete fact is actually stated, label `hot_take` regardless of how technical the post sounds. This example → `hot_take`.

**Edge case type 3 — a question that signals its own answer.**
Example: "Is SGA actually better than Curry or are we all just caught up in recency bias?"
Phrased as a question, but "recency bias" signals the poster's skepticism toward the popular answer.
→ **Decision rule**: If the question's wording implies a preferred answer (loaded phrasing, leading framing), label `hot_take` — the question is functioning as an assertion in disguise. If the wording is neutral and doesn't tip toward an answer, label `reaction` (under the merged taxonomy, neutral open questions fall under `reaction` since they make no evaluable claim). This example → `hot_take`.

During annotation, any post I can't confidently assign using these three rules will be flagged in a `notes` column in the dataset CSV with my reasoning, so the decision is auditable rather than silent.

### Difficult examples encountered during actual annotation

**Example 1**: _"I don't think this is actually the answer. The amount of fouls per game is generally pretty constant (it's actually kinda a low average right now), regardless of flopping... I think it's purely an issue of refs calling the wrong fouls."_
This references a real, checkable trend (foul-per-game rate being low and roughly constant) and uses it to build an argument — but no exact number is given, just a vague characterization ("kinda a low average"). I labeled this `analysis` because the claim is built on a specific, falsifiable premise (the trend exists and is checkable) rather than a bare assertion, even though the precision is weaker than my cleanest analysis examples. This is the softest edge of the `analysis` boundary in the dataset.

**Example 2**: _"He's good, BUT are you better than him?"_
A short, joking rhetorical question with no real argument and no specific emotional trigger from a live event. It doesn't cleanly fit either remaining definition of `reaction` (not a feeling about a just-happened moment, not a genuinely open question either — it's rhetorical, with an obvious implied answer). I kept it as `reaction` on the reasoning that a rhetorical jab with no checkable claim still fails the "evaluable claim" test that defines `hot_take` and `analysis`, even if it doesn't fit the _prototypical_ examples of `reaction` cleanly. This case exposed that my three-label decision tree handles confident assertions and genuine reactions well, but has a gap for short, low-effort rhetorical asides that are technically questions but functionally jokes.

**Example 3**: _"Do the warriors always set this many moving screens for Steph? If so, then his mobility to move around the court is a little less impressive knowing GS gets away with this shit."_
Phrased partly as a question ("Do the warriors always...") but the second half commits to an evaluative conclusion ("a little less impressive," "gets away with this shit"). I labeled this `reaction` initially but flagged it, because the loaded framing arguably makes the "question" rhetorical in the same way as edge case type 3 — meaning a stricter reading would push this to `hot_take`. I kept `reaction` because the core claim ("moving screens are being missed") is presented conditionally ("if so") rather than asserted outright, but I recognize a different annotator could reasonably disagree and call this `hot_take`. This is the kind of disagreement an inter-annotator reliability check (stretch feature) would be useful for surfacing systematically.

---

## 4. Data Collection Plan

**Source**: r/nba — pulled from game thread comments, weekly discussion threads, and standalone top posts from the most recent 2–3 months, using manual collection (copy-paste from reddit.com) due to current restrictions on new Reddit API script app approval.

**Target volume**: 250 examples total (to allow for some discarded/unusable posts), split roughly under the revised three-label taxonomy:

- `hot_take`: ~45% (this is consistently the largest natural category in r/nba discourse — confirmed during collection)
- `reaction`: ~35% (combines emotional reactions and open questions)
- `analysis`: ~20%

**Split**: 70% train / 15% validation / 15% test, stratified by label so each split preserves the overall label distribution.

**If a label is underrepresented after 200 examples**: `analysis` is the label most likely to fall short, since detailed statistical/tactical posts are rarer than hot takes and reactions in casual threads. If `analysis` falls below ~15% of the dataset after collecting 200 examples, I will targetedly search higher-effort subreddit spaces (e.g., stickied weekly "deep dive" threads, top-of-month posts sorted by upvotes) specifically for that label rather than continuing to sample randomly, since random sampling will keep reproducing the same imbalance. (Note: this is what originally happened with the now-removed `discussion` label — see Section 2 for that design revision.)

---

## 5. Evaluation Metrics

**Accuracy** alone is not sufficient here because the label distribution is not perfectly balanced (~20–45% per label) — a model that defaults to predicting the majority class (`hot_take`) could still post a misleadingly high accuracy without actually distinguishing the classes.

Metrics I will report:

- **Overall accuracy** — baseline sanity check, easy to compare against the zero-shot LLM.
- **Per-class precision, recall, and F1** — this is the metric that actually matters for this task. Recall tells me whether the model is missing real examples of a label (e.g., failing to catch `analysis` posts), while precision tells me whether it's over-predicting a label. Because the three labels are not equally easy to distinguish (I expect `hot_take` vs `reaction` and `hot_take` vs `analysis` to be the hardest boundaries based on the edge cases above), per-class F1 will reveal which specific boundary the model is failing to learn — overall accuracy would hide this.
- **Confusion matrix** — to see exactly which label pairs get confused with each other, which directly tests whether the model learned the boundaries I defined in section 3, or learned a cruder approximation (e.g., confusing `analysis` and `hot_take` whenever a post contains a number, regardless of whether the number is contextualized).

---

## 6. Definition of Success

**Genuinely useful threshold**: Macro-averaged F1 ≥ 0.65 across all three classes, with no single class F1 below 0.50. A model that does well on two labels but collapses on one (e.g., never correctly identifies `analysis`) is not useful for a real community tool, since it would mislabel a meaningful fraction of all posts in a way that's systematic rather than random.

**"Good enough" for deployment in a real community tool**: Macro F1 ≥ 0.70, with the fine-tuned model outperforming the zero-shot Groq baseline by at least 5 points of macro F1. If fine-tuning doesn't clear the zero-shot baseline by a meaningful margin, the honest conclusion is that the labels either aren't learnable from text alone at this dataset size, or the zero-shot model with a well-written prompt is already sufficient — both are valid, reportable findings, not failures of the project.

These thresholds are specific enough to be checked objectively at the end: I will know exactly which number to compute and what would count as clearing or missing each bar.

---

## AI Tool Plan

### Label stress-testing

I generated 8 posts intentionally designed to sit at the boundary between two labels, using the label definitions and edge-case rules from the first draft of this document. Three of the eight (originally about event-triggered generalizations, metric-name-dropping without numbers, and stat-cherry-picking) exposed gaps in the original rules:

- The original rules treated "event-triggered" as sufficient for `reaction`, but several hot takes are also event-triggered — the real distinguishing factor is whether the post generalizes beyond the triggering moment.
- The original rules didn't address posts that name a real metric (e.g., "defensive box plus-minus") without giving an actual value — these read as analytical but provide no checkable evidence.

Both gaps were fixed in the edge case rules in section 3 above before any annotation began.

### Annotation assistance

I used Claude as a pre-labeling assistant: I pasted raw, unlabeled post/comment text collected from r/nba, and Claude suggested a label and reasoning for each example based on the decision rules in Section 3. I then reviewed every suggestion individually rather than batch-accepting them — several suggestions were flagged as genuinely ambiguous (e.g., posts citing real facts in a casual, non-argumentative tone) and required my own judgment call rather than accepting the suggested label outright. This is disclosed here per the project's AI usage requirements; final labels in `takemeter_dataset.csv` reflect my review and correction, not the raw suggestions.

### Failure analysis

After evaluating both the fine-tuned model and the zero-shot baseline, I will compile the full list of test-set examples each model got wrong and give that list to an AI tool, asking it to identify systematic patterns (e.g., "the model confuses `hot_take` and `analysis` specifically when the post contains a number" or "short posts under N words are misclassified more often"). I will then manually verify each proposed pattern against the actual misclassified examples before including it in the evaluation report — the AI's pattern suggestions are a starting hypothesis, not a conclusion I'll report without checking the underlying examples myself.
