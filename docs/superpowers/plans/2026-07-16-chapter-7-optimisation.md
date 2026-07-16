# Chapter 7 Optimisation Narrative Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite Chapter 7 as an evidence-led optimisation study, correct the frozen Chapter 8 optimisation protocol, and document the long-run reference reconstruction in an appendix.

**Architecture:** Keep the mathematical foundations and externally referenced labels in `Chapter7.tex`, but reorder the chapter around the experimental story: qualitative multi-bed motivation, common evaluation protocol, subset organisation, prior-aware preconditioning, preconditioner combination, and transition to Chapter 8. Put detailed baseline diagnostics in a dedicated appendix input file. Preserve unresolved publication records as explicit Zenodo-citation TODOs rather than adding hand-written bibliography entries.

**Tech Stack:** LuaLaTeX, `biblatex`/Biber, `glossaries`, `graphicx`, `subcaption`, PNG figures, GNU Make.

---

## File map

- Modify `Chapter1.tex`: correct the thesis chapter overview so Chapters 6--9 match the actual files.
- Modify `Chapter7.tex`: perform the principal narrative, methods, results, figure, and transition rewrite while preserving public labels used elsewhere.
- Modify `Chapter8.tex`: correct the application protocol to Lewis--Sendov block-diagonal preconditioning and Lehmer order \(q=0.01\).
- Modify `acronyms_symbols.tex`: add the NRMSE acronym.
- Modify `Appendices.tex`: input the new convergence appendix immediately before the Colophon.
- Create `AppendixBaselineConvergence.tex`: contain only the reference-run convergence evidence and figures.
- Do not modify `references.bib`: unresolved PiW and IEEE records remain precise `\TODO{...}` markers for the automatic Zenodo bibliography workflow.
- Add only the selected files from `figures/PiW/` and `figures/preconditioning/`; do not add `subsets/incorrect/`, unused BSREM subset plots, or unrelated `figures/lesion_grower/` assets.
- Do not stage generated `Main.pdf` or `Main.syg`; they already contain user changes.

### Task 1: Correct document-level terminology and chapter map

**Files:**
- Modify: `Chapter1.tex:13-23`
- Modify: `acronyms_symbols.tex:46-55`

- [ ] **Step 1: Record the pre-edit assertions**

Run:

```bash
rg -n 'Chapter 6.*optimisation machinery|Chapter 7.*evaluates' Chapter1.tex
rg -n 'newacronym\{nrmse\}' acronyms_symbols.tex
```

Expected: the two outdated chapter-summary lines are present and the NRMSE acronym is absent.

- [ ] **Step 2: Correct the chapter overview**

Replace the current Chapter 2--7 outline with the actual sequence: Chapter 2 physics and measurement models; Chapter 3 reconstruction priors and algorithms; Chapter 4 bremsstrahlung system-model improvements; Chapter 5 anatomy-informed deconvolution; Chapter 6 \(w\)-dTNV; Chapter 7 optimisation; Chapter 8 joint \(^{90}\)Y PET/SPECT evaluation; Chapter 9 discussion and conclusions. Retain the appendix item.

- [ ] **Step 3: Add the acronym definition**

Add beside the existing quantitative metrics:

```tex
\newacronym{nrmse}{NRMSE}{normalised root-mean-square error}
```

- [ ] **Step 4: Verify and commit**

Run:

```bash
rg -n 'Chapter [6-9]|newacronym\{nrmse\}' Chapter1.tex acronyms_symbols.tex
git diff --check -- Chapter1.tex acronyms_symbols.tex
git add Chapter1.tex acronyms_symbols.tex
git commit -m "docs: align thesis overview with optimisation chapters"
```

Expected: Chapters 6--9 have distinct, accurate roles; the new acronym appears once; the commit contains only these two files.

### Task 2: Establish the Chapter 7 motivation and common protocol

**Files:**
- Modify: `Chapter7.tex:1-64,318-378`
- Add: `figures/PiW/comparison.png`

