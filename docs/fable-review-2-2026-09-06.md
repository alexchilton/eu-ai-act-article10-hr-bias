# Fable review 2, 2026-09-06

Second adversarial pass, after commits 0884de1, 3a3b0d0, ab8d7fa, 1c7c7c2.
Method: every notebook re-run from its own source with the kernel's interpreter (`/usr/local/bin/python3.12`, sklearn 1.7.1, numpy 2.3.1, pandas 2.3.1, gensim 4.4.0), then attacked with more seeds, held-out splits, and the papers' own protocols.
Nothing in the repo was edited by the reviewer.

**Status: all of Section A is closed, and Section B items 1, 4, 5, 6, 7 and 9
are done.** A1-A4 in commit `733a321`; A5-A8 and the improvements in the commit
that follows it. Not done: fairlearn's `ExponentiatedGradient` (a parity penalty
implemented directly was used instead, so no new dependency) and re-running
notebook 7 on ACSIncome (needs `folktables`). The items the review itself
recommends against are not done, by agreement with it.

Skipped: a per-column adversary for notebook 1 (the script crashed on sparse one-hot output; not needed for any finding).

---

## SECTION A: what is still wrong

Ordered most serious first.

### A1. Notebook 2's figure still scores in-sample and still states the retracted claim

`notebooks/02_label_choice_and_selective_labels.ipynb` cell 10 (the chart) does `m.predict_proba(X)` and `roc_auc_score(repaid, p)` on all 20,000 rows, half of which are the training rows.
Cell 5 was fixed to a held-out split; the chart was not.

| model | cell 5, held out | cell 10 chart, all rows | on training rows only |
|---|---|---|---|
| A human decision | 0.627 | 0.625 | 0.623 |
| B repayment, approved only | 0.622 | **0.698** | 0.774 |
| C repayment, everyone | 0.636 | **0.728** | 0.820 |

The generator's ceiling, printed by the notebook itself in cell 5, is 0.667. The committed `figures/02_label_choice.png` shows C at 0.73, above the ceiling, under the suptitle **"The human-decision label is worse on both axes at once"**, which is the sentence cell 8 now retracts.
The DIR bars in the chart (0.33 / 0.82 / 0.79) are also the in-sample values, not the 0.34 / 0.80 / 0.75 of cell 5.
Confidence: high.

### A2. Notebook 2's headline sentence is false in its own generator

Cell 0: *"No correction technique recovers as much as choosing the right label in the first place."*
No correction technique is run. Three were run on label A, 12 seeds, same 50/50 split, held out:

| trained on | AUC vs true repayment | sd | DIR | sd |
|---|---|---|---|---|
| A, as is | 0.635 | 0.006 | 0.35 | 0.012 |
| A + Kamiran-Calders reweighing | 0.647 | 0.005 | 0.73 | 0.022 |
| A + per-group thresholds (post-processing) | 0.635 | 0.006 | 1.00 | 0.000 |
| A with the three proxies dropped | **0.651** | 0.007 | **0.77** | 0.015 |
| C, right label | 0.647 | 0.005 | 0.71 | 0.040 |

Reweighing the wrong label lands exactly where the right label does. Dropping the proxies from the wrong label beats the right label on both axes. Post-processing beats it on DIR at no AUC cost.
The reason is in cell 3: `risk` gives `postcode` weight 0.05 and `phone_carrier` and `occupation_cat` weight zero. The proxies carry only the human penalty and no repayment signal, so removing them is free by construction.
That is the opposite premise to notebook 1, whose whole point is that proxies carry legitimate signal and cannot be dropped for free. The two generators contradict each other and the README presents both results as findings.
Confidence: high.

### A3. The root notebook's new correction still manufactures the FAIL the checklist reports

`eu_ai_act_article10_hr_compliance.ipynb` cell 14 scores candidates on `years_experience` and `education_level` only, then sets per-group thresholds **for gender only** (`for col in [SENSITIVE[0]]`).
The corrected decision is therefore blind to ethnicity and age by construction: the population DIR of every ethnicity and age group under the corrected rule is 1.00.
Cell 26 then runs the checklist on those corrected decisions and reports `B1 FAIL: ['age_group:18-30 (DIR=0.80)', 'age_group:60+ (DIR=0.59)', 'ethnicity:Black (DIR=0.67)']`.

Measured, seed 42, n=500:

| group | DIR before | DIR after correction | n |
|---|---|---|---|
| Black | 0.97 | **0.67** | 54 |
| 60+ | 0.67 | 0.59 | 19 |
| 18-30 | 0.99 | 0.7997, printed as "0.80" and flagged | 176 |

