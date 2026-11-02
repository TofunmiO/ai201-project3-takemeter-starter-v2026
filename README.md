# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

I picked a public sports subreddit where people post game updates, clips, player reactions, and quick takes about major events. The dataset includes both substantive discussion and much weaker posts that are just emotional reactions or unsupported opinions.

The posts I wanted the model to learn were the ones that argued from a fact or explained why a result mattered, rather than simply sharing a headline, sport news/summary or a reaction. I also noticed a split between short clip posts and longer posts that offered context, interpretation, or a reasoned claim.

Finally, I wanted the classifier to separate fact-backed judgment from raw feeling: a post that points to a stat, quote, or reported development is different from one that is mainly shock, frustration, or hype.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** A post that makes a claim and supports it with a specific, checkable fact such as a stat, date, quote, or concrete game detail.

**Example 1:**
> This is why NHL goalies are covered in protective gear. <https://www.reddit.com/r/sports/comments/427uvb/this_is_why_nhl_goalies_are_covered_in_protective/>

**Example 2:**
> This is why hockey teams never wear white pants..... <https://www.reddit.com/r/sports/comments/3vl9ny/this_is_why_hockey_teams_never_wear_white_pants/>

### `hot_take`

**Definition:** A confident claim that sounds like a judgment but offers no evidence, no supporting fact, and no argument beyond the opinion itself.

**Example 1:**
> Dönermann with one of the greatest saves of all time. <https://www.reddit.com/r/sports/comments/1wm0ms8/d%C3%B6nermann_with_one_of_the_greatest_saves_of_all/>

**Example 2:**
> Dele Alli is overrated. <https://www.reddit.com/r/sports/comments/6hf87w/dele_alli_is_overrated/>

### `reaction`

**Definition:** An immediate emotional response to something that just happened—shock, frustration, excitement, or disbelief—where the main point is the feeling, not a fact-backed explanation or argument.

**Example 1:**
> This high school kid's athleticism is unbelievable <https://www.reddit.com/r/sports/comments/5brk1l/this_high_school_kids_athleticism_is_unbelievable/>

**Example 2:**
> Tiger Woods' unbelievable birdie from the bunker on 18 (GIF) <https://www.reddit.com/r/sports/comments/wvwiz/tiger_woods_unbelievable_birdie_from_the_bunker/>

### The hardest boundary

**Which two labels:** `hot_take` and `reaction`

**The decision rule I used every time:** If the post is mainly an immediate emotional burst—shock, anger, excitement, or disbelief—it is `reaction`. If the post is mainly a confident judgment with no evidence or supporting reason, even if there is a clip attached, it is `hot_take`. A fact-backed claim with an argument stays in `analysis`.

**Borderline example:** "Tiger Woods' unbelievable birdie from the bunker on 18 (GIF)" <https://www.reddit.com/r/sports/comments/wvwiz/tiger_woods_unbelievable_birdie_from_the_bunker/> sits right on the edge of reaction and hot take: there is a clip and a strong reaction, but the main point is the immediate emotion and excitement, so I kept it in `reaction`.

**AI stress test:** I tested the definitions by asking an AI model to write eight borderline posts near the `hot_take`/`reaction` boundary. The outputs were cleanly classifiable once the rule was explicit: feeling-first posts stayed in `reaction`, while unsupported but confident judgments stayed in `hot_take`, and fact-backed arguments stayed in `analysis`.

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** r/sports

**How I labelled them:** I started with a cold pass on the first 20 posts, then used a quick AI pre-label pass on the remaining posts and corrected each row by hand before locking the dataset. The first 20 are marked `cold`; the remaining 200 are marked `pre-labelled, corrected`.

**Counts per label:**

| Label | Count | Share |
|---|---:|---:|
| `analysis` | 147 | 66.8% |
| `hot_take` | 19 | 8.6% |
| `reaction` | 54 | 24.5% |
| **Total** | **220** | **100%** |

**Three hard cases**