- [ ] **Step 1: Replace the methods-only opening**

Rewrite lines 4--6 so the chapter announces three linked optimisation questions: joint reconstruction of overlapping PET bed positions, organisation of PET/SPECT subsets, and prior-aware preconditioning. State that these choices lead into the already-frozen Chapter 8 application protocol.

- [ ] **Step 2: Move multi-bed reconstruction to the opening**

Move the existing `sec:multibed`, `subsec:multibed_notation`, `subsec:stitching_failure`, and `subsec:joint_multibed` material ahead of subset/preconditioner results while preserving all labels and equations. Retain the combined-bed forward model, gradient, and sensitivity equations.

Soften the conclusion to: separately reconstructed and fused beds show overlap-region noise/discontinuities in the qualitative example, whereas a single combined image permits the edge-preserving prior to act consistently across the overlap. Do not claim a quantitative improvement or a general guarantee.

- [ ] **Step 3: Add the qualitative PiW figure**

Use a single centred figure:

```tex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.72\linewidth]{figures/PiW/comparison.png}
    \caption{Qualitative comparison of separately reconstructed and fused PET bed positions with a joint multi-bed reconstruction using CT-guided edge-preserving regularisation. The shaded region marks the bed overlap and the cyan lines mark the displayed profiles. This example illustrates overlap-region behaviour only; no quantitative comparison was performed.}
    \label{fig:optim_multibed_qualitative}
\end{figure}
```

Remove the unresolved `\cite{porter_simultaneous_2025}` command and keep the missing PiW publication as `\TODO{Add the Zenodo-managed citation for the multi-bed PET work.}`; do not add a manual entry to `references.bib`.

- [ ] **Step 4: Correct the SVRG formulation for the analysed runs**

For data functions \(\mathcal D_i\) sampled uniformly with replacement, write the actual prior-always estimator as

```tex
\begin{equation}
g^{(k)} = \nabla\mathcal R(\bm z^{(k)})
+ \nabla\mathcal D(\widetilde{\bm z})
+ N_s\!\left[\nabla\mathcal D_{i_k}(\bm z^{(k)})
-\nabla\mathcal D_{i_k}(\widetilde{\bm z})\right],
\qquad i_k\sim\operatorname{Unif}\{1,\ldots,N_s\}.
\end{equation}
```

Explain that the prior remains outside the sampler and its complete gradient is evaluated at every stochastic update. Preserve the projected update and decaying-step equations.

- [ ] **Step 5: Add the common evaluation protocol and metric**

State: anthropomorphic phantom scanned on the Mediso AnyScan at NPL; 18 PET and 18 SPECT subsets; five stochastic realisations per curve; first 20 complete data passes; one 1000-epoch reference run using the same objective but more conservative optimisation settings. Define, for each run and region \(R\),

```tex
\begin{equation}
\operatorname{NRMSE}(x_k,x^\star;R)
=
\frac{\sqrt{N^{-1}\sum_{j\in R}(x_{k,j}-x^\star_j)^2}}
     {\sqrt{N^{-1}\sum_{j\in R}(x^\star_j)^2}}.
\end{equation}
```

Define \(R\) as the whole image, hot-sphere mask, or cold-sphere mask. State that \(x^\star\) is the final long-run reference iterate, not a known global minimiser. The plotted centre is the arithmetic mean of five per-run NRMSE values and the envelope is their pointwise minimum--maximum.

- [ ] **Step 6: Verify and commit**

Run:

```bash
rg -n 'methods-only|no numerical results|fig:optim_multibed_qualitative|NRMSE|sampled.*replacement|prior.*every' Chapter7.tex
git diff --check -- Chapter7.tex
git add Chapter7.tex figures/PiW/comparison.png
git commit -m "docs: add multi-bed motivation and convergence protocol"
```

Expected: the methods-only claim is absent, the qualitative claim boundary is explicit, and only the selected PiW image is added.

### Task 3: Replace the subset sweep with the approved paired/separate comparison

