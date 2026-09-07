# Papers

The sources behind the notebooks. Open access only. Each is cited from the
notebook that uses it.

## Downloaded here

| File | Paper |
|---|---|
| `dwork-2012-fairness-through-awareness.pdf` | Dwork, Hardt, Pitassi, Reingold, Zemel, *Fairness Through Awareness* (ITCS 2012). Named "fairness through unawareness" and showed why it fails. [arXiv:1104.3913](https://arxiv.org/abs/1104.3913) |
| `hardt-2016-equality-of-opportunity.pdf` | Hardt, Price, Srebro, *Equality of Opportunity in Supervised Learning* (NeurIPS 2016). Equalised odds. [arXiv:1610.02413](https://arxiv.org/abs/1610.02413) |
| `kleinberg-2016-inherent-tradeoffs.pdf` | Kleinberg, Mullainathan, Raghavan, *Inherent Trade-Offs in the Fair Determination of Risk Scores*. First impossibility result. [arXiv:1609.05807](https://arxiv.org/abs/1609.05807) |
| `chouldechova-2017-fair-prediction.pdf` | Chouldechova, *Fair Prediction with Disparate Impact*. The second impossibility result, written directly about recidivism scoring. [arXiv:1610.07524](https://arxiv.org/abs/1610.07524) |
| `corbett-davies-2018-measure-and-mismeasure.pdf` | Corbett-Davies, Gaebler, Nilforoshan, Shroff & Goel, *The Measure and Mismeasure of Fairness* (JMLR 2023; the 2018 arXiv preprint is Corbett-Davies & Goel). The strongest critique of naive fairness constraints, including the case that they can make everyone worse off. [arXiv:1808.00023](https://arxiv.org/abs/1808.00023) |
| `kusner-2017-counterfactual-fairness.pdf` | Kusner, Loftus, Russell, Silva, *Counterfactual Fairness*. Used in notebook 9. [arXiv:1703.06856](https://arxiv.org/abs/1703.06856) |
| `kilbertus-2017-causal-reasoning.pdf` | Kilbertus et al., *Avoiding Discrimination through Causal Reasoning*. Separates a legitimate causal path from a discrimination channel. Notebook 9 is the case where the separation cannot be made, because the graph is the disagreement. [arXiv:1706.02744](https://arxiv.org/abs/1706.02744) |
| `elazar-2018-adversarial-removal.pdf` | Elazar & Goldberg, *Adversarial Removal of Demographic Attributes from Text Data*. Adversarial training removes the attribute from the classifier, not from the representation. [arXiv:1808.06640](https://arxiv.org/abs/1808.06640) |
| `gonen-2019-lipstick-on-a-pig.pdf` | Gonen & Goldberg, *Lipstick on a Pig*. Debiasing word embeddings hides the bias rather than removing it. [arXiv:1903.03862](https://arxiv.org/abs/1903.03862) |
| `buolamwini-2018-gender-shades.pdf` | Buolamwini & Gebru, *Gender Shades* (FAccT 2018). Representativeness as a measurable data-quality failure. |
| `narayanan-2018-21-fairness-definitions.pdf` | Narayanan, *21 Fairness Definitions and Their Politics* (FAccT 2018 tutorial). Why the choice of definition is a value judgement. Talk is on YouTube. |
| `nist-sp-1270-managing-bias-in-ai.pdf` | NIST SP 1270, *Towards a Standard for Identifying and Managing Bias in AI*. |
| `barocas-hardt-narayanan-fairness-and-machine-learning.pdf` | Barocas, Hardt, Narayanan, *Fairness and Machine Learning: Limitations and Opportunities* (MIT Press 2023). The textbook. Free at [fairmlbook.org](https://fairmlbook.org/). |
| `cinelli-2020-making-sense-of-sensitivity.pdf` | Cinelli & Hazlett, *Making Sense of Sensitivity: Extending Omitted Variable Bias* (JRSS-B 2020). The robustness value. Notebook 9 runs it where the confounder is known: the bias bound recovers the true bias to four decimals, while the headline robustness value alone would have cleared a confounder that in fact explains the whole estimate. [doi:10.1111/rssb.12348](https://doi.org/10.1111/rssb.12348) |
| `dressel-2018-accuracy-fairness-limits.pdf` | Dressel & Farid, *The accuracy, fairness, and limits of predicting recidivism* (Science Advances 2018, CC BY-NC). COMPAS is matched by a two-feature model. Replicated in notebook 9: age and prior count reach AUC **0.728** against the 137-item instrument's **0.711**. [doi:10.1126/sciadv.aao5580](https://doi.org/10.1126/sciadv.aao5580) |

## Not downloadable, read at the link

- **Obermeyer, Powers, Vogeli, Mullainathan (2019), _Dissecting racial bias in an
  algorithm used to manage the health of populations_, Science.** Paywalled.
  <https://www.science.org/doi/10.1126/science.aax2342>
  The single most useful paper here. A health system's algorithm predicted
  **cost** as a stand-in for **need**; less was spent on Black patients, so it
  rated them healthier at equal sickness. Changing the label removed the bias.
  Notebook 2 is built on this idea.
- **Lakkaraju, Kleinberg, Leskovec, Ludwig, Mullainathan (2017), _The Selective
  Labels Problem_, KDD.** You only observe the outcome for applicants somebody
  approved. Also notebook 2.

## Cited in the notebooks, not bundled

Linked rather than downloaded. Each is named where it is used.

- **Ravfogel, Elazar, Gonen, Twiton & Goldberg (2020), _Null It Out: Guarding
  Protected Attributes by Iterative Nullspace Projection_ (ACL).** INLP.
  Notebook 5 uses their stopping rule - project until the probe reaches chance -
  and an earlier version of that notebook got a different answer by using a
  coefficient-norm rule instead. <https://arxiv.org/abs/2004.07667>
- **Kamiran & Calders (2012), _Data preprocessing techniques for classification
  without discrimination_, KAIS.** Reweighing. Notebook 2 implements it in five
  lines: weight each (group, label) cell by `P(group)P(label) / P(group, label)`.
- **Liu, Simchowitz & Hardt (2019), _The Implicit Fairness Criterion of
  Unconstrained Learning_ (ICML).** Why calibration by group falls out of a model
  that predicts the outcome well. Notebook 7 measures it on Adult.
  <https://arxiv.org/abs/1808.10013>
- **Ding, Hardt, Miller & Schmidt (2021), _Retiring Adult: New Datasets for Fair
  Machine Learning_ (NeurIPS).** Why UCI Adult is a poor benchmark and what to
  use instead. Noted in notebook 7. <https://arxiv.org/abs/2108.04884>
- **Agarwal, Beygelzimer, Dudik, Langford & Wallach (2018), _A Reductions
  Approach to Fair Classification_ (ICML).** The `ExponentiatedGradient` method
  notebook 8 uses for in-training constraints. <https://arxiv.org/abs/1803.02453>
- **Bertrand & Mullainathan (2004), _Are Emily and Greg More Employable than
  Lakisha and Jamal?_, AER.** The resume audit design. Referenced as the
  experiment the root notebook does not run.
- **Westreich & Greenland (2013), _The Table 2 Fallacy_, Am J Epidemiol
  177(4):292-298.** One regression returns k coefficients and at most one of them
  is a causal effect. Notebook 9 section 1b demonstrates it: adding a mediator
  raises R-squared from 0.578 to 0.742 and drives one true coefficient of +0.60
  to -0.010, while the other stays correct.
- **Westreich & Greenland (2013)**, **Holland (1986)** and **Greiner & Rubin
  (2011)** below are paywalled, so they are cited and not bundled. Everything in
  the table above is open access and redistributable.
- **Holland (1986), _Statistics and Causal Inference_, JASA 81(396):945-960**, and
  **Greiner & Rubin (2011), _Causal Effects of Perceived Immutable
  Characteristics_, Rev Econ Stat 93(3):775-785.** No causation without
  manipulation, and what is actually manipulable when the attribute is race.
  Notebook 9 section 4.

## Data

`../data/compas-scores-two-years.csv` is the ProPublica COMPAS release, from
<https://github.com/propublica/compas-analysis>. 7,214 rows. Used in notebooks 6
and 9. Note that its `two_year_recid` column records a re-arrest, not a
reoffence: 67.6% of the events it counts are misdemeanour arrests and 23.3% are
violent. Notebook 9 section 3 is about what follows from that.

`../data/acs/` is the folktables cache: the 2018 one-year ACS person file for
California, 378,817 rows, downloaded on first run by notebook 7. It is US Census
public-use microdata, already de-identified at source.

UCI Adult, German Credit and LSAC are fetched from OpenML at run time and not
stored in the repo. Everything else under `../data/` is synthetic.