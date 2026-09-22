# Evaluation Reliability in Multimodal Machine Unlearning
### Benchmark Conditioning, Reference Consistency, and Knowledge Recoverability

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue.svg)](https://www.python.org/)

## TL;DR

How reliable are the criteria used to evaluate multimodal machine unlearning?

This repository accompanies our study of **evaluation reliability in multimodal machine unlearning**. Rather than proposing another unlearning algorithm, we audit how conclusions about an edited model depend on:

- the **benchmark operating regime**;
- the **evaluation criterion**;
- the **definition of the counterfactual reference**; and
- the **probe family used to test residual knowledge**.

Our experiments show that conclusions can change substantially across these choices.

Across three VQA benchmarks, pooled forgetting–retention rankings show a strong negative association:

\[
\tau_b=-0.869.
\]

When the benchmark is held fixed to MMUBench, the association becomes positive:

\[
\tau_b=+0.421,\qquad p=0.1184.
\]

Within MMUBench, different criteria can also prefer different unlearning methods.

Reference-based diagnostics show substantial agreement across representation spaces: activation distance (AD) and Jensen–Shannon divergence (JS) have strong method-level concordance,

\[
\tau_b=0.867,\qquad p_{\mathrm{exact}}=0.0167,
\]

but this agreement is **not external validation**, because the reference-based quantities share the same retain-only counterfactual anchor.

Finally, low direct forget accuracy does not establish semantic erasure. In our bounded Knowledge Recoverability pilot, residual target information was detected in **23 of 24** directly suppressed cases.

The main conclusion is therefore:

> **Evaluation conclusions are conditioned on the benchmark, metric, reference definition, and probe family.**

---

## Evaluation Framework

We evaluate an edited multimodal model along several distinct dimensions.

| Criterion | Preferred direction | What it measures | What it does **not** establish |
|---|---:|---|---|
| **FA** | ↓ | Direct target-answer accuracy | Semantic erasure |
| **RA** | ↑ | Retained-task utility | Successful forgetting |
| **MIA<sub>adv</sub>** | ↓ | Membership distinguishability under the evaluated attack | General privacy |
| **AD** | ↓ | Activation-space proximity to a retain-only reference | Semantic erasure |
| **JS** | ↓ | Predictive-distribution proximity to the same reference | Semantic erasure |
| **d<sub>ΔW</sub>** | ↓ | Effective LoRA-update proximity to the same reference | Functional equivalence |
| **KR** | ↓ | Recoverability under the evaluated probe family | Exhaustive worst-case leakage |

No individual criterion is treated as an erasure certificate.

---

## Experimental Design

Two analysis panels are deliberately kept separate.

### Cross-benchmark panel

**4 methods × 3 benchmarks × 3 unlearning seeds = 36 checkpoints**

Methods:

- Gradient Ascent (GA)
- Random Labels (RL)
- FT-Retain
- SalUn

Benchmarks:

- MLLMU-Bench
- UnLOK-VQA
- MMUBench

This panel is used to study **benchmark conditioning and pooling effects**.

### Common-MMUBench panel

**6 methods × 3 unlearning seeds = 18 checkpoints**

Methods:

- Gradient Ascent
- Random Labels
- FT-Retain
- SalUn
- NegGrad+
- SCRUB

This panel is used for:

- common-benchmark method comparison;
- metric-dependent method selection;
- reference-consistency analysis; and
- descriptive equal-weight rank profiles.

The two panels are not pooled because their method coverage differs.

---

## Finding 1 — Benchmark Operating Regime Matters

The three evaluated benchmarks occupy substantially different operating regimes under our configuration.

Pooling them produces a strong apparent negative relationship between forgetting and retention:

\[
\tau_b(\mathrm{FA},\mathrm{RA})=-0.869.
\]

Within MMUBench alone:

\[
\tau_b(\mathrm{FA},\mathrm{RA})=+0.421,\qquad p=0.1184.
\]

The result therefore should not be interpreted as an intrinsic universal forgetting–retention relationship.

Instead, it shows that **benchmark composition can strongly affect pooled evaluation conclusions**.

This is why the repository reports benchmark-specific results before pooled analyses.

---

## Finding 2 — Different Criteria Can Prefer Different Methods

Holding the benchmark fixed does not eliminate evaluation disagreement.

Within the six-method MMUBench panel, different criteria emphasize different properties of the edited model.

For example:

- Gradient Ascent shows strong AD and JS alignment but weak direct forgetting under FA.
- Random Labels performs strongly under FA and RA but poorly under membership distinguishability.
- NegGrad+ and SCRUB have comparatively consistent equal-weight rank profiles.

Mean rank and rank dispersion are reported only as **descriptive summaries**.

They do not define a universal method ordering because an equal-weight rank average is not a deployment utility function.

---

## Finding 3 — Reference-Based Diagnostics Show Cross-Space Consistency

We compare edited models with the same retain-only counterfactual reference \(M^*\).

The reference was not trained on the forget examples.

Reference proximity is examined in three spaces:

1. **activation space** — AD;
2. **predictive-distribution space** — JS;
3. **effective adapter-update space** — \(d_{\Delta W}\).

Across the 18 common-MMUBench checkpoints, association with effective-update proximity is strongest for:

- JS: \(\rho=+0.874\)
- AD: \(\rho=+0.787\)

At the method level, AD and JS show:

\[
\tau_b=0.867,\qquad p_{\mathrm{exact}}=0.0167.
\]

This is evidence of **cross-space reference consistency**, not external ground truth.

All three quantities are conditioned on the same retain-only reference.

---

## Finding 4 — Direct Suppression Does Not Establish Semantic Erasure

A model can fail to produce the target answer under the original query while still retaining information that supports recovery under another probe.

We therefore introduce **Knowledge Recoverability (KR)** as a separate audit axis.

KR is evaluated only for examples satisfying direct:

\[
\mathrm{FA}=0.
\]

### Explicit probes

Explicit evidence requires reconstruction of the target information, including:

- semantic paraphrases;
- completion prompts; and
- indirect questions requiring the forgotten fact.

### Implicit probes

Implicit probes test whether target information remains discriminatively accessible without requiring free reconstruction.

Examples include negation and candidate-discrimination tests.

### Pilot result

| Method | Directly suppressed cases | Any KR evidence |
|---|---:|---:|
| Gradient Ascent | 6 | 5/6 (83%) |
| SalUn | 18 | 18/18 (100%) |
| **Combined** | **24** | **23/24 (96%)** |

The **23/24** result is a bounded pilot over a finite, non-adaptive probe family.

It is **not** an estimate of worst-case adversarial recovery probability.

The result supports the narrower conclusion:

> **Low direct forget accuracy alone does not establish that target information is unrecoverable.**

---

## Identifiable LoRA Reference Distance

Raw LoRA factors are not uniquely identifiable.

For a LoRA update,

\[
\Delta W=\frac{\alpha}{r}BA,
\]

an invertible transformation \(S\) gives

\[
(BS^{-1})(SA)=BA.
\]

The raw factors \(A\) and \(B\) can therefore change while representing the same effective update.

For reference-based parameter analysis, we consequently compare **effective updates** rather than raw LoRA factors:

\[
d_{\Delta W}(\widehat{M},M^*)
=
\frac{1}{L}
\sum_{\ell=1}^{L}
\frac{
\left\|
\Delta W_{\ell}^{(\widehat{M})}
-
\Delta W_{\ell}^{(M^*)}
\right\|_F
}{
\left\|
\Delta W_{\ell}^{(M^*)}
\right\|_F+\epsilon
}.
\]

This removes the raw-factor reparameterization ambiguity.

It should still be interpreted as an **effective adapter-update diagnostic**, not as a unique functional distance between neural networks.

---

## Membership-Distinguishability Correction

The final analysis does **not** use the original forget-set-only non-member success rate as a lower-is-better privacy metric.

For each checkpoint, we first compute balanced attack accuracy:

\[
\mathrm{BA}=
\frac{\mathrm{TPR}+\mathrm{TNR}}{2},
\]

then define:

\[
\mathrm{MIA}_{\mathrm{adv}}
=
2|\mathrm{BA}-0.5|.
\]

Therefore:

- \(\mathrm{MIA}_{\mathrm{adv}}=0\) means the evaluated attack performs at chance;
- larger values indicate greater membership distinguishability.

The transformation is applied **checkpoint-by-checkpoint before seed averaging**.

This is an attack-specific diagnostic and should not be interpreted as a general privacy guarantee.

---

## Optional Scalarization

The primary evidence in this repository is the **metric vector**, not a single composite score.

A scalar ordering requires a deployment-specific utility function.

For sensitivity analysis, the supplementary material retains an illustrative Unified Quality Score (UQS):

\[
\mathrm{UQS}
=
0.0745(1-\mathrm{FA})
+
0.1719\,\mathrm{RA}
+
0.1781(1-\mathrm{MIA}_{\mathrm{adv}})
+
0.2727e^{-\mathrm{AD}/100}
+
0.3028(1-\mathrm{JS}).
\]

The weights are data- and reference-dependent.

They are **not statistically optimal, externally validated, or universal**.

UQS is therefore provided only as a sensitivity example.

> **No single scalar is treated as an erasure or privacy certificate.**

---

## Secondary Architecture Check

A limited qualitative check is also provided using BLIP-2 OPT-2.7B on MMUBench at seed 42.

For the two evaluated methods:

- GA: FA = 0.667, RA = 0.600, AD = 1.87
- FT-Retain: FA = 0.667, RA = 0.630, AD = 44.84

Because this check contains only **two methods and one seed**, it is not presented as a cross-architecture replication.

The main quantitative conclusions of the paper are based on the LLaVA-1.5-7B experiments.

---

## Statistical Conventions

All rank analyses are preference aligned:

- lower is better for FA, MIA<sub>adv</sub>, AD, and JS;
- higher is better for RA;
- rank 1 always means better performance.

Ties use midranks.

The three unlearning seeds share the same seed-42 base model. They therefore characterize variability in the unlearning procedure rather than independent full-training replications.

For the cross-benchmark analysis, the 36 checkpoints form **12 method × benchmark blocks**.

For the common-MMUBench reference analysis, the 18 checkpoints form **six method blocks**, each containing three seeds.

Dependence-aware block/cluster procedures are used where appropriate.

---

## Quickstart

### Clone the repository

```bash
git clone https://github.com/abdullahak07/UnifiedUnl.git
cd UnifiedUnl
pip install -r requirements.txt
```

### Quick analysis check

```bash
python benchmark/run_benchmark.py --quick
```

This mode is intended for quickly checking the analysis pipeline and representative outputs.

### Analyse pre-computed results

```bash
python benchmark/run_benchmark.py --results-only
```

### Full experimental pipeline

```bash
python download_models.py
python main.py --stage all
```

See [`REPRODUCE.md`](REPRODUCE.md) for the detailed reproduction procedure and hardware requirements.

---

## Repository Structure

```text
UnifiedUnl/
├── main.py
├── config.py
├── download_models.py
├── run_blip2_minimal.py
├── kr_pilot.py
│
├── models/
├── data/
├── unlearning/
├── evaluation/
├── analysis/
├── benchmark/
├── scripts/
│
├── outputs/
│   ├── multimodal_results.json
│   ├── unimodal_results.json
│   ├── uqs_weights.json
│   ├── blip2_minimal_summary.json
│   ├── kr_pilot_results.json
│   ├── analysis_results.json
│   ├── figures/
│   └── tables/
│
├── REPRODUCE.md
├── DATASHEET.md
├── CITATION.cff
└── croissant_metadata.json
```

---

## Recommended Reporting Protocol

Based on the evaluation audit, multimodal-unlearning studies should report:

1. pre-unlearning target exposure or memorization;
2. benchmark-specific metric vectors before pooling;
3. separate analysis panels when method coverage differs;
4. preference-aligned metric directions and tie conventions;
5. rank summaries only alongside the underlying metric values;
6. dependence among seeds/checkpoints and appropriate uncertainty estimates;
7. the exact construction of any counterfactual reference;
8. identifiable parameter representations when parameter-space distance is used;
9. explicit and implicit recoverability probes after direct suppression; and
10. the utility, normalization, weights, and sensitivity analysis of any scalarization.

---

## Scope and Limitations

The quantitative experiments use LLaVA-1.5-7B with LoRA.

The three unlearning seeds share one seed-42 base model and therefore are not independent full-training replications.

MLLMU-Bench and UnLOK-VQA occupy compressed operating regimes under the evaluated direct-answer configuration. This observation is configuration-specific and does not imply that either benchmark is intrinsically defective.

The retain-only \(M^*\) is a counterfactual reference rather than a unique ground-truth retraining solution.

Effective-update distance removes raw LoRA-factor non-identifiability but is not a unique functional distance over the data manifold.

The KR experiment uses a finite probe family and is not an exhaustive adversarial recovery evaluation.

MIA<sub>adv</sub> describes the evaluated membership attack and is not a general privacy guarantee.

The BLIP-2 experiment is qualitative and does not support architecture-level generalization.

---

## Paper

**Evaluation Reliability in Multimodal Machine Unlearning: Benchmark Conditioning, Reference Consistency, and Knowledge Recoverability**

Abdullah Ahmad Khan, Hamid Laga, and Ferdous Sohel.

If you use this repository, please cite the accompanying paper. The final venue-specific BibTeX entry should be added here once bibliographic details are available.

---

## License

Apache 2.0. See [`LICENSE`](LICENSE).