**1.**
> *The post:* "Premier League football shirts made by workers on £1 an hour"
>
> *Could have been:* `hot_take` or `analysis`
>
> *I chose `analysis`, because:* it is framed as a labor-reporting claim with a concrete fact about wage conditions, even though the title is sharp and opinionated.

**2.**
> *The post:* "Seriously - why do fans toss their drinks like this at soccer games?"
>
> *Could have been:* `reaction` or `analysis`
>
> *I chose `reaction`, because:* the post starts from immediate frustration and the emotional tone is the main point, even though it is framed as a question.

**3.**
> *The post:* "LIV Golf CEO says tour will continue, 'wouldn't bet against us'"
>
> *Could have been:* `hot_take` or `analysis`
>
> *I chose `hot_take`, because:* it is a confident unsupported judgment wrapped in a business quote, not a fact-backed explanation.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate `2e-5`, batch size `16`, seed `42`.

**Anything I changed from the defaults, and why:** I used the notebook defaults from the starter setup rather than tuning the model aggressively. The small dataset and the strong class imbalance made it more important to keep a stable, low-risk setup than to overfit to one particular seed. I also kept the split stratified and locked the test set before evaluation so the results were comparable and not contaminated by leakage.

**Split sizes:** The dataset was split at 70/15/15, stratified by label. The final held-out test split contained 33 posts: `analysis` = 22, `hot_take` = 3, and `reaction` = 8. The `hot_take` count is tiny, so its F1 is noisy across seeds and needs a looser expectation than the majority class.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* I asked an AI model to generate and sort borderline sports posts around the `hot_take` / `reaction` boundary, and to explain why a post belonged in each label.
- *What came back:* It produced plausible examples and a quick rule-of-thumb, but it also blurred the boundary by treating some emotional reactions as unsupported judgments.
- *What I changed:* I tightened the rule so that `reaction` is for feeling-first posts and `hot_take` is for confident unsupported judgment. I then used that rule to recheck the ambiguous examples in the dataset and in the model-generated stress-test cases.

**Moment 2**

- *What I asked for:* I asked the model to pre-label the remaining posts in the dataset and flag likely boundary cases before I manually reviewed them.
- *What came back:* It produced a fast draft set of labels, but several posts were mislabeled as `analysis` simply because they sounded like news headlines or had a quote attached.
- *What I changed:* I read and corrected every single row in that batch by hand. The final labels are my corrected labels, and the notes record the workflow: the first 20 were `cold`, and the remaining 200 were `pre-labelled, corrected`.

**Pre-labelling disclosure:** I used AI as a pre-labeling aid for the remaining batch of posts, then reviewed and corrected every label manually before finalizing the dataset. The final CSV is the corrected version, not the raw model output.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.424 | 0.667 | +0.242 |
| Macro F1 | 0.283 | 0.267 | -0.016 |
| F1 — `analysis` | 0.541 | 0.800 | +0.259 |
| F1 — `hot_take` | 0.000 | 0.000 | +0.000 |
| F1 — `reaction` | 0.308 | 0.000 | -0.308 |

**Definitions check:** With the full definitions, baseline accuracy was 0.424;
with bare label names, it was 0.152. The definitions improved accuracy by
0.272 (27.2 percentage points) on the same 33 held-out posts.

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->
I think my trained model will do better at finding hot_take and reaction posts than the baseline.
**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->
The trained model improved overall accuracy from 0.424 to 0.667 and analysis F1 from 0.541 to 0.800, tied on hot_take at 0.000, and performed worse on reaction (0.000 vs. 0.308). So training improved some measures but did not improve every label.