Over 200 seeds, applying the notebook's own correction: the checklist fails B1 after correction in **97%** of seeds; an ethnicity group is flagged after correction in 85%; Black is flagged after correction in 36.5% of seeds on a rule that cannot see ethnicity.
So the README's *"Final checklist: 2 PASS, 2 WARN, 1 FAIL. One failure has to be resolved before deployment"* and `article10_compliance_report.html` report sampling noise from n=54 and n=19 as a violation, and the notebook does not say so.
This is finding 2 from the first review in a new form: last time the correction levelled down; this time it re-rolls the dice on groups it never touched, and the "18-30 (DIR=0.80)" line is a rounding embarrassment on top.
Also the chart `hire_rates_before_after.png` is still titled "Hire Rate Before vs After **Reweighting** Correction" and its ethnicity-after panel shows Black in red.
Confidence: high.

Smaller, same cell: attributing per-group *selection-rate* thresholds to Hardt, Price & Srebro is loose. Their post-processing equalises TPR/FPR (separation). Equal selection rates by threshold is the independence version (Thorndike 1971 / Feldman et al. 2015 lineage; fairlearn's `ThresholdOptimizer(constraints="demographic_parity")`). Notebook 5's description of Hardt et al. is correct; the root notebook's is not.

### A4. Notebook 5: the "drop a" row is a slicing bug, and INLP still deletes the space on half the seeds

`notebooks/05_linear_vs_adversarial_removal.ipynb` cell 5: `report(X[:, [1, 2, 3]], "drop 'a', the visible proxy")`.
That keeps 3 of the remaining 11 columns, not 11. Measured, seed 5:

| | linear | tree |
|---|---|---|
| notebook's `X[:, [1,2,3]]` | 0.501 | 0.899 |
| actual drop-a, `X[:, 1:]` | 0.499 | **0.983** |

The direction survives; the printed 0.899 and the chart bar are wrong. Cell 8 says *"This version uses eleven columns"*; cell 3 prints `X has 12 columns`.

INLP, cell 7. The stopping rule is `norm(coef) < 1e-8`, which is not INLP's rule (Ravfogel et al. stop when probe accuracy reaches chance). Measured over seeds:

| seed | projections applied | printed | rank left | tree AUC |
|---|---|---|---|---|
| 5 (notebook) | 4 | "5" | 8 | 0.979 |
| 2 | 4 | "5" | 8 | 0.969 |
| 0 | 12 | "12" | **1** | 0.783 |
| 1 | 12 | "12" | **1** | 0.683 |

The printed count is off by one when the loop breaks. On seeds 0 and 1 the loop projects all 12 directions, leaves rank 1, and the tree reads float residue: this is finding 4 from the first review, back on two of four seeds. The coefficient norms show why: after **one** projection the probe's coefficient norm is already 0.03-0.05; every later step removes a noise direction with a regularised-to-nothing coefficient. README row 05 *"INLP leaves rank 8 of 12"* is seed luck.
Confidence: high.

### A5. Notebook 4 under Gonen & Goldberg's own protocol

The notebook's numbers reproduce exactly (0.2192 / 100% / 100% before; 0.0000 / 80.5% / 98.2% after).
Cell 10 says the 98.2% is "an easier protocol" than G&G's 88.88%. Their protocol was run on the same debiased GloVe: 5,000 most-biased words, RBF-SVM trained on a random 1,000, tested on the other 4,000, five draws: **93.0%** (92.8-93.3). k-means on the 5,000: 71.3% after (98.9% before).
So the caveat is correct in direction and the comparable number is 93.0%, which is above G&G's, not below it. Worth stating instead of leaving the reader to guess.
One control the notebook lacks: an SVM on 1,000 *mid-ranked* words (bias rank ~15,000) after debiasing gets **55.4%**. The residual gender information is concentrated in the extreme words; on ordinary vocabulary it is close to chance. Cell 0's *"the bias is still entirely there"* and cell 10's *"Nothing was removed"* are both stronger than the data, and cell 0 still uses "entirely" after cell 10 was softened to "much of".
Confidence: high on the numbers; the mid-rank control is one draw.

### A6. Notebook 7 LSAC: the numbers hold, the column labels do not

All numbers reproduce: Adult 0.938 / 0.878 / 0.823 / 0.720 / 0.670 / 0.644; German 0.699 +/- 0.036, range 0.64-0.77; LSAC 0.828 / 0.779 / 0.716 / gender 0.625; LSAT-alone is stable over five shuffles (0.713-0.718).
But OpenML 43890 is "Law School Admissions (Binarized)" from the R `fairml` package: a 1991 survey of **students attending** law school, `race1` pre-binarised to white/non-white, `decile1` and `decile3` = law-school grade deciles (year 1 and 3), `bar` = bar passage, `cluster` = school cluster.
So cell 11's "20,800 real applicants" are enrolled students; "academic record only" includes two years of law-school grades an applicant does not have; and "all remaining features" includes bar passage. The "LSAT alone recovers race at 0.716" headline is unaffected.
Confidence: high on the data source (from the OpenML `DESCR`); moderate on the exact decile semantics (from the LSAC codebook as recalled, not re-read).

### A7. Notebook 3 wording

Cell 6: *"calibration-conditional-on-score ... get[s] worse"* under forced parity. Re-thresholding does not change the score `p`, so its calibration cannot change. What changes is the PPV of the binary decision (predictive parity). Numbers reproduce exactly. Confidence: high.

### A8. Stale text the fixes left behind

- `README.md` line 16: section table still says `10(5)`; line 133 still says "The Article 10(5) justification".
- `README.md` line 141: "Needs numpy, pandas and matplotlib, nothing else." Root cell 14 now imports sklearn.
- Root cell 15: chart title "Reweighting Correction". Cell 24 table: `10(2)(g) - bias mitigated | compute_sample_weights()`, a function that no longer exists. Cell 30: "if violations persist after reweighting".
- Root cell 17 comment and cell 25 docstring still say Article 10(5); the schema string and filename are `_10_5_` (cosmetic, but the checklist row says 4a).
- Root cell 4 prints `wrote data/compas-scores-two-years.csv`. It lists the directory; it did not write that file.
- Notebook 7 cell 0: "three standard public benchmarks", then lists two.
- Notebook 1 chart title "Removing the named proxies barely moves the adversary": the drop is 0.058, about 15% of the excess over chance. "Barely" is editorial.

### Clean bills

- **Notebook 1** is now stable: 8 seeds, 0.889 +/- 0.003 / 0.826 +/- 0.004 / 0.746 +/- 0.004 / 0.587 +/- 0.008. `school` and `postcode` carry signal. Fix holds.
- **Notebook 2 DIR result** holds: 0.35 vs 0.73 / 0.71, 12 of 12 seeds, and the A-vs-B accuracy null (+0.002) holds.
- **Notebook 3** reproduces to the third decimal.
- **Notebook 6** reproduces exactly: 5278 = 3175 + 2103; FPR 42.3 / 22.0; FNR 28.5 / 49.6; decile 10 is 227 vs 50 with gap +14% at SE 0.07. Chouldechova attribution is now correct.
- **Notebook 7** reproduces exactly (numbers above).
- **Root notebook** reproduces exactly: 37.8% -> 38.6%, 90 gained / 86 lost, the 12-row validation table, population DIR Black 0.74.
- **Legal headline**: Regulation (EU) 2026/1744 exists, entered into force 27 July 2026, deleted Article 10(5) and inserted Article 4a with the strict-necessity conditions the notebook lists. Checked against EUR-Lex OJ L 2026/1744 and the Commission's notice. The 24 July publication date was not verified to the day.

---

## SECTION B: improvements and missing ideas

Ranked by value per effort.

1. **Put a confidence interval on every DIR.** (Root, ~20 lines.) The root notebook's entire argument is "the four-fifths rule fails at n=500", and A3 shows the correction step then reports noise as a violation. A Wilson or bootstrap interval on each group's rate, propagated to the ratio, makes the point in one column: at n=19, DIR 0.59 has an interval that spans 0.8 either way. Add a power cell: how many candidates per group to detect DIR 0.80 at 80% power. That is the number an HR auditor needs and nothing in the repo gives it.
2. **Make the root correction consistent, then teach from the residue.** Threshold on the joint group or state plainly that the corrected rule is blind to age and ethnicity by construction and that any flag it raises is sampling noise. Turn the 97%-of-seeds FAIL into the lesson: a compliance test on a rule you know to be group-blind still fails 97% of the time at these group sizes. That is a better headline than the one in the README, and it is already measured.
3. **Run the corrections in notebook 2 and rewrite the claim to what the table shows.** Give `postcode`, `carrier` and `occupation` non-zero weight in `risk` so notebooks 1 and 2 share one world. Then the comparison becomes real: dropping proxies costs repayment accuracy, reweighing does not, and label choice is one axis among several.
4. **Fix notebook 5's INLP loop.** Use `X[:, 1:]`, stop on held-out probe accuracy within 0.01 of 0.5 (the paper's rule), print projections applied, run 8 seeds, report the range. Ten minutes and it closes the last open finding from round one.
5. **Add the missing family: in-training constraints.** The repo covers pre-processing (drop, project) and post-processing (thresholds) and never trains under a constraint. One notebook with fairlearn's `ExponentiatedGradient` on Adult at equal demographic parity, against `ThresholdOptimizer` at the same parity, with accuracy on the same held-out split, completes the three-way taxonomy and shows the accuracy price of each. Medium effort, high teaching value.
6. **Group ROC curves and the feasible region on Adult.** The repo shows independence vs calibration (nb03) and calibration vs error rates (nb06) but never draws the two group ROC curves and the intersection under both. That figure is the clearest picture of why separation needs group-specific thresholds or randomisation. Cheap on Adult or COMPAS deciles.
7. **Show the selective-labels evaluation trap, not just the training one.** Notebook 2 trains B on approved-only rows and evaluates on everyone. Add one line evaluating B on approved-only test rows: the AUC a real lender would report versus the AUC against truth. The gap is the lesson, and the simulation is the only place you can measure it.
8. **A synthetic resume audit.** The root generator already assigns gendered first names and never uses them. Train the screening model on names + features, then flip names and hold everything else fixed. That is the Bertrand-Mullainathan design in 30 lines and it lands harder for HR than a DIR table.
9. **Intersectional table in the root notebook.** Gender x ethnicity at n=500 gives cells under 10. Ten lines. The summary already promises it as a "next step".
10. **One cell on why Adult is a bad fairness benchmark**, with folktables' ACSIncome as the replacement, or better, re-run notebook 7's decay on ACSIncome. Medium effort; it removes the repo's dependence on a 1994 file the field has retired.

