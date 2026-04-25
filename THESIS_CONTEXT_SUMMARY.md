# Thesis Context Summary

Prepared on 2026-03-28 from the local LaTeX thesis source, `references.bib`, and targeted literature/web review.

## What this thesis is about

This thesis is about improving post-treatment imaging and dosimetry after Yttrium-90 (`Y-90`) selective internal radiotherapy (`SIRT`/`TARE`) for liver cancer.

The core problem is that the two clinically available post-treatment modalities are complementary but both flawed:

- `Y-90 PET` has better spatial resolution, but the internal pair-production branch is extremely rare, so the data are very noisy.
- Bremsstrahlung `SPECT` has higher count statistics, but quantification is difficult because the emission spectrum is continuous, making scatter, septal penetration, and blur much harder to model than in ordinary photopeak SPECT.

The thesis argues that these problems should not be handled independently. Instead, PET, SPECT, and CT should be used together in a synergistic reconstruction framework that shares structural information while still allowing each modality to keep its own contrast characteristics.

The main proposed method is a CT-guided joint PET/SPECT variational regularizer called `weighted directional total nuclear variation` (`w-dTNV`), combined with a stochastic variance-reduced optimizer and a custom preconditioning strategy.

## Important caveat about the thesis status

The thesis source is clearly still a draft:

- `Chapter7.tex` is placeholder text (`\blindtext`).
- `Chapter6.tex` still contains a TODO note.
- `Chapter3.tex` also contains drafting notes.

So the best way to understand the actual contribution is not to rely on the formal conclusion chapter. The finished scientific story is mainly in the abstract, `Chapter0`, `Chapter2`, `Chapter3`, `Chapter5`, and `Chapter6`.

## Thesis in one paragraph

The thesis makes the case that accurate post-`Y-90` dosimetry needs three things at once: better physical modeling of bremsstrahlung SPECT, a better mathematically grounded way to couple PET and SPECT using CT-derived anatomical structure, and an optimizer that can handle strong coupled priors without becoming too slow or unstable. The proposed `w-dTNV` prior uses CT only for edge direction guidance, not for intensity coupling, and uses a joint Jacobian/nuclear-norm construction so PET and SPECT edges are encouraged to align without forcing them to have the same values. The empirical claim is that this improves small-lesion recovery and lesion-to-background contrast compared with directional-TV-only, TNV-only, and hybrid-kernel baselines.

## What each chapter is really doing

### 1. Clinical and physical framing

`Chapter0` and `Chapter1` frame the thesis around post-`SIRT` verification and dosimetry, not generic multimodal imaging. That matters because the count regime, physics, and downstream endpoint are unusual:

- activities are high, but image quality is still poor because `Y-90` is a difficult radionuclide to image;
- the relevant endpoint is not just prettier images, but more reliable absorbed-dose estimation and lesion assessment;
- CT is already present in the clinical workflow, so using it as structural guidance is practical.

### 2. Reconstruction gap and thesis direction

`Chapter2` identifies two research gaps:

- existing synergistic priors do not usually combine PET/SPECT coupling with explicit CT-derived directional guidance in a way suited to low-count `Y-90`;
- variance-reduced stochastic optimization and prior-aware preconditioning are underdeveloped for strongly coupled multimodality MAP reconstruction.

This chapter is important because it shows the thesis is not just proposing a new regularizer. It is also trying to solve the optimizer/scaling problem created by that regularizer.

### 3. Before synergy: fix the SPECT forward model

`Chapter3` is easy to underestimate, but it is strategically important. The thesis is saying that better priors are not enough if the SPECT model itself is wrong.

Key points:

- Scatter/additive terms are handled through a practical deep-network initialization followed by a SIMIND-based refinement loop.
- A simple Gaussian image-space model is proposed for bremsstrahlung range blur.
- Residual correction is tested as a cheap proxy for a more accurate Monte Carlo model.

The most useful conceptual result in this chapter is the residual-correction warning: in low-count, strongly regularized settings, correcting only the forward prediction does not restore a matched forward/adjoint pair. In other words, if the likelihood and prior are strong and the projector pair is mismatched, residual correction can change the effective balance of the optimization problem in an uncontrolled way. That is a strong, practically relevant insight.

### 4. Deconvolution chapter as a pilot study

`Chapter4` is not the main contribution, but it acts as a smaller-scale testbed for anatomy-guided inverse problems.

Takeaway:

- explicit directional-TV regularization outperformed RL/KRL/HKRL in the sphere study (`RL-dTV` reached the best NRMSE);
- anatomy helps, but restrictive kernel methods can bias reconstructions if the guidance image is imperfect;
- this chapter motivates why the final thesis method prefers explicit variational coupling over purely kernel-based guidance.

### 5. Mathematical core: `w-dTNV`

`Chapter5` is the mathematical center of the thesis.

What is new here:

- start from `TNV`, which encourages low-rank joint gradients across modalities;
- add weighting to handle scale mismatch across modalities;
- add directional CT guidance so anatomical edges influence edge orientation without making CT a third intensity channel;
- smooth the spectral penalty with a Charbonnier-type function so gradients and curvature surrogates are usable in optimization;
- derive cheap but meaningful voxel-wise preconditioners, especially a `2x2` block structure that matches the two-modality PET/SPECT case.

Interpretation:

- the thesis is trying to keep the good part of `TNV` (shared geometry through joint gradients),
- avoid naive CT intensity coupling,
- and still make the objective numerically tractable.

### 6. Actual validation on `Y-90 PET/SPECT`

`Chapter6` applies `w-dTNV` to the real target problem.

The design choices matter:

- PET and SPECT are reconstructed jointly on a shared PET-space grid;
- CTAC-derived information is used directionally through a local operator, not as a directly reconstructed third image;
- modality scaling is handled explicitly;
- optimization uses SVRG plus a combined data/prior preconditioner.

Main reported findings:

- On the bootstrapped NEMA phantom, `w-dTNV` improved small-sphere recovery relative to `dTV`, `wTNV`, and `SHKEM` at a matched operating point.
- In the patient cohort, `w-dTNV` gave higher lesion TBR than the comparators.
- Background CoV differences were smaller and not clearly in favor of a single method.

The clearest clinical-style result is the lesion TBR comparison:

- `dTV` vs `w-dTNV`: effect ratio `0.802`
- `wTNV` vs `w-dTNV`: effect ratio `0.915`
- `SHKEM` vs `w-dTNV`: effect ratio `0.935`

Those ratios are all below `1`, so all three comparators produced lower lesion TBR than `w-dTNV` in the reported cohort.

## My read of the real scientific contribution

The most important idea in this thesis is not just "joint PET/SPECT reconstruction". It is this more specific package:

- do not trust post-`Y-90` SPECT physics enough to ignore forward-model issues;
- do not force CT intensities into the reconstruction as if CT and emission images should match voxel-wise;
- do use CT edges to steer the geometry of the PET/SPECT penalty;
- do couple PET and SPECT at the level of gradients/Jacobians rather than raw intensities;
- do build the optimizer around the prior instead of treating the prior as an afterthought.

That is a coherent and defensible model-based position.

## How this fits into prior work from the references

### Kernel and anatomy-guided PET lineage

The thesis sits downstream of anatomy-guided and kernel-based PET reconstruction work such as:

- Wang and Qi (2015), PET reconstruction using kernel methods.
- Hutchcroft et al. (2016), anatomically aided PET reconstruction.
- Ehrhardt et al. (2016), PET reconstruction with anatomical MRI priors using parallel level sets / directional-TV style ideas.
- Deidda et al. (2019, 2022, 2023), hybrid-kernel methods for multimodal and `Y-90` settings.

Relative to that line of work, this thesis is pushing toward a more explicit variational model. Kernel methods can work well empirically, but they are more indirect: they encode side information in a basis or similarity structure. This thesis instead writes the coupling directly in the objective through a joint regularizer.

### TNV and vector-valued regularization lineage

The strongest mathematical ancestry is:

- Holt (2014), `Total Nuclear Variation`.
- Rigie and La Riviere (2015), TNV-style joint reconstruction in spectral CT.
- broader vector-valued and structure-promoting regularization work from Ehrhardt, Arridge, Kazantsev, Knoll, and others.

The thesis contribution here is to adapt that family to the specific PET/SPECT/CT `Y-90` setting by adding:

- modality weighting,
- directional anatomical guidance,
- differentiable smoothing,
- and optimization-aware curvature surrogates.

### Synergistic reconstruction as a field

The thesis is very much in the line described by Arridge, Ehrhardt, and Thielemans: synergistic reconstruction should combine modalities at reconstruction time when there is meaningful shared structure, but should avoid forcing false similarity. This thesis follows that philosophy closely.

## Extra outside context from web searches

