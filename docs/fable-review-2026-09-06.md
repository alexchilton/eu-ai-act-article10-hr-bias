# Fable review, 2026-09-06

Adversarial review of the repo by Fable 5.1, requested by Alex. Reproduced as
received. Findings are ordered most serious first. Nothing here has been fixed
yet unless a later commit says so.

---

## 1. Notebook 2's accuracy claim is an in-sample artefact

`notebooks/02_label_choice_and_selective_labels.ipynb` cell 5 trains A and C on
all 20,000 rows and scores them on the same rows. C's 0.707 is above the Bayes
ceiling: the true `risk` score gets AUC 0.668 on a fresh draw from the same
generator. Refit on the notebook's exact data, scored on a fresh 20,000:
A 0.639, B 0.655, C 0.660. On a half split A beats B (0.635 vs 0.625). The
ordering survives but the gap is 0.02, not 0.07, and "no correction technique
recovers as much as choosing the right label" is asserted with no correction
technique run. README row 02 headlines the inflated numbers. The DIR result
(0.31 vs ~0.72) is robust across 8 seeds. Confidence: high.

## 2. The root notebook's "correction" creates the violation its own checklist then fails

`eu_ai_act_article10_hr_compliance.ipynb` cell 13 multiplies the *observed
label* by a weight, so nobody un-hired can become hired: 0 gained, 29 lost.
Black goes from DIR 0.97 to 0.78, 46-60 (a penalised group) becomes the
reference group. Cell 25's `B1 FAIL` and `article10_compliance_report.html`
cite `ethnicity:Black (DIR=0.78)` - manufactured by the step called
"correction". A hostile reader will find this in five minutes. Confidence: high.

## 3. Notebook 1's headline is baked into the generator, and notebook 7 contradicts it

In `01_proxy_detection.ipynb` cell 3, `school` is drawn independently of gender
(zero signal) and `postcode_income_z` shifts by 0.23 SD, while `part_time`
(0.34 vs 0.09), `occupation` (0.55 vs 0.22) and `career_gap` (x1.9) carry the
load. "Stripping the named proxies is close to free of effect" (0.834 to 0.833)
is therefore a design choice, stable across 8 seeds. On real data the same test
says the opposite: Adult drops 0.938 to 0.878 on one removal and to 0.644 after
six. README row 01 headlines the synthetic number. Confidence: high.

## 4. Notebook 5's INLP line is numerically degenerate

`X` has 4 columns; each INLP step removes one rank. Seed 5 stops after 3
projections (rank 1) and reports "4 iterations", tree AUC 0.667. Seeds 0-4 do 4
projections, `P` becomes all-zero to 1e-11, and the tree still reads 0.73-0.81
from float residue. The 0.667 in the README is noise from a toy that INLP simply
deletes.

Also: a *linear* probe on `[b1**2, b2**2, b1*b2]` gets 0.83, so "no direction to
remove, nothing for the method to grip" is basis-dependent, which the INLP paper
itself notes. "Real data looks like this" is asserted; in notebook 1's own
generator `career_gap` is a scale multiplier, which a linear probe sees.
Confidence: high.

## 5. Legal framing errors (root notebook)