Not worth doing:

- Implementing counterfactual fairness. Notebook 5 already says the honest thing: it needs a causal graph you can defend, and synthetic data cannot supply one.
- More embedding-debiasing variants (INLP on GloVe, GN-GloVe). Notebook 4 already lands; A5's mid-rank control is the one addition worth a cell.
- A library tour (fairlearn vs aif360) or a Streamlit front end. Neither adds a measurement.
- Rewriting the compliance checklist as a framework. The owner said code, not governance; the checklist is fine as a demo once A3 is fixed.

---

## SECTION C: gap analysis against Barocas, Hardt & Narayanan

Read from the bundled PDF (compiled 13 Dec 2023, 9 chapters, 262 pages). Chapter references are to that build.

### What the repo already demonstrates in code

| Book | Repo |
|---|---|
| Ch1 "The trouble with measurement": the target variable is a construct; arrests stand in for crime, performance reviews for "good employee" | nb02 (approval vs repayment), nb06 section 5 (re-arrest vs reoffending). Covered, and nb02 is the stronger version because the true outcome is observable. |
| Ch1 "From data to models": proxies / redundant encodings; "we can't just get rid of proxies: they may be genuinely relevant" | nb01, nb07. Covered on synthetic and real data. |
| Ch1 toy hiring example: GPA is a proxy; dropping it hobbles the model; "pick different cutoffs" is crude; EEOC 20% is not a bright line | Root cell 14 does exactly the crude cutoff move the book describes; README's limits section matches the book on four-fifths. |
| Ch3 "No fairness through unawareness", Fig 3.4: many weakly-correlated features build a strong group classifier | nb01 and nb07 are that figure, measured. |
| Ch3 Independence (demographic parity, the 0.8 ratio relaxation) | Root DIR, nb02, nb03. |
| Ch3 Sufficiency / calibration by group | nb03 `calibration_curve`, nb06 decile table. |
| Ch3 Separation (equal FPR and FNR) | nb06 FPR/FNR, nb03 TPR gap. |
| Ch3 Prop. 2: independence and sufficiency cannot both hold when A and Y are dependent | nb03 is a numerical instance of Prop. 2, and now says so correctly ("arithmetic, not the impossibility result"). |
| Ch3 Prop. 4 and 5: separation and sufficiency cannot both hold | nb06 is Prop. 5 by hand, PPV formula included. |
| Ch3 "How to satisfy": pre-processing and post-processing | Pre: nb01, nb04, nb05. Post: root cell 14, nb03, nb06 cell 11. |
| Ch7 "Separation and selective labels" | nb02 model B, partially (see B7). |
| Ch9 datasheets / documentation | Root `DataProvenanceLog` is a five-field datasheet. Worth naming it as such. |
| Ch9 leakage as a cause of inflated performance | The repo has now lived it (nb02) and documented the fix. |

