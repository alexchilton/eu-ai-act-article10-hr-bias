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
| `kusner-2017-counterfactual-fairness.pdf` | Kusner, Loftus, Russell, Silva, *Counterfactual Fairness*. [arXiv:1703.06856](https://arxiv.org/abs/1703.06856) |
| `kilbertus-2017-causal-reasoning.pdf` | Kilbertus et al., *Avoiding Discrimination through Causal Reasoning*. Separates a legitimate causal path from a discrimination channel. [arXiv:1706.02744](https://arxiv.org/abs/1706.02744) |
| `elazar-2018-adversarial-removal.pdf` | Elazar & Goldberg, *Adversarial Removal of Demographic Attributes from Text Data*. Adversarial training removes the attribute from the classifier, not from the representation. [arXiv:1808.06640](https://arxiv.org/abs/1808.06640) |
| `gonen-2019-lipstick-on-a-pig.pdf` | Gonen & Goldberg, *Lipstick on a Pig*. Debiasing word embeddings hides the bias rather than removing it. [arXiv:1903.03862](https://arxiv.org/abs/1903.03862) |
| `buolamwini-2018-gender-shades.pdf` | Buolamwini & Gebru, *Gender Shades* (FAccT 2018). Representativeness as a measurable data-quality failure. |
| `narayanan-2018-21-fairness-definitions.pdf` | Narayanan, *21 Fairness Definitions and Their Politics* (FAccT 2018 tutorial). Why the choice of definition is a value judgement. Talk is on YouTube. |
| `nist-sp-1270-managing-bias-in-ai.pdf` | NIST SP 1270, *Towards a Standard for Identifying and Managing Bias in AI*. |
| `barocas-hardt-narayanan-fairness-and-machine-learning.pdf` | Barocas, Hardt, Narayanan, *Fairness and Machine Learning: Limitations and Opportunities* (MIT Press 2023). The textbook. Free at [fairmlbook.org](https://fairmlbook.org/). |

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

## Data

`../data/compas-scores-two-years.csv` is the ProPublica COMPAS release, from
<https://github.com/propublica/compas-analysis>. 7,214 rows. Used in notebook 6.
Everything else under `../data/` is synthetic.