### 1. Quantitative SPECT has moved toward standardization and traceability

Since the thesis motivation depends heavily on quantitative SPECT being clinically useful, the most relevant outside development is standardization:

- The `EANM practice guideline for quantitative SPECT-CT` (published December 5, 2022; issue date 2023) emphasizes calibration, verification, harmonization, partial-volume assessment, and traceable phantom workflows.
- Robinson et al. (published June 23, 2025) push this further by arguing for explicit measurement traceability and uncertainty reporting in quantitative SPECT.

Why this matters for the thesis:

- better reconstruction alone is not enough;
- if `w-dTNV` were ever translated clinically, it would need to sit inside a traceable, calibrated quantitative workflow.

### 2. PET and SPECT dosimetry may be closer than the thesis framing suggests

Kappadath et al. (published May 23, 2024; issue date September 2024) compared `Y-90 PET/CT` and `Y-90 SPECT/CT` dosimetry in 35 patients and found:

- high correlation between PET- and SPECT-based mean absorbed dose estimates;
- PET gave, on average, higher tumor mean dose and lower normal-liver mean dose than SPECT;
- the authors concluded that the modalities are largely equivalent for post-procedure dosimetry, with clinically acceptable differences.

Why this matters:

- the thesis is not justified because SPECT is unusable;
- it is justified because the modalities still carry different bias/noise/resolution profiles, especially at lesion level and for small structures;
- that makes joint reconstruction still interesting, but the bar for clinical impact is higher than "PET good, SPECT bad".

### 3. The sequential-kernel comparator kept evolving

Deidda et al. produced two highly relevant papers around the thesis topic:

- `Hybrid kernelised expectation maximisation for Bremsstrahlung SPECT reconstruction in SIRT with Y-90 micro-spheres` (2022): up to `56%` phantom recovery improvement at fixed CoV and `47%` patient ROI mean improvement over the hospital reference.
- `Triple modality image reconstruction of PET data using SPECT, PET, CT information...` (2023): up to `69%` lesion uptake increase over standard methods in `Y-90` patient data.

Why this matters:

- the thesis is entering a live method race, not an empty niche;
- `SHKEM` is a serious comparator, not a weak baseline;
- the thesis contribution is therefore best understood as "a more explicit variational and optimization-grounded alternative to the sequential hybrid-kernel program."

### 4. Hardware improvements are reducing one part of the problem

Linder et al. (published November 9, 2023) showed that long axial field-of-view PET/CT can make `Y-90 PET` much more usable for post-treatment dosimetry. Their phantom/patient work suggested that a `20 minute` scan with `2 iterations`, `5 subsets`, and a `4 mm` filter was sufficient for acceptable image quality and dosimetry on a LAFOV system.

Why this matters:

- at centers with LAFOV PET, the low-count PET problem is less severe than it was on older systems;
- that does not eliminate the thesis motivation, because most centers still do not have LAFOV PET and SPECT remains common;
- but it does mean the thesis sits partly in a technology-transition period.

### 5. Personalized dosimetry is becoming more explicit and indication-specific

Recent reviews and working-group recommendations show that `Y-90` dosimetry is becoming more formalized, not less:

- Lee and Kim (published June 4, 2025) review personalized dosimetry for HCC and emphasize multi-compartment modeling, toxicity constraints, and treatment-intent-specific dosing.
- Lam et al. (published March 28, 2025; issue date August 2025) give disease-specific recommendations for glass microspheres in non-HCC indications and recommend multi-compartment dosimetry for uni- and bilobar treatment, with explicit normal-tissue and tumor dose targets.

Why this matters:

- the thesis addresses exactly the kind of imaging problem that becomes more important when dose prescriptions become more personalized;
- if the field shifts further toward dose-driven treatment decisions, image reconstruction quality matters more, not less.

### 6. The optimizer direction taken in the thesis has been validated externally

Ehrhardt, Kereta, and Schramm (published September 17, 2025) studied PET reconstruction with variance reduction and prior-aware preconditioning and found that incorporating prior effects into the preconditioner is crucial for fast, stable convergence.

Why this matters:

- this directly supports the thesis instinct in `Chapter5` and `Chapter6`;
- the optimization part of the thesis is not side work, it is aligned with where the PET optimization literature continued to move.

### 7. Deep learning is the obvious outside competitor

Recent reviews make it clear that PET reconstruction is moving toward deep learning, model-based deep learning, and learned post-processing.

