# TakeMeter — Planning Document

## 1. Community

**Community chosen**: r/nba (reddit.com/r/nba)

r/nba is one of the largest sports discussion communities on Reddit, with active game threads, weekly discussion threads, and standalone posts running constantly during the season. It's a good fit for a classification task because discourse quality genuinely varies post to post: the same thread can contain a detailed statistical breakdown, an all-caps emotional reaction to a dunk, a confidently-asserted opinion with no backing, and a genuine open question — often within minutes of each other. The community also already has informal vocabulary for these distinctions ("hot take," "actual analysis," "just here to vibe"), which means the labels reflect a real, recognized split rather than an artificial one imposed from outside.

---

## 2. Labels

Four labels, mutually exclusive, assigned by asking in order: (1) is this purely an in-the-moment reaction with no claim beyond the moment? (2) is this a genuinely open question? (3) does this provide specific, checkable evidence? (4) otherwise, it's an assertion without evidence.

### `analysis`

A post that makes a structured argument backed by specific, checkable evidence (a real statistic with a number attached, a named historical comparison, or a concrete tactical observation) where the evidence is presented as part of the reasoning, not as decoration.

- _Example 1_: "SGA's true shooting percentage in clutch situations this season is 63.2%, which ranks top-3 all-time among players with 200+ clutch possessions. The volume argument doesn't hold up when you look at efficiency under pressure."
- _Example 2_: "Jokić's assist-to-turnover ratio in playoff games against switching defenses is 4.8:1 over the past three seasons. Teams that switch on him actually give up more easy buckets than those that go under screens."

### `hot_take`

A bold, confident opinion or generalization stated without specific, checkable evidence. This includes posts that _reference_ a metric or comparison by name without supplying the actual number, and posts that generalize beyond what a cited event or fact actually supports.

- _Example 1_: "LeBron was never clutch. Anyone who watched the 2011 Finals knows what he is."
- _Example 2_: "The Thunder are a fraud. They beat a weak West all year and nearly choked to Indiana. SGA is not a top-5 player."

### `reaction`

An immediate emotional response to a specific live event, with no claim that extends beyond that moment. The post expresses a feeling — it does not generalize, predict, or conclude anything about a player or team beyond what just happened.

- _Example 1_: "HE CROSSED HIM UP AND THEN DUNKED ON HIM ARE YOU KIDDING ME THIS MAN IS UNREAL"
- _Example 2_: "I cannot believe they blew that lead. 20 points. Gone. I'm done with this team until next season."

### `discussion`

A genuine, open-ended question or prompt inviting community input, where the poster does not signal a preferred answer through loaded language.

- _Example 1_: "Who do you think wins Finals MVP if Haliburton stays healthy through Game 7?"
- _Example 2_: "Best defensive player in the league right now who isn't getting talked about enough?"

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
→ **Decision rule**: If the question's wording implies a preferred answer (loaded phrasing, leading framing), label `hot_take`. If the wording is neutral and doesn't tip toward an answer, label `discussion`. This example → `hot_take`.

During annotation, any post I can't confidently assign using these three rules will be flagged in a `notes` column in the dataset CSV with my reasoning, so the decision is auditable rather than silent.

---

## 4. Data Collection Plan

**Source**: r/nba — pulled from game thread comments, weekly discussion threads, and standalone top posts from the most recent 2–3 months, using the Reddit API (PRAW) or the old.reddit JSON endpoints.

**Target volume**: 250 examples total (to allow for some discarded/unusable posts), split roughly:

- `hot_take`: ~75 (30%)
- `reaction`: ~65 (26%)
- `discussion`: ~55 (22%)
- `analysis`: ~55 (22%)

**Split**: 70% train / 15% validation / 15% test, stratified by label so each split preserves the overall label distribution.

**If a label is underrepresented after 200 examples**: `analysis` is the label most likely to fall short, since detailed statistical posts are rarer than hot takes and reactions in casual threads. If any label falls below ~15% of the dataset after collecting 200 examples, I will targetedly search higher-effort subreddit spaces (e.g., stickied weekly "deep dive" threads, top-of-month posts sorted by upvotes) specifically for that label rather than continuing to sample randomly, since random sampling will keep reproducing the same imbalance.

---

## 5. Evaluation Metrics

**Accuracy** alone is not sufficient here because the label distribution is not perfectly balanced (~22–30% per label) — a model that defaults to predicting the majority class (`hot_take`) could still post a misleadingly high accuracy without actually distinguishing the classes.

Metrics I will report:

- **Overall accuracy** — baseline sanity check, easy to compare against the zero-shot LLM.
- **Per-class precision, recall, and F1** — this is the metric that actually matters for this task. Recall tells me whether the model is missing real examples of a label (e.g., failing to catch `analysis` posts), while precision tells me whether it's over-predicting a label. Because the four labels are not equally easy to distinguish (I expect `hot_take` vs `reaction` and `hot_take` vs `analysis` to be the hardest boundaries based on the edge cases above), per-class F1 will reveal which specific boundary the model is failing to learn — overall accuracy would hide this.
- **Confusion matrix** — to see exactly which label pairs get confused with each other, which directly tests whether the model learned the boundaries I defined in section 3, or learned a cruder approximation (e.g., confusing `analysis` and `hot_take` whenever a post contains a number, regardless of whether the number is contextualized).

---

## 6. Definition of Success

**Genuinely useful threshold**: Macro-averaged F1 ≥ 0.65 across all four classes, with no single class F1 below 0.50. A model that does well on three labels but collapses on one (e.g., never correctly identifies `analysis`) is not useful for a real community tool, since it would mislabel a quarter of all posts in a way that's systematic rather than random.

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

I will **not** use an LLM to pre-label the dataset. Given the dataset size (250 examples) and the fact that the label boundaries are still fairly fresh and required active correction during stress-testing, I judged that pre-labeling risks anchoring my own judgment to a model's (possibly wrong) labels rather than applying the decision rules independently. All 250 examples will be hand-labeled by me using the rules in section 3.

### Failure analysis

After evaluating both the fine-tuned model and the zero-shot baseline, I will compile the full list of test-set examples each model got wrong and give that list to an AI tool, asking it to identify systematic patterns (e.g., "the model confuses `hot_take` and `analysis` specifically when the post contains a number" or "short posts under N words are misclassified more often"). I will then manually verify each proposed pattern against the actual misclassified examples before including it in the evaluation report — the AI's pattern suggestions are a starting hypothesis, not a conclusion I'll report without checking the underlying examples myself.
