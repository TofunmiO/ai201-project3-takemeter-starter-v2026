# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1.
At least 72.5% overall accuracy on the test set.
<!-- Your criterion. It must name a number. -->


**Why this target:**
I treated the task as moderately difficult: the labels are distinct enough to be learnable, but the boundary between hot_take and reaction is fuzzy enough that a perfect score is unrealistic. A target of 72.5% overall accuracy is therefore a realistic threshold for a useful classifier while still requiring meaningful performance above a naive baseline.


 
## 2.

<!-- Your criterion. -->
Each label should reach at least 0.50 F1 on the held-out set, and the strongest label should not outperform the weakest by more than 0.25 points.

**Why this target:**
My dataset is heavily skewed toward `analysis`, and the smallest labels are only a handful of posts in the test split. A threshold of 0.50 F1 still requires the model to learn the minority classes, but it is realistic given that `hot_take` and `reaction` are harder and smaller than the majority class.

---

## 3.

<!-- Your criterion. -->
No label should make up less than 5% of the data unless it is intentionally rare, and the model should not rely on majority-label guessing.

**Why this target:**
The final dataset is about 67% `analysis`, with `reaction` at 25% and `hot_take` at 9%, so no label exceeds the 70% cap and the model still has to learn the minority classes instead of defaulting to the majority class.

---

## 4.

<!-- Your criterion. -->
Two labelers should agree on at least 75% of posts, or the taxonomy needs revision.

**Why this target:**
The hardest boundary in this project is between `reaction` and `hot_take`, and that boundary is exactly where disagreement is most likely. A target of 75% agreement is high enough that the definitions are usable in practice, but still realistic for a taxonomy that contains fuzzy emotional language and opinion-heavy sports posts.

---

## 5.

<!-- Your criterion. -->
Predictions above 80% confidence should be correct at least 75% of the time, and uncertain cases below 60% confidence should be flagged or excluded.

**Why this target:**
In a small and imbalanced dataset, confidence is a useful safety check. If the model is very confident but wrong, it is overfitting or misreading the boundary between classes; forcing the system to flag uncertain cases makes the classifier safer and more honest than a raw one-size-fits-all prediction.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
