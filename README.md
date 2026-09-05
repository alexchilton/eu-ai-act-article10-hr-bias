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

## The point: validate the test, not just the data

Everything here is generated, so the true bias per group is known. That is the
whole reason for synthetic data. On real data you can measure a disparity but
you can never check whether your test was right, because nobody knows the
answer.

Run that check and the four-fifths rule does badly. Section 2 injects a known
penalty into six groups and the test is scored against it:

```
attribute  group        n    rate   DIR  flagged  injected  verdict
gender     Non-binary   19  31.6%  0.74  YES        -0.12   true positive
gender     Woman       242  33.5%  0.78  YES        -0.07   true positive
age_group  60+          19  26.3%  0.67  YES        -0.15   true positive
age_group  46-60       105  39.0%  1.00  no         -0.05   FALSE NEGATIVE
ethnicity  Hispanic     55  27.3%  0.68  YES        -0.06   true positive
ethnicity  Black        54  38.9%  0.97  no         -0.10   FALSE NEGATIVE
ethnicity  Other        29  31.0%  0.78  YES        +0.00   FALSE POSITIVE

12 groups: 4 true positive, 5 true negative, 2 FALSE NEGATIVE, 1 FALSE POSITIVE
The four-fifths rule got 3 of 12 groups wrong at n=500.
```

Black took the second-largest penalty in the dataset, -0.10, and the test read
DIR 0.97 and cleared it. `Other` had no penalty at all and got flagged, on 29
people.

Section 4 repeats the failure independently on a separate 400-row set: three
injected biases all recovered, plus `Group_D` flagged at DIR 0.78 with nothing
injected.

So the headline is not "here is how to run a bias test". It is that a
compliance test you have not validated can clear a group that really was
discriminated against, and the report will look clean.

## Other results

```
Overall hire rate  original 37.8%  ->  corrected 32.0%
```

Final checklist: **2 PASS, 2 WARN, 1 FAIL**. One failure has to be resolved
before deployment.

## Limits

- Synthetic data throughout. The numbers demonstrate the method, they say
  nothing about any real hiring process.
- The four-fifths rule is a US EEOC convention. It is a useful screen under
  Article 10(2)(f) but it is not what the AI Act specifies, and it is not a
  legal test on its own. As the results above show, at these group sizes it
  also misses real bias.
- Group sizes here are small. `Non-binary` and `60+` are 19 people each, `Other`
  is 29. A rate on 19 people is noise. That is a property of the demo, but it is
  also what a real HR dataset looks like for the groups that matter most.
- The provenance records in Section 1 describe fictional sources. The files
  exist and the SHA-256 hashes are of those files, but "Workday, tenant:
  acme-corp" and the rest are invented. Replace them before the report means
  anything.
- The Article 10(5) justification is a drafting aid with a completeness check.
  It is not legal advice and it does not decide whether the exception applies.
- Correcting a disparity in the data does not make the system compliant. It is
  one of several obligations.

## Running it

Open `eu_ai_act_article10_hr_compliance.ipynb`. Needs numpy, pandas and
matplotlib, nothing else. Run all cells; every number in this README is
reproduced exactly, because every generator is seeded.

To point it at real data, replace `generate_synthetic_resumes()` with an ATS
export and rewrite the Section 1 provenance records. Read the limits first.

`data/` holds the three CSVs the Section 1 provenance records name, written by
the notebook and hashed by it. They are synthetic: `SYN-` ids, invented names.

Outputs already in the repo: `article10_compliance_report.html`,
`article_10_5_justification.json`, and the two hire-rate charts.
