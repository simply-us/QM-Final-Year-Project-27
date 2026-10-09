# MSc AI Dissertation — Proposal Notes
**Queen Mary University of London | Part-time MSc AI, Year 2**
**Working title area:** Verifying whether LLM clinical reasoning is faithful to, and consistent with, established NHS clinical guidelines

---

## 1. Background / Motivation

- Full-time Tech Lead at UBS (sponsoring the MSc); pursuing the degree to move meaningfully into AI, not for career necessity.
- Core interests considered: health, alignment/verification of AI intent (new interest), Africa, entrepreneurship, information retrieval (work-related, lower personal impact priority).
- Decision driver: **real-world impact** over technical novelty. Wanted applied work with genuine relevance, not an abstract benchmark exercise.
- Landed on combining **health** (existing familiarity, clearer access) with **alignment/verification** (new interest) — auditing whether AI reasoning in a clinical context can be trusted, rather than building yet another predictive model.

---

## 2. Why Not Just Use Accuracy?

Key reasoning established early in scoping:

- High accuracy (e.g. 99%) tells you a model is right *often*, not right for the *right reasons*, and not reliably right on rare-but-critical cases.
- **Shortcut learning risk**: models can achieve strong accuracy by keying off spurious correlates (e.g. hospital/scanner artifacts, ward assigned) rather than true clinical signal — this fails silently when deployed elsewhere.
- **Rare-event problem**: aggregate accuracy can look excellent while performance on the small subset that matters most (e.g. true sepsis cases) is poor.
- **Distribution shift**: accuracy is a snapshot on one dataset; guideline conformance is a check against a standard that should hold everywhere.
- **Accountability**: "99% accurate" is not a defensible answer to a clinician, regulator, or coroner. "We verified its logic against NICE/NEWS2 and know exactly where it diverges" is.

This is the core motivating argument for the dissertation: **verification of reasoning, not just outcome accuracy, is the missing layer for trustworthy clinical AI.**

---

## 3. The "Unit Test" Analogy (and where it breaks)

Personal framing (software engineering background) that shaped the methodology:

- **Where it holds:** guideline-conformance testing is essentially black-box behavioral testing — like unit tests, but derived from a clinical spec (NEWS2, QRISK3, NICE guidance) instead of a code spec. E.g.: *"given worsening oxygen saturation, risk score must not decrease."* This is close to established **behavioral/invariance testing** approaches (e.g. CheckList-style testing, applied to ML).
- **Where it breaks:** unlike `add(2,3) == 5`, clinical "correctness" is rarely a single deterministic value — guidelines themselves are simplifications, and acceptable answers form a range. So this becomes closer to **property-based testing** ("for all inputs satisfying X, invariant Y must hold") than exact-value unit testing.
- **The extra 20%:** passing behavioral tests doesn't guarantee the model isn't right for a spurious reason (mirrors the shortcut-learning problem). This motivates adding an **interpretability/faithfulness layer** on top of pure behavioral testing — agreed as ~80% behavioral testing, ~20% interpretability/faithfulness.

---

## 4. Data & Access Landscape (UK-focused)

Decision: stick to UK/NHS context rather than US-centric datasets (e.g. MIMIC), for coherence between the "guideline" and the "system" (NHS guideline audited against NHS-relevant context).

Real patient-level NHS data (CPRD, HES, full Trust data) requires ethics approval / data-sharing agreements — generally not feasible on a part-time, one-year dissertation timeline without an existing insider route.

Relevant UK-accessible options identified:
- **Simulacrum** — synthetic cancer registry data (NHS England / PHE), free, no re-identification risk.
- **SynAE** — synthetic A&E extract, NHS England pilot, built from real A&E attendance patterns.
- **CPRD synthetic datasets** — synthetic primary care data, usable for ML training/evaluation.
- **NHS Digital / HES open/aggregate datasets.**

**Key pivot:** decided *not* to train a bespoke model on this data at all. Reasoning: if the dissertation trains its own predictive model, any weird verification result is ambiguous — is it a genuine finding about AI reasoning, or just a mediocre model implementation? Far stronger to **audit an existing model you didn't build** — either a published clinical ML model, or a general-purpose LLM (GPT/Claude-class) already being informally used for clinical reasoning.

This also removes the patient-data access bottleneck: the LLM-auditing approach needs **well-constructed clinical vignettes grounded in guideline documents**, not large real/synthetic patient-record datasets.

---

## 5. Candidate Ground-Truth Guidelines (UK)

- **NEWS2** (National Early Warning Score 2) — NHS hospital/A&E deterioration scoring system based on vitals (RR, SpO2, temperature, BP, HR, consciousness). Fully public, well-documented, trusted operational standard.
- **QRISK3** — NHS-endorsed 10-year cardiovascular risk calculator used in primary care/GP settings, transparent public formula.

Both offer a clean, publicly documented "ground truth" logic to test AI reasoning against — chosen instead of building/using a fuzzier standard.

---

## 6. Current Shape of the Project