### What the book argues that the repo does not touch

Ranked by how much a practitioner running an HR audit will miss it.

1. **Ch3 "Calibration by group as a consequence of unconstrained learning" (Fact 3, Liu et al. 2019).** The book's sharpest link between the repo's two halves: sufficiency comes free from a good model *precisely because* group membership is predictable from the features. nb01/nb07 prove the premise; nb03/nb06 show the consequence; no notebook connects them. One markdown paragraph and one experiment (train on Adult without `sex`, plot calibration by sex) closes it. Practitioner-relevant: it explains why "we did not use the attribute" and "the model is calibrated by group" are not two findings but one.
2. **Ch3 "Conditional acceptance rates" and Ch7 rows 5-6 (regression tests, conditional demographic parity).** Every real HR audit hits "but the gap disappears once you control for role and level". The book's answer is that the choice of what to condition on decides the verdict and that you must not condition on the mechanism of discrimination. The repo never computes a conditional DIR. This is the most practical gap in the list.
3. **Ch3 case study: credit scoring.** Group ROC curves, four threshold strategies (max profit, single threshold, independence, separation), group-specific trade-offs. The repo has no group ROC curve anywhere. See B6.
4. **Ch3 "How to satisfy": in-training.** Absent. See B5.
5. **Ch7 audit studies and resume audits (Bertrand & Mullainathan), and the book's caution that they test blindness only.** Nothing in the repo intervenes on an input and holds the rest fixed. See B8. The book's warning that attribute-flipping "does not generally produce counterfactuals that we care about" should go next to it.
6. **Ch7 outcome test and infra-marginality.** nb06 compares PPV across races after equalising FPR. The book says comparing PPV across groups is the outcome test, and inferring different thresholds from it is a fallacy. nb06 does not make that inference, but a reader will. One paragraph.
7. **Ch9 on Adult.** The book says Adult is atypical: a logistic model is *more* accurate on Black and female rows because almost all are below the $50k cutoff, and it points to "Retiring Adult" (Ding et al. 2021). nb07 uses Adult as "the standard fairness benchmark" without the caveat. See B10.
8. **Ch1 feedback loops and "predictions that affect the training set".** nb06 section 5 has one paragraph on detention causing the outcome. No simulation. A two-round loop in the root generator (hire, observe performance of hires only, retrain) would show label drift in 30 lines. Medium value; the selective-labels version (B7) is the cheaper first step.
9. **Ch2 legitimacy: target vs goal mismatch, recourse, "failing to consider relevant information".** The book reads Obermeyer as a target-goal mismatch, which is how nb02 uses it. The rest of the chapter is normative and belongs in the README's limits, not in code.
10. **Ch4 relative notions of fairness.** This is the chapter that answers nb03's "which of those you want is a judgement" and nb06's "which error to equalise is a policy act". Its point that error-rate parity is "the hardest criterion to rigorously connect to any moral notion" and that the two error types carry asymmetric moral weight is exactly the argument nb06 needs and does not have. Prose, not code.
11. **Ch5 causality.** Absent from code. Berkeley admissions, direct vs indirect effects, path inspection, counterfactuals. nb05 names it in one paragraph and the Kusner/Kilbertus PDFs are bundled. The book itself says the direct effect "cannot detect any form of proxy discrimination", which is the repo's thesis stated causally. Academic for this repo: without a defensible graph there is nothing to run, and the book spends ten pages on why the graph is hard to defend.
12. **Ch6 US anti-discrimination law.** Not applicable to an EU repo beyond the four-fifths provenance, which the README already has right.
13. **Ch8 structural discrimination (Uber earnings gap, three levels of discrimination).** Out of scope for a code repo. The one transferable line is that the study's narrow definition of discrimination missed a 2.7x participation gap while explaining a 7% earnings gap, which is a warning about what a DIR table cannot see.

### Summary for the holiday read

Read Chapter 3 first and closely; the repo is a numerical companion to it and already covers independence, sufficiency, separation and two of the three impossibility propositions. Then Chapter 7 (the tests you can actually run, and their limits) and the Adult passage in Chapter 9, because those change what the repo should do next. Chapters 4 and 5 will sharpen the prose in nb03, nb05 and nb06 but will not produce a notebook. Chapters 2, 6 and 8 are context.