- Cell 15 quotes Article 10(5) from the **2021 Commission proposal** ("bias
  monitoring, detection and correction"), not Regulation 2024/1689, which says
  "bias detection and correction", adds "exceptionally", and lists six mandatory
  conditions (a)-(f). Condition (a) requires showing synthetic or anonymised
  data cannot do the job; the worked justification's `alternatives_considered`
  never mentions synthetic data, and Section 4 argues synthetic data "solves
  this tension" - the reverse of what 10(5)(a) asks.
- Cell 7: 10(2)(f) does **not** "specifically reference Article 21 of the
  Charter". Mitigation is 10(2)(g), not (f) (cells 23, 29).
- Cell 7: four-fifths rule "also referenced in EU equality case law" -
  unsupported; CJEU indirect-discrimination case law uses no fixed ratio.
  Contradicts the README's own limits section.
- 10(3) is quoted without "sufficiently" and "to the best extent possible".
- **Gender and age are not GDPR Article 9 special categories.**
  `01_proxy_detection.ipynb` cell 6 says 10(5) is what lets you retain gender;
  root Section 4 says demographic data "triggers Article 9". Wrong for two of
  the three attributes used.
- The 10(5) example is performance scoring (Annex III 4(b), not 4(a) as cell 29
  says), and 10(5) is a *provider* exception.

Confidence: high on all but the case-law point (no case-law search was run).

## 6. Impossibility theorems misstated

`03_impossibility_and_thresholds.ipynb` cell 0 says calibration, equal error
rates and demographic parity are "mathematically incompatible" and cites
Chouldechova/Kleinberg. **Neither theorem involves demographic parity**; both are
calibration/PPV vs FPR/FNR balance. The demo shows the DP-vs-calibration
tension, and forcing parity *halves* the TPR gap (0.140 to 0.074, robust over 8
seeds), which "breaks two other things" omits. `06_compas_recidivism.ipynb` cell
12 names "equal treatment of equal scores" as the third leg; it is not in the
theorem.

Ironically the COMPAS demo (equal FPR, FNR 49.2 vs 49.6, PPV 71.0 vs 59.5) *is*
Chouldechova exactly, and the text does not say so. Confidence: high.

## 7. Citation problems

- `07_proxy_detection_real_data.ipynb` cell 0: **Hardt, Price & Srebro did not
  use Adult**; their experiments are on FICO (0 hits for "Adult", 24 for "FICO"
  in the bundled PDF).
- `papers/README.md`: the bundled JMLR 2023 PDF has five authors (Corbett-Davies,
  Gaebler, Nilforoshan, Shroff, Goel), cited as "Corbett-Davies & Goel".
- `04_lipstick_on_a_pig.ipynb` cell 10 compares k-means 80.5% to G&G's 92.5%
  (word2vec hard-debias) but omits their GloVe-family figure (85.6%) and their
  SVM figure (88.88%, on a train-1000/test-4000 protocol; the notebook's 98.2%
  is 5-fold CV on the 1000 most-biased words). "Not sensitive to those choices"
  is untested. "Bias is still entirely there" overstates their "much of the bias
  information". The notebook never uses the word "reproduce", so the wording is
  defensible; the comparison is not.

## 8. Statistical soundness

- Notebooks 1, 2 (DIR), 3: stable across 8 seeds; fold std 0.005-0.02. Fine.
- Notebook 7 German Credit 0.688 is one split: 25 folds give 0.710 +/- 0.028,
  range 0.63-0.76.  Adult and LSAT-alone are stable (+/- 0.002-0.011).
- Notebook 6 cell 6: decile 10 is 50 white defendants; gap 0.14, SE 0.07.
  Deciles 3-4 are +0.07 at SE 0.042. "Holds through 9, breaks at 10, where the
  stakes are largest" is a 2-SE reading.
- Root notebook validation is one seed. Over 200 seeds: Black flagged 83%,
  Woman 47%, 46-60 48%, zero-penalty ethnicity groups 25-28%. Also the scoring
  criterion is "any penalty", but expected DIRs are Woman 0.82 and 46-60 0.93 -
  neither is a four-fifths violation by the rule's own definition, so "true
  positive" and "FALSE NEGATIVE" are mislabelled; Black (expected 0.71) is the
  real miss. The README story survives; the table does not.

## 9. Embarrassments

- Notebooks 1-4 argue against "Filter 2", "Filter 3" and "the output box" of a
  flowchart **that is not in the repo**.
- Root notebook says "four key requirements" then lists five.
- Blackstone is 1760s, not "three centuries".
- Notebook 2's "selective-labels penalty is real" is 0.005 AUC on fresh test -
  noise.

## Fine as is

COMPAS filtering matches ProPublica (5278 = 3175 + 2103) and the FPR/FNR numbers
hold; notebook 4's direction; notebook 7's Adult and LSAC numbers; the README
limits section; most of `papers/README.md`.