**Core question:** Does an LLM's clinical reasoning behave consistently with, and faithfully to, an established NHS clinical guideline (NEWS2 or QRISK3) — or does it diverge in ways that would be invisible to standard accuracy metrics?

**Method, in two layers:**

1. **Behavioral / invariance testing (~80%)**
   - Construct a suite of clinical vignettes derived from NEWS2/QRISK3 logic.
   - Test properties such as monotonicity (e.g. worsening vitals → risk should not decrease), consistency under clinically-irrelevant rephrasing, and sensitivity to guideline-relevant variables.
   - Run against a target LLM (or published clinical model), report violation rate, severity, and patterns (e.g. does it fail more on edge cases or specific subgroups).

2. **Faithfulness / interpretability layer (~20%)**
   - For cases where the model states a rationale ("I recommend X because Y"), apply **counterfactual intervention**: alter/remove the cited factor (Y) and check whether the output changes as it should if the stated reasoning were causally true.
   - If removing the cited justification doesn't change the output, the explanation is likely post-hoc rather than the true driver — a direct, implementable analogue to current chain-of-thought faithfulness research, requiring no deep interpretability tooling (no activation patching required).

**Target system:** leaning toward auditing a general-purpose LLM (e.g. GPT/Claude-class) used for clinical reasoning over vignettes, rather than a bespoke tabular ML model — this fits the alignment framing more directly and avoids the patient-data access bottleneck.

---

## 7. Literature Check (Aug 2026) — Is This Novel Enough?

Confirmed this is an **active, current research area** (multiple 2025–2026 papers), but with a clear gap matching this project's angle:

- Existing work evaluates faithfulness of closed-source LLMs (ChatGPT, Gemini) in medical reasoning using causal ablation, positional bias, and hint-injection probes — generically, on medical QA / RCT-style evidence.
- MedCounterFact benchmark tests LLM behavior under counterfactual/adversarial medical evidence; finds models often accept fabricated evidence at face value.
- Other work perturbs demographic details (e.g. pronouns) while holding clinical facts constant, finding reasoning shifts (cited risk factors, guideline references) even when final diagnoses stay stable.
- Recent work shows LLMs can be diagnostically accurate yet structurally inconsistent across similar cases.

**The gap / contribution:** none of the existing work anchors its tests to a **specific, real, operationally-used clinical guideline** (like NEWS2 or QRISK3). Existing benchmarks test against generic "medical reasoning" or RCT evidence, not a codified national standard actually used in NHS practice. Testing against a real, deployed NHS scoring system is more specific, more actionable for a real health system, and appears to be unoccupied ground.

**Scoping assessment:** the *methodology* (behavioral/invariance testing + counterfactual faithfulness probing) is well-established and validated in current literature — appropriate to reuse rather than invent from scratch. The novelty is in **specificity of application** (a real, national, currently-used clinical standard) rather than inventing a new method — this is the right level of ambition for a part-time master's dissertation: original contribution, bounded scope, defensible timeline.

---

## 8. Open Decisions / Next Steps

- [ ] Finalise choice between NEWS2 (hospital/urgent care) vs QRISK3 (primary care/preventive) as the anchor guideline — or scope for both if time allows.
- [ ] Decide exact target system to audit: general-purpose LLM (GPT-4/Claude-class) vs a published clinical ML model vs both.
- [ ] Design the vignette suite: source guideline documents (NEWS2 spec, QRISK3 formula/NICE docs), define invariant properties to test.
- [ ] Define faithfulness testing protocol (counterfactual intervention on stated rationale).
- [ ] Draft formal one-paragraph research proposal (title, question, method, contribution) for supervisor discussion.
- [ ] Confirm ethics approval requirements — likely minimal/light since no real patient data is used (synthetic vignettes only), but confirm with QMUL ethics process.
- [ ] Position literature review explicitly against 2025–2026 faithfulness papers (MedCounterFact, closed-source LLM faithfulness study, pronoun-perturbation study, Clinical Reasoning Graphs) to state the contribution clearly: guideline-specific faithfulness testing vs generic medical-QA faithfulness testing.

---

## 9. Key Sources Referenced

- *Faithful or Just Plausible? Evaluating the Faithfulness of Closed-Source LLMs in Medical Reasoning* — arXiv:2603.13988
- *Faithfulness vs. Safety: Evaluating LLM Behavior Under Counterfactual Medical Evidence* (MedCounterFact) — arXiv:2601.11886
- *MEDEQUALQA: Evaluating Biases in LLMs with Counterfactual Reasoning* — ACL Anthology, 2025
- *Clinical Reasoning Graphs: Structured Evaluation of LLM Diagnostic Reasoning Reveals Competence Without Consistency* — arXiv:2606.29876
- *Training and Evaluation of Guideline-Based Medical Reasoning in LLMs* — arXiv:2512.03838
- NHS England DART — Synthetic Data in Health Overview (Simulacrum, SynAE, CPRD synthetic datasets)
- UCL Centre for Advanced Research Computing — "Reimagining How We Use NHS Data" (synthetic data governance layers)
