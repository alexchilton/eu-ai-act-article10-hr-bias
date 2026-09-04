# EU AI Act Article 10 - HR bias testing

A worked notebook on what Article 10 actually asks of an HR system, and how to
test for it. HR screening is Annex III high risk, so Article 10 applies in full.

Everything runs on synthetic data. No real candidates, no PII.

## What it covers

| Section | Article | What it does |
|---|---|---|
| 1. Data provenance | 10(2)(a) | Records where the data came from and how it was governed |
| 2. Demographic parity | 10(2)(f), 10(3) | Hire rates per group, disparate impact ratio, four-fifths rule |
| 3. Special category data | 10(5) | Drafts and completeness-checks the narrow 10(5) justification |
| 4. Synthetic data | 10(3) | Generates PII-free test data with deliberately injected bias |
| 5. Checklist | - | Rolls the five checks into a pass/warn/fail verdict |

## Results as it stands

Bias detection on the sample dataset:

```
Overall hire rate: 37.8%
FOUR-FIFTHS RULE VIOLATION  Woman       DIR = 0.78   33.5%
FOUR-FIFTHS RULE VIOLATION  Non-binary  DIR = 0.74   31.6%
FOUR-FIFTHS RULE VIOLATION  60+         DIR = 0.67   26.3%
FOUR-FIFTHS RULE VIOLATION  Hispanic    DIR = 0.68   27.3%

Overall hire rate  original 37.8%  ->  corrected 32.0%
```

Section 4 checks the detector rather than trusting it. Known bias goes into the
synthetic data, and the test has to find it:

```
Injected   gender:Woman -0.12, ethnicity:Group_C -0.15, age_group:Over 50 -0.10

DETECTED    Woman     DIR = 0.66
DETECTED    Group_C   DIR = 0.45
DETECTED    Over 50   DIR = 0.68
UNEXPECTED  Group_D   DIR = 0.78   not injected
```

Three of three recovered, and one false positive that the notebook reports
rather than hides.

Final checklist: **2 PASS, 2 WARN, 1 FAIL**. One failure has to be resolved
before deployment.

## Limits

- Synthetic data throughout. The numbers demonstrate the method, they say
  nothing about any real hiring process.
- The four-fifths rule is a US EEOC convention. It is a useful screen under
  Article 10(2)(f) but it is not what the AI Act specifies, and it is not a
  legal test on its own.
- The Article 10(5) justification is a drafting aid with a completeness check.
  It is not legal advice and it does not decide whether the exception applies.
- Correcting a disparity in the data does not make the system compliant. It is
  one of several obligations.

## Running it

Open `eu_ai_act_article10_hr_compliance.ipynb`. Needs numpy, pandas and
matplotlib. To point it at real data, replace `generate_synthetic_resumes()`
with an ATS export, and read the limits above first.

Outputs already in the repo: `article10_compliance_report.html`,
`article_10_5_justification.json`, and the two hire-rate charts.