**Files:**
- Modify: `Chapter7.tex:238-316` after Task 2 reordering
- Add: `figures/preconditioning/subsets/subsets_only/exp_subset_selection_1bpos_mm_diag_tight_alpha_1_prior_always.png`
- Add: `figures/preconditioning/subsets/subsets_only/exp_subset_selection_1bpos_mm_diag_tight_alpha_1_SPECT_prior_always.png`

- [ ] **Step 1: Remove the invalid experiment from the narrative**

Delete `subsec:subset_prior_strategy`, its probability/epoch equations, and every result or methods statement about sampling the prior as its own stochastic subset. Retain only the full-prior-every-update configuration.

- [ ] **Step 2: Restrict the comparison to the one-bed 18+18 experiment**

Keep `eq:separate_subset_functions` and `eq:paired_subset_functions`. Explain that one matched data pass contains 36 modality-specific data-subset evaluations in both organisations:

- paired: 18 joint PET+SPECT parameter updates and 18 prior-gradient evaluations;
- separate: 36 modality-specific parameter updates and 36 prior-gradient evaluations.

State that this matches data-projection work but not update count or total compute.

- [ ] **Step 3: Add the two-panel subset figure**

Stack the figures at full text width using existing `subcaption` conventions:

```tex
\begin{figure}[htbp]
    \centering
    \begin{subfigure}{0.99\linewidth}
        \includegraphics[width=\linewidth]{figures/preconditioning/subsets/subsets_only/exp_subset_selection_1bpos_mm_diag_tight_alpha_1_prior_always.png}
        \caption{PET.}
    \end{subfigure}
    \begin{subfigure}{0.99\linewidth}
        \includegraphics[width=\linewidth]{figures/preconditioning/subsets/subsets_only/exp_subset_selection_1bpos_mm_diag_tight_alpha_1_SPECT_prior_always.png}
        \caption{SPECT represented on the PET grid.}
    \end{subfigure}
    \caption{Convergence for paired and modality-separated subsets using the MM diagonal curvature preconditioner. Lines show the mean per-run NRMSE over five stochastic realisations and shading shows the pointwise minimum--maximum over the first 20 matched data passes. The complete prior gradient was evaluated at every update.}
    \label{fig:optim_subset_organisation}
\end{figure}
```

- [ ] **Step 4: Write the bounded interpretation**

Report that separated subsets converge more rapidly for both modalities, most clearly in the hot- and cold-sphere regions. Present the doubled number of parameter updates per data pass as a plausible explanation. Explicitly note the extra prior evaluations, so this is not an isolated comparison of modality grouping or a compute-matched speed benchmark.

Remove the unresolved `\cite{porter_optimising_2024}` command. Leave the source as `\TODO{Add the Zenodo-managed citation for IEEE Xplore document 10654962.}`; do not add `porter_optimising_2024` to `references.bib` manually.

- [ ] **Step 5: Verify and commit**

Run:

```bash
rg -n 'Prior Sampling Strategy|prior_sampling_probability|subset_epoch_length|fig:optim_subset_organisation|36.*evaluations|18.*joint' Chapter7.tex
git diff --check -- Chapter7.tex
git add Chapter7.tex figures/preconditioning/subsets/subsets_only/exp_subset_selection_1bpos_mm_diag_tight_alpha_1_prior_always.png figures/preconditioning/subsets/subsets_only/exp_subset_selection_1bpos_mm_diag_tight_alpha_1_SPECT_prior_always.png
git commit -m "docs: report paired and separated subset convergence"
```

Expected: no prior-sampling subsection remains; only the two MM-diagonal subset figures are tracked.

### Task 4: Reconcile the preconditioner theory and terminology

**Files:**
- Modify: `Chapter7.tex:66-236` after reordering

- [ ] **Step 1: Retain the data-curvature foundation**

Keep `eq:shared_grid_sensitivity`, `eq:bsrem_preconditioner`, and `eq:data_hessian_surrogate`. Describe BSREM as data-only EM scaling, not as a full-objective curvature model.

