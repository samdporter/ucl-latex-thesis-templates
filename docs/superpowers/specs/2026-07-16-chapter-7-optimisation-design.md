# Chapter 7 optimisation narrative design

## Goal

Rewrite Chapter 7 as a cohesive optimisation study that motivates and records the practical choices used for synergistic PET--SPECT reconstruction, while leading honestly into the Chapter 8 application. The account must distinguish empirical observations from explanations and must not retrofit the Chapter 8 configuration as the optimum of experiments performed later.

## Narrative structure

1. **Multi-bed reconstruction motivation.** Open with the qualitative dual-bed PET result from the Physics in Writing work. Show that reconstructing bed positions separately and fusing them produces conspicuous noise or discontinuities in the overlap, whereas joint reconstruction with a CT-guided edge-preserving prior suppresses these artefacts. Treat this as qualitative evidence only.
2. **Evaluation protocol.** Introduce the single-bed anthropomorphic phantom scanned on the Mediso AnyScan at NPL. Use 18 PET and 18 SPECT subsets, five stochastic runs per tested setting, and the first 20 data passes. Compare each iterate with the final long-run baseline reconstruction using whole-image, hot-sphere, and cold-sphere NRMSE. Plot the pointwise run mean with minimum--maximum envelopes.
3. **Subset organisation.** Compare only paired PET--SPECT subsets with modality-separated subsets. Sampling is with replacement and the complete prior remains outside the sampler, so its full gradient is included in every stochastic update. One data pass contains the same 36 modality-specific data-subset evaluations in both schemes, but these are arranged as 18 joint updates or 36 modality-specific updates; the latter also evaluates the prior 36 rather than 18 times. Use only the MM-diagonal-tight PET and SPECT figures. Report the faster convergence of separate subsets and present the greater number of updates per data pass as a plausible explanation, not an isolated causal result or a compute-matched comparison.
4. **Prior-aware preconditioning.** Compare the available MM, Lewis--Sendov block-diagonal, and BSREM data preconditioners. Establish that data-only BSREM can diverge for the count-limited PET reconstruction, whereas the SPECT reconstruction is less sensitive and can perform competitively with BSREM. The prior-aware methods behave broadly similarly, with no universal winner. Use the step-size comparison to motivate modality-dependent step sizes as future work.
5. **Combining data and prior curvature.** Describe the inverse-sum (parallel-sum) and Lehmer combinations without claiming that the individual preconditioners majorise the objective. The comparison is restricted to the Lewis--Sendov block-diagonal prior model. Report that Lehmer order \(q=0.01\) is marginally better but close to the inverse-sum result.
6. **Transition to Chapter 8.** State explicitly that the Chapter 8 protocol had already been fixed with the Lewis--Sendov block-diagonal preconditioner and \(q=0.01\) before the broader comparison was completed. Its use therefore reflects protocol continuity rather than evidence that it was the best-performing method.

## Figures

- Use `figures/PiW/comparison.png` for the qualitative multi-bed result.
- Use the two `mm_diag_tight` figures in `figures/preconditioning/subsets/subsets_only/` for subset organisation.
- Show BSREM divergence only in the later preconditioner comparison.
- Use the existing one-bed PET and SPECT preconditioner figures and the Lewis--Sendov Lehmer-versus-parallel figures.
- Put the full long-run baseline diagnostic plots and statistics in an appendix; retain only a compact convergence justification in Chapter 7.

## Metric and baseline reporting

For each run and region \(R\), define

\[
\operatorname{NRMSE}(x_k,x^\star;R)
=
\frac{\sqrt{N^{-1}\sum_{j\in R}(x_{k,j}-x^\star_j)^2}}
     {\sqrt{N^{-1}\sum_{j\in R}(x^\star_j)^2}}.
\]

The plotted line is the arithmetic mean of the five per-run NRMSE values, not the NRMSE of an averaged reconstruction. Describe \(x^\star\) as the final long-run baseline iterate rather than a known global minimiser. Support effective stationarity using the recorded objective-gap, relative-change, step-size, and iterate-velocity diagnostics.

## Claim boundaries

- Do not make a quantitative multi-bed claim from the qualitative comparison.
- Do not include or interpret the flawed prior-as-a-random-subset experiment.
- Do not call the tested preconditioners majorisers or claim guaranteed descent.
- Do not claim that Lewis--Sendov block-diagonal was selected because it was optimal or uniquely elegant.
- Do not pool numerical NRMSE values across experiments with different baseline objectives.
- Distinguish observations from suggested mechanisms, especially for subset splitting and modality-dependent step sizes.

## Thesis changes and verification

- Rewrite the relevant Chapter 7 methods, results, discussion, and transition text; add the selected figures and captions.
- Correct Chapter 8 wherever it identifies a different preconditioner or Lehmer order.
- Add the baseline-convergence appendix material and cross-reference it from Chapter 7.
- Preserve unrelated working-tree changes.
- Build the thesis, inspect the affected PDF pages, and check citations, references, figure legibility, and LaTeX warnings before completion.