---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | At least 0.725 | 0.667 | 0.667 | 0.667 | MISSED |
| 2. Per-label F1 and balance | Every label >= 0.500 F1; strongest–weakest gap <= 0.250 | analysis 0.800; hot_take 0.000; reaction 0.000 | analysis 0.800; hot_take 0.000; reaction 0.000 | analysis 0.800; hot_take 0.000; reaction 0.000 | MISSED |
| 3. Class balance / majority guessing | Each label >= 5% of data; model does not guess only the majority label | Dataset: analysis 66.8%, hot_take 8.6%, reaction 24.5%; predicted only analysis | Same dataset; predicted only analysis | Same dataset; predicted only analysis | MISSED: model guessed only analysis |
| 4. Labeler agreement | At least 75% agreement | 28/30 (single check) | N/A | N/A | MET |
| 5. Confidence | >80%-confidence predictions >=75% correct; <60%-confidence predictions flagged/excluded | Most-confident-third accuracy 0.818; no threshold flags recorded | Most-confident-third accuracy 0.636; no threshold flags recorded | Most-confident-third accuracy 0.727; no threshold flags recorded | MISSED: no low-confidence flag/exclusion; exact thresholds not measured |

### Three-seed spread (Section 6)

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.667 | 0.667 | 0.667 | 0.000 |
| Macro F1 | 0.267 | 0.267 | 0.267 | 0.000 |
| F1 — `analysis` | 0.800 | 0.800 | 0.800 | 0.000 |
| F1 — `hot_take` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `reaction` | 0.000 | 0.000 | 0.000 | 0.000 |

Accuracy moved 0.000 across seeds. Stable enough to trust. The stored
confidence summary reports accuracy for the most- and least-confident thirds,
not the exact >80% and <60% thresholds in criterion 5; those thresholds still
need to be measured.

**Device:** Apple Silicon GPU (MPS), PyTorch 2.14.1.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** | 22 | 0 | 0 |
| **hot_take** | 3 | 0 | 0 |
| **reaction** | 8 | 0 | 0 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->
The largest confusion was 8 real `reaction` posts predicted as `analysis`,
while 0 real `analysis` posts were predicted as `reaction`; the model leaned
toward `analysis` on this pair. The agreement report can help investigate
whether inconsistent labeling contributed to this pattern.

---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | At least 0.725 on every seed | MISSED | Accuracy was 0.667 for all three seeds. |
| 2 | Per-label F1 and balance | Every label >= 0.500 F1; strongest–weakest gap <= 0.250 | MISSED | All seeds had analysis F1 0.800 and hot_take/reaction F1 0.000; the gap was 0.800. |
| 3 | Class balance / majority guessing | Every label >= 5% of data; model does not guess only the majority label | MISSED | Data shares met the 5% floor, but the confusion matrix shows every test post was predicted as analysis. |
| 4 | Labeler agreement | At least 75% agreement | MET | Agreement check matched 28 of 30 staff-labeled posts (93%). |
| 5 | Confidence | >80%-confidence predictions >=75% correct; <60%-confidence predictions flagged/excluded | MISSED | The run did not flag or exclude low-confidence predictions. The saved summary measures confidence thirds, not the exact 80%/60% thresholds. |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

Criterion 4 is a single staff-labeling comparison, not a model score that
changes by seed; 28/30 (93%) exceeds the 75% target.

- **Criterion 1 — accuracy below target.** Accuracy was 0.667 in every seed,
  below 0.725. The confusion matrix shows the model predicted only `analysis`,
  which accounts for 22 of 33 test posts; the observed majority-only behavior
  explains the 0.667 score. The three seeds show this was consistent across
  these splits, but do not by themselves identify why the model collapsed.

- **Criterion 2 — minority-label F1 and balance gap missed.** In every seed,
  `hot_take` and `reaction` F1 were 0.000 while `analysis` F1 was 0.800, a
  0.800-point gap. The matrix shows all 3 `hot_take` and all 8 `reaction` test
  posts were predicted as `analysis`; the small `hot_take` test support (3)
  makes its estimate especially limited. The staff-labeling exercise found
  two rule-application inconsistencies, and Unit 5 feedback also flagged some
  `analysis` examples as not matching the written fact-backed rule. These are
  evidence that labeling consistency deserves review, not proof that it caused
  the model's zero scores; the staff exercise used a separate set and taxonomy.

