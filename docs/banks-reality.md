# What lenders actually do, and where the AI Act lands on it

Written 2026-09-07.
Prompted by a Coursera responsible-AI exercise: loans approved for 80% of men
and 30% of women, remove gender, both come out near 45%, done.

**Legal position as at 7 September 2026.**
The AI Act was amended by **Regulation (EU) 2026/1744** ("the Digital Omnibus on
AI"), published 24 July 2026 and in force from 27 July 2026.
That amendment **deleted Article 10(5)** and moved the legal basis for
processing special categories of personal data for bias detection into a new
**Article 4a**: 4a(1) sets six cumulative conditions for providers of high-risk
systems, and **4a(2) extends the basis to other providers and to deployers**.
The high-risk timetable also moved: Annex III obligations now apply from
**2 December 2027** for systems classified under Article 6(2) - which is where
credit scoring sits - and 2 August 2028 under Article 6(1).
This file was first written citing Article 10(5) as live law and the old
2 August 2026 date, and was corrected on 7 September 2026.
Check for later amendments before relying on any of it.

**Provenance, because this file is different from the rest of the repo.**
Everything else here is measured - a notebook runs, a number comes out, and the
number is in the README.
This file is mostly not.
Sections 1-8 are legal and industry background, cited from memory of the sources
named, and not verified against the primary texts in this session.
Article and case numbers should be checked before any of it is repeated
somewhere that matters.
Where I am unsure I say so inline.

**Section 9 is the exception and is measured**: the Fairlearn API counts and
method signatures in it came from importing `fairlearn 0.14.0` under
`python3.12` on this machine on 2026-09-07 and enumerating the package.
Those numbers are checkable by re-running the same import.

Nothing else in this file is a finding of this repo.

---

## 1. The toy result is arithmetic, not a finding

In the exercise gender is the only signal.
Delete it and nothing is left but the pooled base rate, so 45% is
`(0.8 x n_men + 0.3 x n_women) / n`.
Any generator built that way ends the same way.
It is not evidence that blinding works.

Three separate problems are folded together in the exercise and none of them is
named.

**The label.**
Nobody says what "approved" means.
Train on the loan officer's decision and you learn the loan officer, prejudice
included.
Train on repayment and you learn something closer to reality - but repayment is
only observed for applicants somebody approved.
That is the selective-labels problem (Lakkaraju et al. 2017), which notebook 2
is built on.
If women were historically approved only when exceptionally strong, their
observed default rate is *low* and a naive model concludes women are safe.
The bias inverts.

**Proxies.**
Deleting gender does not delete gender.
Measured on Adult in this repo: dropping one proxy takes AUC 0.938 to 0.878, and
it takes six removals to reach 0.644.
It takes six because the information is spread thin.
The load-bearing proxies are also the legitimate predictors - income *is* the
risk factor and *is* the channel gender operates through, at the same time.
There is no version where you keep the first and drop the second.

**Blindness destroys measurement.**
Delete the attribute and you cannot audit for bias in it.
This is Dwork et al. (2012), and it is why the AI Act carries a derogation for
holding the attribute at all - Article 10(5) until July 2026, Article 4a since.

## 2. Blinding can make it worse, not just useless

If a feature means different things across groups - "part-time" as instability
in one case and childcare in another - a model *with* the attribute can learn the
interaction.
A model without it applies one pooled coefficient to everyone, which in practice
is the majority-calibrated coefficient applied to the minority.
Accuracy and fairness degrade together.
Corbett-Davies & Goel argue this at length in the bundled JMLR paper.

## 3. What lenders actually run

The gap between this literature and production is not laziness.
The law forbids the main academic move.

**In US consumer credit, ECOA / Regulation B makes it illegal to collect or use
race, sex or marital status for most lending decisions.**
Not discouraged - prohibited.
The large exception is mortgages, where HMDA *requires* collecting race,
ethnicity and sex.

So outside mortgage the lender does not hold the attribute, "retain it for the
audit" is unimplementable, and every method that adjusts a score by group at
decision time is likely unlawful disparate treatment.
Nearly all of the fairness-constraint literature assumes the attribute is
available when you decide.
In US credit it is not.
That one fact explains most of the distance.

What is actually deployed:

| Practice | Status |
|---|---|
| Blindness | Forced by statute, not chosen |
| BISG proxy imputation for auditing | Standard in mortgage and auto |
| Disparate-impact regression testing | Decades old, routine compliance |
| Reject inference | Standard, done for accuracy not fairness |
| Less-discriminatory-alternative search | Real, commercial, growing, minority |
| Special Purpose Credit Programs | Real, the legal route to a remedy |

**BISG** - Bayesian Improved Surname Geocoding, imputing race from surname and
census tract.
CFPB published the methodology in 2014 and used it in the auto-lending
enforcement wave; Ally Financial settled for $98M in 2013.
It is contested: industry argued it overestimates, and Congress repealed the
CFPB's auto-lending guidance in 2018 under the Congressional Review Act.
The proxy method is itself a political fight.

**Disparate-impact regression** - fit the decision on the legitimate credit
factors plus the imputed protected class, ask whether the residual coefficient is
significant.
This is a crude version of the decomposition in notebook 9, run at scale for
decades, and it is a textbook Table 2 fallacy site that nobody in the workflow
names as one.

**Reject inference** - parcelling, augmentation, buying bureau performance on
declined applicants.
The industry solved a version of selective labels commercially long before the
fairness literature named it.
The methods are rough and the motive was accuracy, but it is real.

**LDA search** - search the model space for a variant with less disparate impact
and comparable performance, and document the search.
This is the closest thing in production to the per-feature cost-of-fairness table
this repo argues for.
It is a product category: Zest AI, FairPlay AI and others sell it, and CFPB and
DOJ have pushed on it since roughly 2021.
I have no figure for what share of lenders run it; my impression is a minority,
concentrated in fintech rather than the large banks.

**SPCPs** - ECOA section 1002.8 permits targeted credit programmes for
disadvantaged groups, and the CFPB issued an advisory opinion in 2021
encouraging them.
Note what this is: a *product* remedy, not a model remedy.
The legal system's answer to a real risk difference is a different product, not a
reweighted loss function.

## 4. What is purely academic

- Causal graphs, path-specific effects, counterfactual fairness.
  I know of no lender running Kusner-style counterfactual fairness in a live
  credit decision.
- Explicit fairness constraints at decision time - blocked by the
  disparate-treatment problem above.
- Most of the impossibility literature.
  Correct, interesting, and not what a compliance department argues about.

Two constraints the papers rarely mention:

**Adverse action notices.**
ECOA requires telling a declined applicant the principal reasons.
That is a hard explainability requirement on the primary decision model, and it
is why credit is still dominated by scorecards - logistic regression, WOE
binning, points.
ML enters mostly as a second look on declines, where a rejection can be
overturned but never created.

**The score is upstream.**
Most bank models eat a bureau score as an input.
The population-level disparity in that score is imported wholesale and the bank
never touches it.

## 5. The AI Act collision

**The AI Act does not create a non-discrimination duty on outcomes.**
This is the part most commentary gets wrong.

It is product-safety legislation on the New Legislative Framework: risk
management, data governance, documentation, logging, human oversight,
conformity assessment, CE marking.
Article 10(2)(f)-(g) requires *examination* of datasets for biases "likely to
lead to discrimination prohibited under Union law", and appropriate measures to
detect, prevent and mitigate.
The phrasing points **outward** at existing law.
The Act does not define discrimination and sets no threshold.
There is no four-fifths rule in it, and no numeric bar of any kind.

A provider can pass conformity assessment while shipping a model with a large
gap, provided it examined, documented, and can argue the mitigation was
appropriate.

**The collision inside the Act:** Article 10 says mitigate bias, Article 15 says
achieve appropriate accuracy and declare the metrics, Article 9 says reduce
fundamental-rights risk as far as technically feasible through design.
All three use "appropriate".
There is no ranking anywhere in the text.
The Act does not resolve it - the resolution runs through the underlying
directives instead.

### The actuarial defence: available for indirect, not for direct

This asymmetry is the whole game.

**Direct discrimination - the model uses gender as an input - has essentially no
justification defence in EU law.**
Accuracy is not a defence.
Profitability is not a defence.
(Age is the odd exception; Article 6 of Directive 2000/78 permits justified
direct age discrimination.)

**Indirect discrimination - the model uses income and produces a gender gap -
does have one.**
The test across the directives: objectively justified by a legitimate aim, means
appropriate and *necessary*.
Predictive accuracy can be a legitimate aim.
"Necessary" then means no less discriminatory route to the same aim - the LDA
test again, in European clothes.

So **deleting gender converts direct discrimination into indirect
discrimination, which is defensible.**
Blinding is a technically bad choice made for an entirely rational legal reason.
That is the honest answer to "does removing gender make sense": not
statistically, yes legally.

### Test-Achats is narrow, and it is the case where the defence was removed

C-236/09 (2011) struck Article 5(2) of Directive 2004/113 as incompatible with
Charter Articles 21 and 23; the unisex rule took effect 21 December 2012.
The Court did not dispute that gender predicts mortality and claims frequency.
It treated predictive validity as irrelevant to whether the derogation could
stand.
Insurers argued the actuarial case and lost.

Consequences ran the expected way - young women's motor premiums rose, young
men's fell, and the pooled average rose, because a risk you may not price gets
priced conservatively.
**I do not have a verified figure for the magnitude.**
Treat the direction as solid and the size as unchecked.

This was a statutory derogation being struck, in insurance, on gender.
It did not establish that accuracy never justifies disparity.
In credit the proportionality defence is still live.

### Two enforcement tracks that never meet

| | AI Act | Non-discrimination law |
|---|---|---|
| Who acts | national market surveillance authority | an individual, in national court |
| The wrong | failure to examine, document, mitigate | your model disadvantaged me |
| Penalty | up to EUR 15M or 3% of worldwide turnover (Art 99(4); tiers in section 10) | damages, injunction, reputation |
| Burden | on you to produce the file | shifts to you on a prima facie case |

Article 74(6) designates the **financial supervisor** as market surveillance
authority for credit institutions.
The AI Act therefore arrives through the regulator banks already talk to, not a
new agency - which will matter more for adoption than anything in the
substantive text.

The tracks are independent.
A bank can pass conformity assessment and lose a discrimination claim on the
same model in the same year.

### The awkward result

Non-discrimination law says you may not use race in the decision.
Article 4a says you may hold race to prove you did not.
GDPR minimisation says hold as little as possible.
All three are coherent together, and the outcome is that you must collect a
protected characteristic in order to demonstrate you ignored it.

That is exactly the position US mortgage lenders have occupied since HMDA.
Europe is arriving at the American arrangement through a different door, about
thirty years later.

### Two things worth keeping straight

- **Gender and age are not GDPR Article 9 special categories.**
  Only ethnicity is, of the three attributes this repo tests.
  Article 4a is not what permits retaining gender; an ordinary Article 6 lawful
  basis is.
  This was flagged as item 5 of `fable-review-2026-09-06.md` and the root
  notebook's Section 3 now states it correctly.
- **Article 4a(2) reaches deployers, and Article 10(5) did not.**
  An earlier draft of this file said a deployer running its own monitoring is
  not the addressee.
  That was true of 10(5) and is no longer true under 4a.

### The provision that will actually generate work

**Article 27, the fundamental rights impact assessment.**
It applies to deployers that are public bodies or provide public services - *and*
to deployers of Annex III points 5(b) and 5(c) specifically, which are
creditworthiness assessment and life/health insurance risk pricing, named
outright.
A bank running a credit scoring model is a deployer with a FRIA obligation
regardless of who built the model.
Credit scoring is Annex III 5(b), classified under Article 6(2), so the
obligations bite from **2 December 2027** - pushed back from 2 August 2026 by
Regulation (EU) 2026/1744.

## 6. So what would you actually do

In order, and the first one is not optional:

1. **Fix the label before touching features.**
   Define default precisely.
   Deal with selective labels - reject inference, or better, a randomised
   approval slice below threshold.
   Expensive, and the only thing that yields an unbiased outcome.
2. **Keep the attribute for the audit, out of the model** - where the law lets
   you.
   In the EU that is straightforward for gender and needs Article 4a for race.
   In US non-mortgage credit it is not available and you are on BISG.
3. **Run a per-feature cost-of-fairness table.**
   For each feature carrying disparate impact, fit with and without, record the
   AUC delta and the DIR delta.
   If a feature buys 0.004 AUC and costs 0.17 of DIR you drop it and can defend
   dropping it.
   That table *is* the less-discriminatory-alternative test, and it is auditable.
4. **Draw the graph and say which paths are permitted.**
   gender -> income -> default may be legitimate.
   gender -> officer's judgement -> default is not.
   Statistics will not tell you which is which; that is Kilbertus et al. (2017),
   and notebook 9 is the case where two people who disagree about the graph
   cannot be separated by data.
5. **Do not assume equal approval rates is the target.**
   In lending, forcing parity means approving loans that default, and the cost
   lands on the borrower as damaged credit and repossession.
   Calibration plus a single threshold is usually the defensible choice.
   If unequal approval rates survive that, the remedy is a product remedy - see
   SPCPs above - not a reweighted loss.

## 7. On the "single mother is a worse risk" argument

Part of that difference is probably real.
Part of it is the lender's own footprint: if a group historically received worse
terms, it defaulted more *because of the terms*.
The interest rate is a treatment and default is the outcome, and reading a
treatment effect as a trait is the same error notebook 9 documents in COMPAS,
where the arrest rate measures policing as much as offending.
Same structure, same undecidability from observational data alone.

Even where the difference is real, the question is whether the feature you use is
the risk itself - income volatility, dependants-to-income - or a demographic
category standing in for it.
Substituting the direct measurable risk for the category is usually both fairer
and more accurate at once.

## 8. Is refusing a loan a favour?

Raised against the framing above: default is genuinely bad for the borrower.
Repossession, garnishment, a credit file damaged for years, bankruptcy.
So if you can see someone is likely to default and you decline, are you not
helping them, whatever the reason?

The objection is stronger than it looks, and the law agrees with more of it than
most commentary admits.

### Where it is right

**Responsible-lending rules already mandate exactly this.**
US Dodd-Frank Title XIV and the CFPB ATR/QM rule (2013) make it unlawful to
originate a mortgage without a reasonable good-faith determination that the
borrower can repay.
The EU has the same duty in Mortgage Credit Directive 2014/17 Article 18 and in
the consumer credit creditworthiness obligation.
Both exist because 2008 was caused by doing the opposite.

So the law does not hold that denial is a harm.
It holds that denial *on the wrong basis* is a harm.
Those are different claims and the fairness literature routinely runs them
together.

**The formal version is a published result.**
Liu, Dean, Rolnick, Simchowitz & Hardt (2018), *Delayed Impact of Fair Machine
Learning* (ICML), models the borrower's trajectory after the decision and shows
that a demographic-parity or equal-opportunity constraint can actively harm the
group it was imposed to protect, because over-lending drives scores down.
Corbett-Davies et al. make a related argument in the bundled JMLR paper.

### Where it breaks

**It proves too much.**
If denial is a favour then denying everyone is maximum beneficence, and a lender
that approves nobody is a saint.
The argument needs a threshold, and where the threshold sits is the original
question untouched.
Beneficence relocates the problem; it does not answer it.

**The counterfactual is not "no loan".**
The declined applicant still needs the money.
They go to a payday lender, a doorstep lender, a pawnbroker, or family.
Denial routes people down the credit ladder, not off it, so "I saved them from a
12% loan" can mean "I sent them to a 300% one".
The empirical literature on whether payday access helps or harms is genuinely
contested - Zinman, Melzer and Morse reach different conclusions - so this does
not settle in either direction.
But the comparison is loan versus alternative, never loan versus nothing.

**"You know they will default" is doing enormous work.**
At a 20% predicted default rate you decline 100 people to prevent 20 defaults,
and 80 of them would have repaid.
Precision is worst exactly where it matters: for the marginal applicant at the
cutoff the model is close to a coin flip.
You almost never know someone will default.
You know they sit in a bucket where one in five does.

**The beneficence coincides exactly with self-interest.**
The lender's downside from a default is bounded and already priced into the rate.
The borrower's is not.
The party claiming to decline for your own good is the party that captures the
gain from declining.
That is not proof of bad faith, but it makes the paternalistic justification
unfalsifiable from outside, which is why no legal system accepts it as a defence
on its own.

**Denial compounds.**
Credit access builds credit history.
Decline, thin file, worse score, declined again.
The denial is an input to the next decision, so a model that is accurate at t=0
manufactures by t=1 the disparity it predicted.
This is the mechanism by which redlining outlived the maps.
Denial is not a neutral non-event.

**Default is partly the lender's decision, not the borrower's property.**
Likely to default at what rate?
Offer 8% and they repay; offer 29% and they do not.
The model predicts default conditional on the treatment the lender chose.
Reading that as a trait of the person is the error in section 7 above, and the
same error notebook 9 documents in COMPAS.

### Where the law really is strange

Not quite where the objection puts it, but nearby.

Disparate-impact doctrine can require dropping a feature that genuinely predicts
default.
That makes lending decisions worse, and somebody defaults who otherwise would
not have.
The cost is real, it lands on real people, and the doctrine rarely states it out
loud.
*Test-Achats* did precisely this in insurance: the Court knew the actuarial
difference was real and prohibited its use anyway.

So European law does knowingly accept worse predictions in some places.
It holds that equal treatment is worth that price.
That is a value judgement made in the open.
It can be disagreed with; it is not a confusion.

### What the objection actually exposes

Every fairness metric in this repo - DIR, demographic parity, equal opportunity -
treats **approval rate as if it were welfare**.
It is not.
An approval ending in repossession is not a benefit, and the objection is correct
that the metrics cannot tell the difference.

The conclusion is not "accuracy is fine after all".
It is **measure welfare, not approval rate**.
Almost nobody does, because default is observable in the data and welfare is not.
That is a measurement failure the field has built its whole evaluation apparatus
on top of.

Which lands back where notebook 2 already sits: the label chosen is what does the
damage, not the classifier fitted to it.

## 9. What Fairlearn covers, and why the causal work is not in production

**This section is measured**, unlike the rest of this file.
The API counts below came from `fairlearn 0.14.0` installed under
`python3.12` on this machine on 2026-09-07, by importing the package and
enumerating it.
Notebook 8 already uses this library.

### What is in the box

| Submodule | Public objects | What it does |
|---|---|---|
| `metrics` | 41 | `MetricFrame` plus group differences and ratios |
| `reductions` | 17 | `ExponentiatedGradient`, `GridSearch`, constraint moments |
| `postprocessing` | 2 | `ThresholdOptimizer`, its plot |
| `preprocessing` | 2 | `CorrelationRemover`, `PrototypeRepresentationLearner` |
| `adversarial` | 2 | `AdversarialFairnessClassifier` / `Regressor` |

Grepping the installed package source for causal vocabulary returns nothing:

    causal             0 files
    counterfactual     0 files
    DirectedAcyclic    0 files
    do_operator        0 files
    backdoor           0 files

So Fairlearn covers the **metrics** in section 5 of this file and nothing else in
it.
It measures disparity and it enforces constraints.
It has no view on the label, no selective-labels tooling, no proxy discovery, no
welfare model, no legal mapping, no sensitivity analysis and no reject inference.
`CorrelationRemover` is linear proxy removal, which is the operation notebook 5
shows does not work.

### The signature that matters for lending

    ThresholdOptimizer.predict(self, X, *, sensitive_features, random_state=None)
    ExponentiatedGradient.predict(self, X, random_state=None)
    GridSearch.predict(self, X)

`sensitive_features` is a **required keyword-only argument** on
`ThresholdOptimizer.predict`.
The method cannot be called without the protected attribute at inference time,
because it applies group-specific thresholds.
That is disparate treatment on its face and unusable in a US credit decision.

The reductions are better placed: they need the attribute at `fit` but not at
`predict`.
Whether a decision rule whose parameters were *fitted* using race counts as
treatment is genuinely contested and not settled either way.
Notebook 8 uses `ExponentiatedGradient`, so it sits on the defensible side of
that line, but not by much.

### Why there is no causal module

Structural, not an oversight.
Fairlearn's whole vocabulary is observational - functions of `(y, y_hat, A)`.
Causal fairness needs a **graph**, and a graph is an input the library cannot
supply, cannot validate, and cannot check against the data.
There is no API for "draw the correct DAG".
This is not a gap that gets filled in a later release.

The tooling exists elsewhere and is research code: DoWhy and EconML in the PyWhy
stack, `fairadapt` (Plecko & Meinshausen, R), and `faircause` from Plecko &
Bareinboim's *Causal Fairness Analysis*.
Little of it is production-hardened and much of it is R.

### Why organisations do not use it

Four reasons, and the third is the load-bearing one.

**The graph is a liability in an adversarial proceeding.**
A DAG is an assumption set.
Opposing counsel asks why there is no arrow from X to Y and there is no
data-driven answer.
Notebook 9's finding is exactly that two graphs fit the same data.
In science that is a result.
In a deposition it is a hole.
A regression coefficient with a p-value is defensible not because it is better
but because there are forty years of precedent for arguing about one.

**Sensitivity analysis returns bounds and compliance wants a verdict.**
"Under assumption set A the effect is 0.03, under B it is 0.11" is not a
compliance artefact.

**The law does not pose a causal question.**
Disparate impact asks three things: is there a disparity, is there business
necessity, is there a less discriminatory alternative.
None of the three requires identifying a causal effect.
The doctrine was built on *observable* disparity precisely so that a plaintiff
would not have to prove causation, which they could never do.
Causal inference is therefore a correct answer to a question nobody in the
process is asking.
The tool and the requirement are orthogonal, and that is not institutional
stupidity.

**The data is not there.**
Causal identification needs the confounders measured.
Lenders hold no wealth data, no household structure, no local labour market.
Selective labels removes the outcome for exactly the population that would be
needed.
The machinery runs on data nobody has.

### Inertia, cynicism, or "it works"?

Mostly the third, with real amounts of the second, and very little of the first.
The stats teams have read Pearl.

The thing to see is that **the compliance function is not optimising accuracy or
fairness - it is optimising defensibility**, and it is right to.
A method that is 10% more correct and 50% less defensible is strictly worse for
them.
That is the incentive structure working as designed, not failing.
Nobody is promoted for a better DAG.
People are fired for a consent order.

Some of it is knowingly priced in.
Ally's $98M was not existential against Ally's balance sheet.
Fair-lending exposure is a line item and gets forecast like one.

But the purely cynical reading is incomplete, because LDA search is being adopted
now.
That reveals the real adoption criteria: a method gets in when it is legible to an
examiner, expressible in existing doctrine, and sold by a vendor who absorbs the
model risk.
LDA search meets all three.
Causal fairness meets none.

The uncomfortable version: the methods being adopted are the ones that fit the
legal vocabulary, not the ones most likely to be right.
Those two sets overlap by accident rather than by design.

## 10. What Microsoft actually ships, and what would change any of this

### Fairlearn is not the whole stack, and section 9 should not be read as saying so

Microsoft's responsible-AI tooling also includes InterpretML (glassbox EBMs),
DiCE (counterfactual *explanations*, which are not counterfactual fairness),
Error Analysis, **EconML** for heterogeneous treatment effects, and **DoWhy** -
which came out of Microsoft Research and is the library notebook 9 uses.
The Azure ML responsible-AI dashboard bundles these into one interface.
Fairlearn itself is no longer Microsoft-governed; it moved to independent
community governance.

Confidence: only `fairlearn 0.14.0` and `dowhy 0.14` are installed on this
machine - `econml`, `interpret`, `dice-ml`, `responsibleai`, `raiwidgets` and
`aif360` are all absent, checked 2026-09-07 - so the dashboard's exact
composition here is recall and not measured.

### The finding worth having

**One organisation built both halves and did not connect them.**

Fairlearn returns 0 files for `causal`, `counterfactual`, `backdoor` and
`do_operator` (measured, section 9).
DoWhy and EconML live in a separate stack, PyWhy, now co-maintained with AWS.
The dashboard places them in adjacent tabs, where the causal tab estimates
treatment effects on features and does not do causal fairness.

This is not negligence.
Joining them requires the graph, and the graph is the thing that cannot be
supplied, validated or defended - section 9 again.
The connection is missing inside the one organisation that owns both pieces, for
a structural reason rather than a lazy one.

### Article 99, since section 5 cites it without stating it

Penalties under Regulation (EU) 2024/1689, each cap being whichever is *higher*:

| Breach | Cap |
|---|---|
| Article 5 prohibited practices | EUR 35M or **7%** of worldwide annual turnover |
| Other obligations - providers (Art 16), deployers (Art 26), importers, distributors, notified bodies, Art 50 transparency | EUR 15M or **3%** |
| Incorrect, incomplete or misleading information to authorities | EUR 7.5M or **1%** |

Turnover is total worldwide, not EU revenue.
Article 99(6) inverts the rule for SMEs and start-ups - whichever is *lower* -
so the fixed sum shelters small firms rather than the percentage punishing them.

A credit model lands in the 3% tier.
Article 10 binds providers of high-risk systems, and provider non-compliance is
99(4).
The 7% tier is reserved for the Article 5 prohibitions.
Credit scoring is explicitly not social scoring - Recital 31 says so, and
Annex III 5(b) classifies it as high-risk instead.

Adjacent: Article 100 covers EU institutions (EUR 1.5M / 750k, imposed by the
EDPS) and Article 101 covers general-purpose AI model providers (EUR 15M or 3%,
imposed by the Commission).
The penalty provisions applied from 2 August 2025.
The Annex III high-risk obligations, which is where credit scoring sits, were
pushed back by Regulation (EU) 2026/1744 and now apply from **2 December 2027**
under Article 6(2), and 2 August 2028 under Article 6(1).
Nothing has landed on anyone yet, and on this timetable nothing can for another
fifteen months.

### Why weak assessments are the rational choice right now

**The documentation duty can reward the weaker method.**
Run a strong analysis, find a large disparity, and you have created a document
stating that you found a large disparity - discoverable in litigation and
reportable to a supervisor.
Run a weak analysis, find nothing, and you hold a clean file satisfying the same
obligation.
The incentive gradient points at methods that do not find things.
Nobody needs to be cynical for this to operate; it is enough that the people who
look harder are marginally less likely to be promoted.

**The cost asymmetry sits in the wrong place in the lifecycle.**
A bias finding at design time is cheap - change the label, change the features,
retrain.
A finding after deployment means retrain, revalidate, re-approve, possibly
remediate past decisions and possibly disclose.
So the incentive to look hard falls exactly as the cost of finding something
rises, and governance frameworks concentrate the heaviest assessment at the
pre-deployment gate, the last moment before that step change.

**The ambiguous result is the common case and the worst one.**
A clear finding is actionable.
An ambiguous finding is a decision about how much more to spend investigating,
taken by the person whose budget it is, against a ship date.
Ambiguity resolves toward no action by default.

**Usually nobody decided.**
"Not rocking the boat" implies a choice was made.
More often the assessment goes to a vendor or a junior, on a template, with a
deadline, and there is no moment at which anyone weighed looking harder against
not looking.
The absence of a decision point is how most of this happens.

### The argument against reading all of the above as cynicism

This is what every compliance regime looks like early.
SOX in 2003 was box-ticking.
Bank model validation in the early 1990s was box-ticking.
Both improved, and the mechanism in both cases was enforcement creating real
cost, plus a professional class forming with its own standards and personal
liability.
Fairness governance is at roughly SOX-2003.
That is a timeline, not a defence.

There is already a clean natural experiment.
The CFPB auto-lending wave changed industry behaviour, and nothing in the
tooling changed - Ally paid $98M.
The cost of a weak assessment changed.

So the prediction, stated as a prediction: this remains theatre until a penalty
lands on someone identifiable, and then it stops being theatre fairly quickly.
Better libraries will not do it.
Article 99 might.

## Sources named above, not all verified in session

Bundled in `../papers/`: Dwork et al. (2012); Corbett-Davies et al. (JMLR 2023);
Kusner et al. (2017); Kilbertus et al. (2017).
Cited not bundled: Lakkaraju et al. (2017); Kamiran & Calders (2012);
Liu, Dean, Rolnick, Simchowitz & Hardt (2018), *Delayed Impact of Fair Machine
Learning* (ICML) - <https://arxiv.org/abs/1803.04383>.
Named without a specific citation in section 8: Zinman, Melzer and Morse on
payday-lending access, who disagree with each other.
Named in section 9: Plecko & Meinshausen, `fairadapt` (R); Plecko & Bareinboim,
*Causal Fairness Analysis* and `faircause`; the PyWhy stack (DoWhy, EconML).

Software enumerated in sections 9 and 10: `fairlearn 0.14.0`,
`scikit-learn 1.7.1`, `dowhy 0.14`, `python3.12`, 2026-09-07.
Absent on this machine and therefore not checked: `econml`, `interpret`,
`dice-ml`, `responsibleai`, `raiwidgets`, `aif360`.

Section 10 adds: Regulation (EU) 2024/1689 Articles 99, 100, 101 and Recital 31;
Microsoft InterpretML, DiCE, Error Analysis, EconML, DoWhy (PyWhy).

Legal instruments referenced, none read in this session:
Regulation (EU) 2024/1689 Articles 4a, 9, 10, 11, 15, 27, 43, 74, 85, 86, 99,
Annex III 5(b), as amended by Regulation (EU) 2026/1744 (Digital Omnibus on AI,
in force 27 July 2026), which deleted Article 10(5) and inserted Article 4a;
Directives 2000/43/EC, 2000/78/EC, 2004/113/EC;
CJEU C-236/09 *Test-Achats*;
US ECOA / Regulation B, HMDA / Regulation C, ECOA section 1002.8;
CFPB BISG methodology (2014) and SPCP advisory opinion (2021);
US Dodd-Frank Title XIV and the CFPB ATR/QM rule (2013);
Directive 2014/17/EU Article 18 and the consumer credit creditworthiness duty.