My interpretation:

- this thesis remains valuable because it is physically interpretable and easier to constrain;
- but any future extension should probably compare against modern learned reconstruction or learned correction approaches, especially for scatter/correction and low-count denoising.

## What this thesis does especially well

- It treats the problem as a full pipeline problem, not just a prior-design problem.
- It identifies a real failure mode for residual correction in regularized low-count reconstruction.
- It uses CT in a restrained way: directional guidance instead of forcing intensity agreement.
- It makes the optimizer part of the contribution, which is exactly what strong coupled priors usually need.

## What still looks unresolved

- Robustness to PET/SPECT/CT misregistration is important and only partly addressed.
- CT artefact transfer is acknowledged, but not deeply solved.
- Clinical validation is still small.
- The evaluation focuses on image metrics and lesion TBR more than full downstream dosimetric decision impact.
- Comparison against modern learned methods is still missing.
- Chapter7 is unfinished, so the final thesis narrative has not been fully polished into a single closing argument.

## Bottom-line interpretation

This is a strong model-based reconstruction thesis aimed at a very specific and clinically meaningful problem: post-`Y-90` activity imaging for dosimetry. Its real contribution is not merely proposing another multimodal prior. It is proposing a coherent strategy for combining:

- improved bremsstrahlung SPECT modeling,
- explicit PET/SPECT coupling,
- restrained CT guidance,
- and optimization that is aware of the coupled prior.

The thesis is most convincing when read as an answer to this question:

"How do we exploit PET, SPECT, and CT together in low-count post-`Y-90` imaging without forcing the wrong modalities to look the same, and without making the resulting optimization unusably slow?"

That is the right question, and the thesis gives a technically serious answer.

## Selected external sources

- Arridge, Ehrhardt, Thielemans (2021), `(An overview of) Synergistic reconstruction for multimodality/multichannel imaging methods`  
  https://pubmed.ncbi.nlm.nih.gov/33966461/

- Tsoumpas et al. (2021), `Synergistic tomographic image reconstruction: part 2`  
  https://pmc.ncbi.nlm.nih.gov/articles/PMC8255945/

- Dickson et al. (published December 5, 2022; issue date 2023), `EANM practice guideline for quantitative SPECT-CT`  
  https://pmc.ncbi.nlm.nih.gov/articles/PMC9931838/

- Deidda et al. (2022), `Hybrid kernelised expectation maximisation for Bremsstrahlung SPECT reconstruction in SIRT with Y-90 micro-spheres`  
  https://pubmed.ncbi.nlm.nih.gov/35377085/

- Deidda et al. (2023), `Triple modality image reconstruction of PET data using SPECT, PET, CT information...`  
  https://link.springer.com/article/10.1186/s40658-023-00549-4

- Linder et al. (2023), `Optimization of Y-90 Radioembolization Imaging for Post-Treatment Dosimetry on a Long Axial Field-of-View PET/CT Scanner`  
  https://pmc.ncbi.nlm.nih.gov/articles/PMC10670048/

- Kappadath et al. (published May 23, 2024; issue date September 2024), `Quantitative Evaluation of Y-90 PET/CT and Y-90 SPECT/CT-based Dosimetry following Yttrium-90 Radioembolization`  
  https://pmc.ncbi.nlm.nih.gov/articles/PMC11731408/

- Robinson et al. (published June 23, 2025), `Establishing measurement traceability for quantitative SPECT imaging`  
  https://pmc.ncbi.nlm.nih.gov/articles/PMC12185843/

- Lee and Kim (published June 4, 2025), `Optimizing Yttrium-90 Radioembolization Dosimetry for Hepatocellular Carcinoma: A Korean Perspective`  
  https://pmc.ncbi.nlm.nih.gov/articles/PMC12235541/

- Lam et al. (published March 28, 2025; issue date August 2025), `Clinical and dosimetric considerations for yttrium-90 glass microspheres... recommendations from an international multidisciplinary working group`  
  https://pubmed.ncbi.nlm.nih.gov/40148510/

- Ehrhardt, Kereta, Schramm (published September 17, 2025), `Fast PET reconstruction with variance reduction and prior-aware preconditioning`  
  https://pmc.ncbi.nlm.nih.gov/articles/PMC12484155/

- Hashimoto et al. (published February 6, 2024), `Deep learning-based PET image denoising and reconstruction: a review`  
  https://pubmed.ncbi.nlm.nih.gov/38319563/