- [ ] **Step 2: Present the MM surrogate family without guarantees**

Retain the local diagonal, Gershgorin-inflated diagonal, and local voxel-block constructions, but replace “majoriser” claims with “curvature surrogate”, “Gershgorin-inflated surrogate”, or “conservative surrogate”. State explicitly that none of the implemented preconditioners has been established as a majoriser of the full objective and no monotone-descent guarantee is claimed.

- [ ] **Step 3: Add the Lewis--Sendov block-diagonal construction**

Describe the Lewis--Sendov option as the voxel-block diagonal extracted from the exact Hessian of the smoothed spectral penalty. State that it retains PET/SPECT cross-curvature within each \(2\times2\) voxel block but discards spatial off-diagonal blocks. Explain that its update requires the voxel-wise singular-value decomposition used by the spectral Hessian, so updating it every epoch has a non-negligible computational cost.

- [ ] **Step 4: Define inverse-sum combination**

Use preconditioner operands \(P_{\mathrm D}=\widehat H_{\mathrm D}^{-1}\) and \(P_{\mathcal R}=\widehat H_{\mathcal R}^{-1}\), and call

```tex
\begin{equation}
P_{\parallel}
=
\left(P_{\mathrm D}^{-1}+P_{\mathcal R}^{-1}\right)^{-1}
=
\left(\widehat H_{\mathrm D}+\widehat H_{\mathcal R}\right)^{-1}
\end{equation}
```

the inverse-sum (parallel-sum) combination. Do not call either operand or the result a majoriser.

- [ ] **Step 5: Replace the methods table**

List exactly: data-only BSREM; MM local diagonal; MM Gershgorin-inflated diagonal; MM local voxel-block; Lewis--Sendov block-diagonal. Mark all prior-aware methods as using inverse-sum combination in the principal comparison. Describe the long-run reference separately as one 1000-epoch Gershgorin-diagonal MM run with the same objective and different optimisation settings.

- [ ] **Step 6: Verify and commit**

Run:

```bash
rg -n -i 'Lewis|inverse-sum|parallel-sum|majoris|monotone|tab:preconditioner_settings' Chapter7.tex
git diff --check -- Chapter7.tex
git add Chapter7.tex
git commit -m "docs: reconcile prior-aware preconditioner models"
```

Expected: Lewis--Sendov appears in the theory/table; any remaining “majorisation--minimisation” text names the MM method rather than claiming a preconditioner guarantee.

### Task 5: Add the preconditioner and step-size results

**Files:**
- Modify: `Chapter7.tex`
- Add: `figures/preconditioning/exp3_best_precond_1bpos_limited.png`
- Add: `figures/preconditioning/exp3_best_precond_1bpos_SPECT.png`
- Add: `figures/preconditioning/exp3_best_precond_1bpos_step1_2_limited.png`
- Add: `figures/preconditioning/exp3_best_precond_1bpos_step1_2_SPECT.png`

- [ ] **Step 1: Add the principal PET/SPECT comparison**

Create `fig:optim_preconditioners` from stacked `0.99\linewidth` PET and SPECT subfigures. The caption must state: five-run mean; minimum--maximum envelope; first 20 data passes; same objective/reference within the comparison; data-only BSREM versus the four prior-aware models.

- [ ] **Step 2: Report the observed modality dependence**

State that all prior-aware methods have broadly similar convergence, with modest region-dependent differences and no universal winner. Report the data-only BSREM divergence for count-limited PET. Contrast this with SPECT, for which BSREM remains stable and is sometimes competitive or faster, indicating that the prior-aware alternatives can be conservative for that modality.

- [ ] **Step 3: Add the step-size comparison**

Create `fig:optim_step_size` from the two `step1_2` assets, stacked as PET and SPECT. Explain in the caption that solid curves use initial step 1 and dashed curves use initial step 2.

In prose, state that increasing the common initial step improves SPECT convergence but destabilises the limited-count PET case. Present modality-dependent relaxation parameters as future work, not as a tested solution.

