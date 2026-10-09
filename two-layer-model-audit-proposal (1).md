# Two-Layer Auditing of Reasoning Faithfulness in Financial Models

## Overview

This proposal outlines a methodology for auditing whether AI/ML models used in trading, pricing, and risk management are not just *accurate* but *trustworthy in their reasoning* — i.e., whether their outputs respect known domain rules, and whether any stated rationale for a decision reflects what actually drove that decision.

The methodology is deliberately applied to a model the researcher did not build. Auditing one's own model makes any anomalous result ambiguous between "genuine finding" and "artifact of a flawed implementation." Auditing an independent, existing model removes that ambiguity and produces a result that generalizes.

The audit has two layers, applied sequentially.

## Layer 1: Behavioral / Invariance Testing

**Idea:** Derive test cases from a real, authoritative, domain-specific standard, and check whether the model's outputs respect the known logical properties of that standard — unit tests written against a domain spec, rather than against code.

In finance, several model classes have closed, published, mechanically checkable specs that support this kind of testing:

- **Derivatives pricing:** no-arbitrage constraints — call prices monotonically decreasing in strike, non-negative butterfly spreads, put-call parity, monotonicity in time-to-maturity for American options, no calendar or butterfly arbitrage in the implied volatility surface.
- **Credit risk / scoring:** monotonicity properties such as "probability of default should not decrease as leverage increases, all else held constant," or "loss-given-default should not improve as collateral quality worsens." Fair-lending regulation (e.g., adverse-action requirements) effectively mandates a version of this already.
- **Regulatory capital formulas** (e.g., FRTB, IRB): published, closed-form, authoritative — internal model sensitivities can be tested against what the regulatory formula implies.
- **Risk measure properties:** subadditivity of Expected Shortfall (note: VaR notoriously fails this — a useful illustration that even "authoritative" specs can have known pathologies, which the audit should be able to surface rather than assume away).

The general test pattern: *"If input X gets worse, holding everything else constant, the model's risk/price/output should not improve."* Violations are damning even when overall accuracy is high, because they reveal the model isn't actually implementing the relationship it's assumed to implement — it's approximating it well enough on average, which is a weaker and more fragile property than genuine conformance.

**Important boundary:** this layer only works for model classes with an external, closed spec. Alpha/forecasting models generally have no such spec — there is no authoritative "correct" sensitivity of a momentum signal to volume the way there is a no-arbitrage rule for option prices. The methodology should be scoped to pricing, credit, and regulatory-capital model classes first, where the invariance layer has something to bite on.

**Non-stationarity caveat:** unlike a static domain rulebook, correct sensitivities in markets can be legitimately regime-dependent (correlation breakdown under stress, liquidity-adjusted VaR behaving differently in crises). Invariance tests should be regime-scoped rather than global, or a real regime-conditional relationship will be misflagged as a violation.

## Layer 2: Faithfulness / Interpretability Testing

**Idea:** For any case where the model gives a stated rationale for its decision, apply counterfactual intervention — alter or remove the specific factor it claims justified the output, then check whether the output actually changes as it should if that reasoning were genuine. If the output doesn't change, the stated explanation is likely post-hoc rationalization rather than the true driver of the decision: a red flag that the model may be right for the wrong reasons — a failure mode that typically only surfaces later, in cases the original testing didn't anticipate.

This layer is largely spec-independent and portable across model types: gradient-boosted models with SHAP/LIME explanations, LLM-generated credit memos or trade rationales, attention/attribution-based sequence models. It directly targets a known, documented weakness of current explainability tooling in finance — post-hoc attribution methods (SHAP in particular) are widely used for regulatory "explainability" in credit and fraud models, but are known to sometimes disagree with the model's true internal decision process, and different faithfulness metrics frequently disagree with each other.

## Why Accuracy Alone Is Insufficient

A model can be highly accurate in aggregate while still failing this audit, for several distinct reasons:

1. **Aggregate accuracy hides where the errors are.** A model that is very accurate on routine cases but unreliable on the tail — precisely the high-severity, low-frequency cases where being wrong is most costly — can still post a strong overall accuracy number.
2. **It doesn't reveal *why* the model is right.** A model can match its target 90% of the time either by tracking a genuine causal driver or by tracking a spurious correlate that happened to align with the label historically. Both look identical in an accuracy metric. The difference only shows up when the correlation breaks — a new regime, a new population, an input combination outside the training distribution.
3. **Invariance violations are damning independent of accuracy**, because they show the model's internal logic contradicts a rule known to hold, even on cases it labels correctly.
4. **Regulatory and legal exposure runs on individual cases, not aggregate accuracy.** In credit specifically, adverse-action requirements demand that a stated reason for a decision be genuine, not merely that the model is usually correct.
5. **Ground truth itself can be compromised.** Labels are often outcomes shaped by the model's own prior decisions (e.g., only approved loans generate repayment labels — classic selection bias in credit scoring) or are noisy proxies for what actually matters. Layer 1 and Layer 2 check internal logical consistency against an external spec and against the model's own stated reasoning, respectively — neither depends on the label being clean.

## Positioning Relative to Existing Research

There is active, recent work adjacent to this proposal, and the gap between two existing literatures is where this methodology sits.

**LLM trading-agent benchmarks** (e.g., StockBench, PortBench, LiveTradeBench, DeepFund, and related 2025–2026 work) evaluate agents primarily on outcome metrics — cumulative return, maximum drawdown, Sortino ratio, relative to a buy-and-hold baseline. This is outcome-based evaluation, structurally the same limitation as "accuracy alone" above, just reproduced at the level of agent P&L rather than classification accuracy.

A smaller thread within this literature has begun probing *why* agents make decisions, and is closer in spirit to this proposal's Layer 2. One recent benchmark found that an LLM trading agent's stated rationale changed entirely depending on whether it could see the actual asset ticker versus anonymized price data alone — producing a confident buy recommendation with fabricated-sounding justification when the ticker was visible, and refusing to trade at all on the identical numeric series when the ticker was anonymized. This is, in effect, an ad hoc counterfactual-intervention test that surfaced exactly the failure mode Layer 2 is designed to catch systematically. Related work tracks whether trading agents are leaning on memorized pretraining knowledge of specific tickers and dates rather than genuine reasoning over the data provided, and other work attributes portfolio-management failures to specific stages of a multi-step reasoning pipeline rather than scoring only the final outcome.

**General chain-of-thought faithfulness research** (from Anthropic, academic groups, and others) is an active area studying whether a model's stated reasoning is the true causal driver of its output, using counterfactual and unlearning-based interventions — the general-purpose version of this proposal's Layer 2. This work is not finance-specific and is not paired with domain-spec invariance testing.

**Explainability-faithfulness research in credit and fraud models** documents that post-hoc attribution methods (SHAP, LIME) frequently diverge from a model's true decision behavior, and that different faithfulness metrics disagree with one another — motivating exactly the kind of direct counterfactual check this proposal's Layer 2 performs, rather than relying on attribution methods alone.

**The gap:** no existing work combines a systematic, domain-axiom-derived invariance test suite (Layer 1) with counterfactual faithfulness testing of stated rationale (Layer 2) as a single audit protocol applied to a financial model. Trading-agent benchmarks test profitability or memorization; CoT-faithfulness research tests general reasoning tasks in isolation from any authoritative external spec; credit/fraud XAI research tests faithfulness of attribution methods without a parallel invariance layer. This proposal's contribution is not a novel method at either layer individually, but the combination, applied rigorously to a model class with a genuine external spec.

## Proposed Scope

- **Model class:** a pricing/greeks model or a credit-scoring model, chosen specifically because each has a closed, authoritative, mechanically checkable rule set (no-arbitrage constraints; monotonicity and fair-lending constraints, respectively).
- **Subject model:** an existing, independently built model — not one built by the researcher — to preserve the interpretive clarity of the audit.
- **Layer 1 deliverable:** a derived test suite of invariance properties from the relevant domain spec, run against the subject model, with a violation rate reported per property.
- **Layer 2 deliverable:** for cases where the model produces a stated rationale, a counterfactual intervention protocol (altering or removing the claimed causal factor) with a measured rate of "explanation does not match behavior" cases.
- **Framing for outcome:** positioned explicitly as a pre-deployment verification gate — the kind of check a quant or risk team would run before a model is granted trading or lending authority — rather than as a standalone audit function, since this is closer to how systematic/quant funds and credit-risk teams already structure model validation workflows.

## Anticipated Objection and Response

*"We already backtest and have risk limits — why do we need this?"*

Backtest accuracy and risk limits do not catch axiom violations or post-hoc-rationalized reasoning, and these are exactly the failure modes that surface later, outside the backtest window, when they are most expensive to discover — the ticker-anonymization finding above is a concrete, already-observed example of this happening in a live evaluated system.