- **Criterion 3 — the dataset balance floor was met, but the model guessed only
  the majority label.** Dataset shares were 66.8% `analysis`, 8.6% `hot_take`,
  and 24.5% `reaction`, so each exceeded 5%. The criterion was still missed
  because the confusion matrix shows every prediction was `analysis`. This
  directly establishes majority-label guessing, though the underlying cause
  needs further investigation.
  
- **Criterion 5 — confidence requirement not demonstrated.** The results
  summarize accuracy for the most- and least-confident thirds, rather than
  accuracy among predictions above 80% confidence or behavior below 60%.
  No low-confidence predictions were flagged or excluded. Thus the available
  evidence does not establish the high-confidence target, and the low-
  confidence safeguard was not met.

### Improvement for the after run

I reviewed the `analysis` vs. `hot_take` boundary and changed one row in
`labels.csv`: “Soccer now tops hockey for young Canadians” changed from
`analysis` to `hot_take` because the post states a confident comparison but
does not provide a supporting fact. No other labels or training settings were
changed. I set Section 6 to `RUN_LABEL = "after"` and reran all three seeds.

## Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | At least 0.725 | 0.667 | 0.667 | 0.667 | MISSED |
| 2. Per-label F1 and balance | Every label >= 0.500 F1; strongest–weakest gap <= 0.250 | analysis 0.800; hot_take 0.000; reaction 0.000 | analysis 0.800; hot_take 0.000; reaction 0.000 | analysis 0.800; hot_take 0.000; reaction 0.000 | MISSED |
| 3. Class balance / majority guessing | Each label >= 5% of data; model does not guess only the majority label | Dataset: analysis 66.4%, hot_take 9.1%, reaction 24.5%; predicted only analysis | Same dataset; predicted only analysis | Same dataset; predicted only analysis | MISSED: model guessed only analysis |
| 4. Labeler agreement | At least 75% agreement | 28/30* | 28/30* | 28/30* | MET |
| 5. Confidence | >80%-confidence predictions >=75% correct; <60%-confidence predictions flagged/excluded | Most-confident-third accuracy 0.818; no threshold flags recorded | Most-confident-third accuracy 0.636; no threshold flags recorded | Most-confident-third accuracy 0.727; no threshold flags recorded | MISSED: no low-confidence flag/exclusion; exact thresholds not measured |

*The agreement exercise is one separate staff-labeling check, not a measure
that changes with the model seed; the same result is shown for each seed.

The relabeling did **not** improve the measured model scores: accuracy remained
0.667, macro F1 0.267, and per-label F1 stayed at 0.800 for `analysis` and
0.000 for both `hot_take` and `reaction` in all three seeds. The model still
predicted only `analysis`. This one-row relabel was a controlled, documented
test of the boundary, but it was not enough to change the observed results.


---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 28 / 30 = 93%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:* “Three of us ran the same session on the same day, different shoes, within 10 seconds of each other over 5 miles. I'm not saying shoes do nothing, but whatever they do was smaller than the noise between three people.”
>
> *Staff said / I said:* `analysis` / `hot_take`
>
> *My call, and why:* Staff is right: the post uses a statistical comparison to support its claim that the shoe effect was smaller than the variation between runners.
>
> *Which pile:* (a) The rule covered the fact-backed comparison; I applied it loosely.

**2.**
> *The post:* “Four minutes off. FOUR. I've been chasing that time for two years and I'm sat on the kerb outside the finish just grinning at nothing.”
>
> *Staff said / I said:* `reaction` / `hot_take`
>
> *My call, and why:* Staff is right: the post describes what happened to the writer and their emotional response, rather than making a general claim about the world.
>
> *Which pile:* (a) The reaction rule covered it; I applied it loosely.

**What the pattern in my disagreements tells me:**
Both disagreements were covered by the written rules, and I applied those rules
loosely. In one case I overlooked that the comparison statistics supported the
claim; in the other, I treated a personal emotional response as a general
opinion.



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