- [ ] **Step 4: Verify and commit**

Run:

```bash
rg -n 'fig:optim_preconditioners|fig:optim_step_size|BSREM|modality-dependent|universal winner' Chapter7.tex
git diff --check -- Chapter7.tex
git add Chapter7.tex figures/preconditioning/exp3_best_precond_1bpos_limited.png figures/preconditioning/exp3_best_precond_1bpos_SPECT.png figures/preconditioning/exp3_best_precond_1bpos_step1_2_limited.png figures/preconditioning/exp3_best_precond_1bpos_step1_2_SPECT.png
git commit -m "docs: add preconditioner convergence results"
```

### Task 6: Add the Lehmer study and honest Chapter 8 transition

**Files:**
- Modify: `Chapter7.tex`
- Add: `figures/preconditioning/lehmer_vs_parallel_1bpos_ls_block_diag.png`
- Add: `figures/preconditioning/lehmer_vs_parallel_1bpos_ls_block_diag_SPECT.png`

- [ ] **Step 1: Define the matrix Lehmer family**

For voxel-wise SPD preconditioners \(B=P_{\mathcal R}\) and \(D=P_{\mathrm D}\), define

```tex
C=D^{-1/2}BD^{-1/2},\qquad
L_q(B,D)=D^{1/2}f_q(C)D^{1/2},\qquad
f_q(t)=\frac{t^q+1}{t^{q-1}+1}.
```

State that the implemented output is \(\tfrac12L_q\), so \(q=0\) equals the parallel sum. Note once that the plot legend uses \(p\) for the same order denoted by \(q\) in the text.

- [ ] **Step 2: Add the PET/SPECT comparison**

Create `fig:optim_lehmer_parallel` using the stacked Lewis--Sendov PET and SPECT assets. State in the caption that this experiment changes only the combination rule for the Lewis--Sendov block-diagonal prior model.

- [ ] **Step 3: Report the bounded result and transition**

Report that \(q=0.01\) is marginally better than the inverse-sum result in these curves, but the difference is small; do not generalise beyond Lewis--Sendov. State that Chapter 8 had already frozen Lewis--Sendov block-diagonal plus \(q=0.01\) before the wider comparison, so its use reflects protocol continuity rather than post-hoc optimal selection.

- [ ] **Step 4: Rewrite the Chapter 7 conclusion**

Summarise: joint multi-bed reconstruction is needed for consistent overlap regularisation; separated modality subsets converge faster under matched data-projection work; prior-aware preconditioning is essential for stable limited-count PET but less decisive for SPECT; tested curvature models are broadly similar; modality-specific step sizes remain future work; Chapter 8 retains the frozen LS/\(q=0.01\) configuration.

- [ ] **Step 5: Verify and commit**

Run:

```bash
rg -n 'q=0\.01|q=0|plot legend|fig:optim_lehmer_parallel|protocol continuity|post-hoc' Chapter7.tex
git diff --check -- Chapter7.tex
git add Chapter7.tex figures/preconditioning/lehmer_vs_parallel_1bpos_ls_block_diag.png figures/preconditioning/lehmer_vs_parallel_1bpos_ls_block_diag_SPECT.png
git commit -m "docs: report Lehmer preconditioner combination study"
```

### Task 7: Correct the frozen optimisation protocol in Chapter 8

**Files:**
- Modify: `Chapter8.tex:173-255,275`

- [ ] **Step 1: Clarify the application subset objective**

Define \(N_{\mathrm d}\) as the total number of PET and SPECT data functions for the reconstruction being described. Retain the application implementation in which each stochastic function receives \(N_{\mathrm d}^{-1}\tilde{\mathcal R}_{\mathrm{w-dTNV}}\), and distinguish this from Chapter 7’s prior-outside-the-sampler experiment. Ensure the SVRG scaling uses \(N_{\mathrm d}\), not an ambiguous \(S\).

