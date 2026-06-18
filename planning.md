# TakeMeter — Planning Document

## Milestone 1: Community and Label Taxonomy

### Community

**r/nba** — one of the largest NBA discussion communities on Reddit. Members post reactions, analysis, hot takes, and questions about games and players constantly. The community itself uses terms like "hot take" and "actual analysis" informally, making discourse quality a native concept here.

---

### Label Taxonomy

#### `analysis`

A post that makes a structured argument backed by specific, verifiable evidence — statistics, historical comparison, or tactical observation. The reasoning is traceable even if you disagree with the conclusion.

- _Example 1_: "SGA's true shooting percentage in clutch situations this season is 63.2%, which ranks top-3 all-time among players with 200+ clutch possessions. The volume argument doesn't hold up when you look at efficiency under pressure."
- _Example 2_: "Jokić's assist-to-turnover ratio in playoff games against switching defenses is 4.8:1 over the past three seasons. Teams that switch on him actually give up more easy buckets than those that go under screens."
- _Uncertain case_: "LeBron is overrated — his playoff win rate against top-seeded opponents is below .500." (one stat + accusatory framing — see edge case 1 below)

#### `hot_take`

A bold, confident opinion stated without traceable evidence. The post asserts rather than argues. A stat may appear but is decorative or cherry-picked rather than part of a reasoned argument.

- _Example 1_: "LeBron was never clutch. Anyone who watched the 2011 Finals knows what he is."
- _Example 2_: "The Thunder are a fraud. They beat a weak West all year and nearly choked to Indiana. SGA is not a top-5 player."
- _Uncertain case_: "Is SGA actually better than Curry or are we all just caught up in recency bias?" (question phrasing, but signals a preferred answer — see edge case 3 below)

#### `reaction`

An immediate emotional response to a specific live event. Little to no argument — the post is expressing a feeling about something that just happened.

- _Example 1_: "HE CROSSED HIM UP AND THEN DUNKED ON HIM ARE YOU KIDDING ME THIS MAN IS UNREAL"
- _Example 2_: "I cannot believe they blew that lead. 20 points. Gone. I'm done with this team until next season."
- _Uncertain case_: "LMAO Thunder blew a 20-point lead. Classic OKC. Their defense just disappears in Q4." (event-triggered + vague claim — see edge case 2 below)

#### `discussion`

A genuine question or open-ended prompt inviting others to share views. The poster makes no strong personal claim — they're seeking community debate or collective knowledge.

- _Example 1_: "Who do you think wins Finals MVP if Haliburton stays healthy through Game 7?"
- _Example 2_: "Best defensive player in the league right now who isn't getting talked about enough?"
- _Uncertain case_: "Is SGA actually better than Curry or are we all just caught up in recency bias?" (see edge case 3 below)

---

### Why These Distinctions Matter

These four types map directly to how r/nba members themselves evaluate discourse quality. Veteran members upvote detailed film breakdowns and call out baseless hot takes. The labels aren't external — they reflect the community's own vocabulary.

---

### Edge Cases and Decision Rules

**Edge case 1 — stat + strong opinion**: "LeBron is overrated — his playoff win rate against top-seeded opponents is below .500."
Could be `analysis` (cites a stat) or `hot_take` (accusatory framing, one isolated number).
→ Decision rule: If the evidence would support the claim even after removing the opinion framing, label `analysis`. If the stat is isolated and not contextualized as part of a logical argument, label `hot_take`. This example → `hot_take`.

**Edge case 2 — reaction with vague claim**: "LMAO Thunder blew a 20-point lead. Classic OKC. Their defense just disappears in Q4."
Could be `reaction` (event-triggered, emotional) or `hot_take` (makes a claim about Q4 defense).
→ Decision rule: If the reasoning is a vague generalization attached to an emotional outburst, label `reaction`. If the post contains a specific, citable claim, label `hot_take`. This example → `reaction`.

**Edge case 3 — question hiding a take**: "Is SGA actually better than Curry or are we all just caught up in recency bias?"
Could be `discussion` (question form) or `hot_take` (implies a preferred answer).
→ Decision rule: If the question clearly signals a preferred answer (here: skepticism via "recency bias"), label `hot_take`. If genuinely open, label `discussion`. This example → `hot_take`.