- [ ] **Step 2: Replace diagonal with block-diagonal preconditioning**

Rename `subsec:90y_precond` to “Block-diagonal preconditioning” while preserving the label. Retain the modality-specific data scaling as the diagonal block \(P_{\mathrm D}\); replace the local-diagonal prior claim with the Lewis--Sendov voxel-block diagonal \(P_{\mathcal R}\). State that the block is recomputed once per epoch via the smoothed spectral Hessian/SVD and held fixed within the epoch.

- [ ] **Step 3: Correct the Lehmer order and rationale**

Use the same \(\tfrac12L_q(P_{\mathcal R},P_{\mathrm D})\) definition as Chapter 7 with \(q=0.01\). Remove both occurrences of \(q=0.1\), the scalar element-wise formula, and the claim that Chapter 7 simply replaced the application rule. State that this setting was frozen before the broader optimisation benchmark.

- [ ] **Step 4: Soften the multi-bed application claim**

At the current line 275, replace “avoids overlap-region artefacts” with wording that joint reconstruction permits one prior over the combined field of view and was motivated by the qualitative overlap comparison in Chapter 7.

- [ ] **Step 5: Verify and commit**

Run:

```bash
rg -n 'q=0\.1|local diagonal|Block-diagonal preconditioning|Lewis|N_\{\\mathrm d\}|frozen|overlap' Chapter8.tex
git diff --check -- Chapter8.tex
git add Chapter8.tex
git commit -m "docs: correct Chapter 8 optimisation protocol"
```

Expected: no \(q=0.1\) or local-diagonal application claim remains; external labels are unchanged.

### Task 8: Add the long-run reference convergence appendix

**Files:**
- Create: `AppendixBaselineConvergence.tex`
- Modify: `Appendices.tex:68-71`
- Modify: `Chapter7.tex`
- Add: `figures/preconditioning/baseline/baseline_convergence_final_objective_panel.png`
- Add: `figures/preconditioning/baseline/baseline_convergence_final_objective_panel_linear.png`
- Add: `figures/preconditioning/baseline/baseline_convergence_final_panel.png`
- Add: `figures/preconditioning/baseline/baseline_convergence_metrics.csv`
- Add: `figures/preconditioning/baseline/baseline_convergence_summary.md`

- [ ] **Step 1: Create the dedicated appendix**

Start the new file with:

```tex
\chapter{Convergence Diagnostics for the Long-Run Reference Reconstruction}
\label{app:baseline_convergence}
```

Explain that the one-bed reference used the same reconstruction objective as the 20-pass comparisons but a 1000-epoch run, Gershgorin-diagonal MM preconditioning, and more conservative relaxation settings. Define \(J^\star=\min_k J_k\) as the best observed objective, not a proven global minimum.

- [ ] **Step 2: Add the diagnostic figures**

Place the log and linear objective panels side by side at `0.49\linewidth` under `fig:app_baseline_objective`. Place the PET iterate-velocity/step panel at `0.82\linewidth` under `fig:app_baseline_velocity`.

- [ ] **Step 3: Add the exact stationarity table**

Create `tab:app_baseline_stationarity` with:

| Quantity | Value |
|---|---:|
| Final objective gap / observed objective decrease | \(8.819\times10^{-9}\) |
| Final relative objective change per epoch | \(1.717\times10^{-11}\) |
| Final PET iterate velocity per epoch | \(7.285\times10^{-8}\) |
| Final relaxed step size | \(0.2174\) |
| Epoch reaching 99.9\% of observed decrease | 45 |
| Step size at that epoch | \(0.8606\) |

Conclude only that the reference was effectively stationary relative to the best observed iterate while the step size remained appreciable.

- [ ] **Step 4: Wire and cross-reference the appendix**

Insert `\input{AppendixBaselineConvergence}` immediately before the Colophon in `Appendices.tex`. Add a compact Chapter 7 sentence pointing to Appendix~`\ref{app:baseline_convergence}` when introducing \(x^\star\).

- [ ] **Step 5: Verify and commit**

Run:

```bash
rg -n 'app:baseline_convergence|fig:app_baseline|tab:app_baseline_stationarity|AppendixBaselineConvergence' Chapter7.tex Appendices.tex AppendixBaselineConvergence.tex
git diff --check -- Chapter7.tex Appendices.tex AppendixBaselineConvergence.tex
git add Chapter7.tex Appendices.tex AppendixBaselineConvergence.tex figures/preconditioning/baseline/baseline_convergence_final_objective_panel.png figures/preconditioning/baseline/baseline_convergence_final_objective_panel_linear.png figures/preconditioning/baseline/baseline_convergence_final_panel.png figures/preconditioning/baseline/baseline_convergence_metrics.csv figures/preconditioning/baseline/baseline_convergence_summary.md
git commit -m "docs: document baseline convergence diagnostics"
```

### Task 9: Build, inspect, and repair the thesis

**Files:**
- Verify: `Main.tex`, `Chapter1.tex`, `Chapter7.tex`, `Chapter8.tex`, `Appendices.tex`, `AppendixBaselineConvergence.tex`
- Generated but do not stage: `Main.pdf`, `Main.syg`, auxiliary LaTeX files

- [ ] **Step 1: Run static source checks**

Run:

```bash
git diff --check
rg -n 'methods-only|no numerical results|Prior Sampling Strategy|q=0\.1|local diagonal.*Chapter|known global|minimiser.*eight' Chapter7.tex Chapter8.tex AppendixBaselineConvergence.tex
rg -n '\\TODO\{.*(10654962|Zenodo|PiW|PET is Wonderful)' Chapter7.tex
```

Expected: the stale scientific claims are absent; missing automatic references are represented only by explicit TODOs.

- [ ] **Step 2: Force a full thesis build**

Run:

```bash
make -B Main.pdf
```

Expected: LuaLaTeX, Biber, glossaries, and both resolving passes complete successfully.

- [ ] **Step 3: Audit warnings**

Run:

```bash
rg -n 'LaTeX Error|Citation .* undefined|Reference .* undefined|There were undefined references|Please \(re\)run Biber|Overfull \\hbox' Main.log
```

Expected: no LaTeX errors or undefined Chapter 7/8/appendix references. Any unresolved bibliography warning must correspond to a pre-existing explicit automatic-citation TODO rather than a newly invented key.

- [ ] **Step 4: Verify rendered content and figure placement**

Run:

```bash
pdftotext -layout Main.pdf /tmp/Main-layout.txt
rg -n 'Preconditioning and Subset Selection|Convergence Diagnostics for the Long-Run|Lewis|0\.01|Paired subsets|Separate subsets' /tmp/Main-layout.txt
pdftoppm -png -r 120 Main.pdf /tmp/thesis-page
```

Use the PDF table of contents and the extracted-text matches to identify all pages in Chapter 7, the corrected Chapter 8 optimisation subsection, and the new appendix. Inspect those rendered PNGs for clipped axes, unreadable legends, float ordering, table overflow, and blank/near-blank pages.

- [ ] **Step 5: Repair and rebuild until clean**

Make only source-level layout corrections such as figure widths, float placement, caption line breaks, or table column widths. Re-run `make -B Main.pdf`, the warning audit, and the rendered-page inspection after each correction.

- [ ] **Step 6: Confirm repository scope and commit final repairs**

Run:

```bash
git status --short
git diff --check
git diff --name-only HEAD
```

Expected: no unrelated assets are staged or committed; `Main.pdf`, `Main.syg`, `figures/lesion_grower/`, unused PiW files, unused subset plots, and `subsets/incorrect/` remain outside the implementation commits. If source repairs were required, commit only those source files:

```bash
git add Chapter1.tex Chapter7.tex Chapter8.tex Appendices.tex AppendixBaselineConvergence.tex acronyms_symbols.tex
git commit -m "docs: refine optimisation chapter layout"
```

If no repairs were required, do not create an empty commit.
