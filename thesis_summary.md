# Thesis Summary & Contextual Reference

**Title:** Exploiting Structural Synergy in post-SIRT imaging: Anatomically Guided Joint Variational Reconstruction of Yttrium-90 PET/SPECT/CT

**Author:** Sam Porter

**Institution:** Institute of Nuclear Medicine, Division of Medicine, University College London (UCL)

**Supervisors:** Kris (Thielemans), Daniel, and Simon

**Degree:** PhD

---

## 1. Clinical Problem and Motivation

### 1.1 Selective Internal Radiation Therapy (SIRT)

SIRT (also called TARE -- Transarterial Radioembolisation) is an established locoregional treatment for unresectable primary and secondary liver malignancies, particularly hepatocellular carcinoma (HCC) and liver metastases. It involves administering Y-90-labelled microspheres into the hepatic artery, which preferentially lodge in tumour vasculature and deliver high-dose beta radiation to the tumour while sparing healthy liver parenchyma. Typical administered activities are of the order of 2 GBq, and recommended tumour doses often exceed 200 Gy.

**Clinical context (2024-2025):**
- NICE has positive guidance for unresectable HCC in the UK; for colorectal liver metastases the position is more conditional, with access in some cases restricted to research settings or special commissioning arrangements
- International recommendations (Levillain et al., EJNMMI 2021) call for minimum mean tumour-absorbed doses of 100-120 Gy for HCC
- The 2024 AAPM Medical Physics Practice Guideline 14.a (Busse et al.) standardises Y-90 microsphere radioembolization practice
- MIRD Pamphlet No. 29 (2024, "MIRDy90") introduces a vetted dosimetry tool for evaluating patient-specific prescribed activities
- Growing momentum toward personalised, image-based dosimetry as standard of care

### 1.2 The Imaging Challenge

Post-therapy imaging is essential for treatment verification, outcome modelling, and optimisation of subsequent therapy. However, Y-90 is designed for therapy, not imaging -- it is approximately a pure beta-minus emitter with no useful direct gamma emission. Two secondary imaging pathways exist, both deeply problematic:

**Y-90 PET (Internal Pair Production):**
- Relies on a rare nuclear transition: the Y-90 nucleus can decay to an excited 0+ state of Zr-90, which de-excites via internal pair production (branching ratio ~32 x 10^-6, i.e., ~0.003%)
- The resulting positron annihilates to produce standard 511 keV photon pairs
- Offers relatively high spatial resolution (PET-quality)
- But is extremely count-starved: the vast majority of decays produce no PET-detectable signal
- Dominated by noise and bremsstrahlung background
- Typical clinical scan: 15 minutes per bed position

**Bremsstrahlung SPECT:**
- High-energy beta particles decelerating in tissue emit bremsstrahlung X-rays across a continuous energy spectrum (up to 2.28 MeV)
- Can be imaged with SPECT gamma cameras
- Offers substantially higher count statistics than PET
- But suffers from: (a) continuous spectrum -- no photopeak, making standard energy-window scatter correction infeasible; (b) severe spatial blur from collimator-detector response and bremsstrahlung range; (c) septal penetration from high-energy photons degrading contrast and resolution
- Quantitative accuracy is historically limited

**The key insight:** Both modalities image the same underlying microsphere distribution but with complementary strengths and weaknesses. Independent reconstruction discards a strong physical coupling. This motivates joint reconstruction.

### 1.3 Role of CT

CT provides:
1. **Attenuation correction (CTAC):** Converting Hounsfield units to linear attenuation coefficient maps at relevant photon energies for both PET and SPECT
2. **Anatomical guidance:** High-resolution structural information (liver boundaries, lesion margins, vascular anatomy) that can regularise the ill-posed emission reconstruction, especially in the extreme low-count regime of Y-90 imaging

---

## 2. Technical Foundations (Chapters 1-2)

### 2.1 Emission Tomography Forward Model

Both PET and SPECT reconstruction solve the same fundamental inverse problem: estimating a non-negative activity distribution x from noisy projection data y under the Poisson statistical model:

```
y_i ~ Poisson((Ax + b)_i)
```

where A is the system matrix encoding measurement physics (geometry, attenuation, detector response) and b represents additive corrections (scatter, randoms).

**PET system matrix factorisation:**
```
A = A_det,sens * A_det,blur * A_att * A_geom * A_ToF * A_positron
```

**SPECT system matrix factorisation:**
```
A = A_det,sens * A_det,blur * A_geom,att * A_bremss
```

The `A_bremss` term is specific to bremsstrahlung SPECT and models the additional blur from bremsstrahlung range -- photons are emitted along the electron track, not at the decay site.

### 2.2 Maximum Likelihood and Its Limitations

The Poisson negative log-likelihood:
```
L(x) = sum_i [ (Ax+b)_i - y_i * log((Ax+b)_i) ]
```

ML estimation via MLEM/OSEM is the clinical standard but has practical limitations:
- MLEM can in principle converge to the ML solution, but convergence is very slow and the ML solution itself is dominated by noise due to the ill-conditioning of A
- In practice, early stopping is used as implicit regularisation, introducing iteration-dependent bias
- OSEM accelerates early convergence via data subsets but exhibits limit-cycle behaviour and does not in general converge to the ML solution
- Both rely on early stopping to produce clinically acceptable images, which compromises quantification

### 2.3 MAP/Penalised-Likelihood Framework

The thesis adopts the Bayesian MAP formulation:
```
x_MAP = argmin_{x >= 0} { L(x) + beta * R(x) }
```

where R(x) is a regularisation functional (prior) and beta controls regularisation strength.

### 2.4 Regularisation Strategies (Chapter 2 -- Literature Review)

The thesis reviews a comprehensive hierarchy of regularisers:

**Image-based (no anatomical guidance):**
- **Quadratic prior:** Oversmooths edges, computationally efficient
- **Total Variation (TV):** Edge-preserving (piecewise constant), non-differentiable at zero
- **Relative Difference Prior (RDP):** Clinically deployed (GE "Q.Clear"), adapts edge preservation to local intensity, interpolates between quadratic and TV behaviour

**Anatomically guided (single modality + guidance):**
- **Bowsher prior:** Data-adaptive neighbourhood selection based on guidance image similarity -- smoothing only within anatomically consistent regions
- **Parallel Level Sets (PLS):** Penalises gradient components of emission image orthogonal to guidance image gradients -- encourages parallel edges
- **Directional Total Variation (dTV):** Normalised direction field from guidance image defines anisotropic projection operator; penalises gradient components orthogonal to anatomical level-set direction; reduces to standard TV in homogeneous guidance regions (Ehrhardt & Betcke, SIAM 2016)
- **Kernel Expectation Maximisation (KEM):** Implicit regularisation via image model x = K*alpha, where K is a kernel matrix built from guidance image features (Wang & Qi, IEEE TMI 2015)
- **Hybrid KEM (HKEM):** Augments kernel with features from the evolving PET estimate to mitigate suppression of PET-unique structures (Deidda et al., EJNMMI Physics 2019/2022)

**Synergistic (joint multi-modality reconstruction):**
- **Joint Total Variation (JTV):** Frobenius-norm coupling of multi-modality Jacobians -- encourages co-located edges
- **Total Nuclear Variation (TNV):** Nuclear-norm coupling of voxel-wise Jacobians -- promotes rank deficiency (edge alignment) through the convex envelope of rank (Holt 2014, Rigie & La Riviere 2015)
- **Sequential HKEM for Y-90:** Deidda et al. (EJNMMI Physics 2023) -- the most directly relevant prior work. Reconstructs SPECT first with CT/SPECT-HKEM, then uses the SPECT estimate as functional guidance for PET reconstruction. Reports 18-69% higher lesion uptake than standard OSEM.

**Distinction between JTV and TNV:** JTV uses the Frobenius norm (Schatten-2 norm) on voxel-wise Jacobians and couples channels only through total gradient magnitude. TNV uses the nuclear norm (Schatten-1 norm), which is the convex envelope of matrix rank. The nuclear norm exhibits sharp grooves along coordinate axes in singular-value space, encouraging one singular value to vanish (promoting low-rank = aligned edges), while the Frobenius norm is smooth and shrinks both singular values equally.

### 2.5 Optimisation Algorithms (Chapter 2)

**MLEM/OSEM:** Clinical standard. OSEM partitions data into subsets for faster early convergence but exhibits limit-cycle behaviour -- does not converge to the MAP solution.

**BSREM:** Adds diminishing step sizes to recover convergence, but can converge slowly with strong priors.

**Variance-Reduced Stochastic Gradient Methods (SAGA/SVRG):**
- Retain low per-iteration cost of subset methods
- Suppress subset-induced stochasticity via stored/reference gradients
- SAGA: maintains table of most recent subset gradients, higher memory
- SVRG: periodically computes reference full gradient, lower memory
- Allow larger step sizes than BSREM while maintaining stable convergence
- Key references: Twyman et al. (IEEE TMI 2023), Ehrhardt et al. (Frontiers in Nuclear Medicine 2025)

**Preconditioning:** Critical for fast convergence. The EM-type preconditioner:
```
P_EM = diag(x / (A^T * 1))
```
approximates the inverse diagonal of the Fisher information matrix. For MAP reconstruction, "prior-aware preconditioning" (Ehrhardt et al. 2025) incorporates the prior Hessian diagonal into the preconditioner via harmonic averaging, yielding substantially faster convergence.

**L-BFGS-B:** Quasi-Newton method with bound constraints. Provides a reference optimiser but is computationally expensive for large-scale emission tomography (requires full objective/gradient evaluation per iteration). Used in the thesis for the deconvolution sandbox problem (Chapter 4).

### 2.6 Identified Research Gap

Two specific gaps motivate the thesis:

1. **Synergistic, anatomy-informed coupling:** Existing Jacobian-based priors (TNV) are not routinely combined with explicit directional guidance from CT in a way robust to low-count regimes
2. **Synergy-aware optimisation:** Variance-reduced subset algorithms and prior-aware preconditioners have mainly been studied for single-modality PET with RDP; their extension to synergistic MAP reconstruction with cross-modality coupling is underexplored

---

## 3. Improving the Y-90 Bremsstrahlung System Model (Chapter 3)

### 3.1 Challenges in Bremsstrahlung Imaging

Three fundamental problems:
1. **Continuous energy spectrum:** No photopeak for scatter windowing
2. **Septal penetration:** High-energy tail degrades spatial resolution
3. **Bremsstrahlung range blur:** Photons produced along the electron track, not at the decay site

### 3.2 Additive Corrections (Scatter Estimation)

The thesis develops a practical scatter correction pipeline:

1. **Deep learning initialisation:** A pre-trained CNN predicts scatter projections from emission projections and CT-derived attenuation information (Xiang et al. 2020). Despite being trained on a different scanner/protocol (domain shift), it provides a sensible starting point.

2. **Iterative OSEM-SIMIND refinement loop:** Alternates between OSEM reconstruction (12 subsets, 5 epochs) and SIMIND Monte Carlo scatter simulation for 10 outer iterations, with 5 mm Gaussian smoothing of intermediate images to suppress noise amplification.

3. **Update strategies:**
   - Offline (fixed): Compute scatter once, fix for remainder of reconstruction
   - Online (intermittent updates): Re-simulate scatter during reconstruction
   - Full replacement vs. damped updates: Damped (relaxed) updates with alpha=0.2 provide better stability

### 3.3 Bremsstrahlung Range Blur Model

A pragmatic approximation: model the bremsstrahlung range blur as an isotropic image-space Gaussian whose variance is matched to Monte Carlo-derived displacement statistics (from GATE fast-simulation studies, Rault et al. 2010). For composite energy windows, Gaussian variances are collapsed by second-moment matching weighted by per-window counts.

Validated against SIMIND and GATE forward projections for clinical energy windows [38-126 keV], [50-150 keV], [75-225 keV] using a point-source-in-sphere configuration.

### 3.4 Residual Correction

Investigates Fu & Qi's residual correction (RC) strategy for bremsstrahlung SPECT. RC intermittently estimates the modelling discrepancy between accurate (expensive) and fast forward models and folds it into an additive term.

**Key finding:** RC works well for high-count, low-background phantom data but becomes unreliable in low-count clinical regimes with strong regularisation. The thesis provides a theoretical analysis: RC corrects the forward prediction but does not restore a matched forward/adjoint pair. In the MAP setting, mismatched adjoints change the effective balance between data fidelity and regularisation, leading to divergence from the solution associated with the fully matched model.

---

## 4. Anatomy-Informed Deconvolution for PET PVC (Chapter 4)

This chapter serves as a controlled "sandbox" -- studying anatomical guidance and regularisation in a simplified image-domain inverse problem (post-reconstruction partial volume correction via deconvolution) before tackling the full reconstruction problem.

### 4.1 Methods Compared

- **Richardson-Lucy (RL):** Baseline deconvolution (EM for image-domain Poisson model)
- **Kernelised RL (KRL):** Implicit regularisation via x = K*alpha (anatomy-derived kernel)
- **Hybrid Kernelised RL (HKRL):** Kernel includes both anatomical and PET-derived features, frozen after initial iterations
- **RL-dTV:** Explicit anatomically guided regularisation via directional TV penalty, optimised with L-BFGS-B

### 4.2 Key Results

On a spherical phantom with known ground truth:
- RL-dTV achieves the best NRMSE (0.0177), highest RC (0.889), and lowest background CoV (0.123)
- KRL/HKRL improve over baseline RL but are sensitive to initialisation
- RL-dTV is insensitive to initialisation (converges to same solution)
- KRL and HKRL behave almost identically (HKRL adds little benefit in this setting)

**Key insight:** All guidance-based methods risk bias when guidance doesn't reflect tracer uptake. KRL is most restrictive (x must lie in range(K)); RL-dTV is less restrictive. HKRL partially mitigates mismatch but must be controlled. These observations directly motivate the synergistic prior design in later chapters.

---

## 5. Weighted Directional Total Nuclear Variation: Theory & Optimisation (Chapter 5)

### 5.1 The w-dTNV Prior

The central theoretical contribution of the thesis. Extends TNV in two orthogonal ways:

1. **Directional guidance from CT:** Instead of appending CT as a third coupled channel (which creates ill-conditioned scaling problems), CT edge information enters through directional projection operators that modify which gradient components are penalised. This decouples the roles of emission-to-emission coupling and anatomical guidance.

2. **Voxel-wise, modality-wise diagonal weighting:** Compensates for heterogeneous scaling and spatially varying reliability across modalities (critical for PET/SPECT where count levels and resolutions differ by orders of magnitude).

**Why not append CT as a third modality?** The thesis provides a detailed argument:
- A single global scaling for CT cannot simultaneously avoid swamping the weaker emission modality and preserve consistent anatomical steering across both channels
- Scaling any one column changes not only its own contribution but also cross-terms and eigenstructure of the Gram matrix
- Using CT only directionally decouples these roles: modality weights control emission-to-emission coupling strength; directional operators control anatomical edge preference
- Retaining a 2-modality Jacobian yields a 2x2 Gram matrix with closed-form expressions for all required spectral quantities (no SVD needed)

**Mathematical formulation:**

The directional multi-modality Jacobian:
```
G_n^dir(z) = [Pi_{n,1} (D z_1)_n | Pi_{n,2} (D z_2)_n] * B
```

where Pi_{n,m} = I - gamma_m * xi_n * xi_n^T is the directional operator (shrinks gradient components along the CT edge normal), B = diag(b_1, b_2) is the modality weight matrix, and D is a d=18 directional finite-difference operator.

The regulariser:
```
R_wdTNV(z) = sum_n phi(Y_n)
```

where phi(Y) = sum_l g(sigma_l(Y)) with g the Charbonnier smoothing g(s) = sqrt(s^2 + epsilon^2).

### 5.2 Differentiability and Efficient Gradient Computation

The thesis establishes that the smoothed nuclear norm is C^2 differentiable via spectral function theory (Lewis & Sendov). The Jacobian-space gradient simplifies to:
```
grad_G phi(G) = G * (G^T G + epsilon^2 I)^{-1/2}
```

For M=2 modalities, this inverse square root of the 2x2 SPD matrix admits a closed-form expression using only scalar invariants (trace and determinant), avoiding per-voxel SVDs entirely:
```
S^{-1/2} = sqrt(t+2s) / (2*delta + s*t) * [[beta+s, -gamma], [-gamma, alpha+s]]
```

The image-space gradient follows by applying the chain rule through the finite-difference operator:
```
grad_z R_TNV(z) = -div Q
```

### 5.3 Synergy-Aware Preconditioners

A major technical contribution: a hierarchy of six preconditioners for the w-dTNV prior, constructed by:
1. Modelling curvature in Jacobian coordinates (via MM/IRLS surrogates or Lewis-Sendov Hessian calculus)
2. Pulling curvature back through the finite-difference stencil to image coordinates
3. Taking voxel-wise block-diagonal or diagonal approximations

**Route I: MM/IRLS quadratic surrogates (fast, 2x2 Gram algebra):**
- **MM-A (block, exact 2x2 weight):** Most accurate MM option, retains cross-modality coupling via closed-form inverse square root. Fastest in practice (~0.79s phantom, ~2.0s patient).
- **MM-B (diagonal, Gershgorin-safe):** Conservative diagonal variant absorbing coupling through Gershgorin row-sums
- **MM-B* (diagonal, naive):** Drops off-diagonal coupling; can underestimate curvature
- **MM-C (Frobenius/isotropic):** Scalar weight depending only on ||Y||_F; cheapest but least expressive

**Route II: Lewis-Sendov Hessian-based majorisers (more accurate, more expensive):**
- **LS-D (block-Gershgorin):** Tighter curvature modelling via Hermitian dilation representation and block-Gershgorin majorisation. Most expensive (~2.85s phantom, ~11.4s patient).
- **LS-E (row-sum diagonal):** Diagonal majoriser of full Jacobian-coordinate Hessian; most conservative LS option

---

## 6. Evaluation: Joint Y-90 PET/SPECT Reconstruction (Chapter 6)

### 6.1 Experimental Setup

**Joint MAP formulation:**
```
x_hat = argmin_{x>=0} { sum_m D_m(y_m | A_m x_m + b_m) + lambda * R_wdTNV(z) }
```

Minimised using SVRG with decaying step size and the synergy-aware diagonal preconditioner combining EM-type data scaling with the w-dTNV curvature proxy via clamped Lehmer mean of order 0.1.

**Directional guidance:** CT attenuation map (mu-map) with stabilised unit direction field. eta = 0.05 * max||D mu||.

**Global modality normalisation:** Percentile-based scaling over CTAC-derived foreground mask:
```
b_m = beta_m / (q_0.99(x_m^init) - q_0.01(x_m^init))
```

**Directional finite-difference operator:** d=18 directions (3 axial face + 6 edge + step lengths 1 and 2).

**Methods compared:**
- **w-dTNV:** Full proposed method (synergistic + directional + weighted)
- **dTV:** Directional TV on PET only (no SPECT coupling)
- **w-TNV:** Weighted TNV without directional guidance (no CT edge information)
- **SHKEM:** Sequential Hybrid Kernel EM (Deidda et al. 2023) -- the state-of-the-art comparator

### 6.2 Data

**NEMA IEC Phantom:**
- Sphere-to-background ratio 10:1
- PET: Siemens Biograph mCT, 15-hour acquisition (bootstrapped into B=20 independent 15-minute datasets)
- SPECT: GE Discovery 670, 15 minutes
- Sphere diameters: 10, 13, 22, 28, 37 mm

**Clinical cohort:**
- 10 post-SIRT patients from Oxford University Hospitals
- 1 patient for hyperparameter tuning, 9 for testing
- PET: GE Discovery 710, 2 bed positions x 15 min
- SPECT: GE Discovery NM/CT 670, 15 min
- Tumour VOIs delineated on vendor PET reconstruction via automatic gradient-based method

### 6.3 Key Results

**Phantom -- Recovery Coefficients (RC):**
- w-dTNV achieves improved RC relative to dTV and w-TNV across all sphere sizes
- The gain is most pronounced for the smallest spheres (10 mm, 13 mm), consistent with guidance stabilising edge geometry when PET statistics are poor
- w-dTNV improves upon SHKEM for the smallest spheres (10 mm: both at ~0.74 effect ratio vs reference, but w-dTNV is the reference)
- SHKEM performs comparably for larger spheres (28 mm, 37 mm: effect ratios ~0.97-0.99, not significant)
- Statistical significance (BH-adjusted): dTV and w-TNV significantly worse than w-dTNV for spheres <= 28 mm (p < 0.05); SHKEM significantly worse for spheres <= 13 mm

**Phantom -- Reproducibility (inter-bootstrap CoV):**
- dTV and w-TNV yield lower voxel-wise CoV than w-dTNV (stronger effective smoothing at matched operating point)
- SHKEM shows higher variability in background/hot regions but improved stability in cold lung insert
- Trade-off: w-dTNV's improved recovery comes at some cost to reproducibility

**Clinical -- Tumour-to-Background Ratio (TBR):**
- w-dTNV yields significantly higher lesion TBR than all comparators:
  - vs dTV: effect ratio 0.802, p < 10^-4
  - vs w-TNV: effect ratio 0.915, p = 0.004
  - vs SHKEM: effect ratio 0.935, p = 0.020
- Background CoV differences are smaller and not statistically significant for most comparisons

**Clinical -- Heterogeneity:**
- Lesions with clear anatomical correlates (calcification, liver capsule boundaries in mu-map) benefit most from mu-guided anisotropy and PET/SPECT coupling
- Lesions lacking such correlates show limited benefit
- This is consistent with the design: CTAC guidance steers orientation preferences but cannot invent edges unsupported by functional data

### 6.4 Limitations

- Sensitivity to CTAC artefacts: structured CT artefacts (e.g., arms-down streaking) can imprint into the directional weighting field
- Operating-point trade-off: improved recovery vs. reduced reproducibility compared to simpler methods
- Single training patient for hyperparameter selection

---

## 7. Discussion, Conclusion & Future Work (Chapter 7)

Chapter 7 is incomplete in the current draft (contains only placeholder text). Based on the thesis content, the key conclusions and future directions are:

### Key Contributions

1. **w-dTNV prior:** A novel synergistic regulariser that combines total nuclear variation (for cross-modality edge alignment) with directional CT guidance (for anatomically informed anisotropy) and modality-dependent weighting (for heterogeneous PET/SPECT scales). The design choice to use CT directionally rather than as a third coupled channel is theoretically justified and empirically validated.

2. **Efficient gradient and closed-form spectral computations:** For M=2 modalities, all required quantities (gradient, curvature proxies) are computed via 2x2 Gram matrix algebra without SVDs, enabling practical large-scale reconstruction.

3. **Hierarchy of synergy-aware preconditioners:** Six practical preconditioners spanning a cost-accuracy trade-off, from cheap diagonal approximations to tight block-diagonal curvature models.

4. **Bremsstrahlung system model improvements:** Gaussian proxy for range blur validated against Monte Carlo, practical scatter correction pipeline, and analysis of residual correction fragility in low-count/regularised settings.

5. **Comprehensive evaluation:** w-dTNV improves small-object recovery and clinical lesion TBR relative to dTV, w-TNV, and the state-of-the-art SHKEM method.

### Likely Future Directions

- Robust direction-field estimation and artefact-aware guidance strategies
- Extension to voxel-level dosimetry (absorbed dose estimation from improved activity maps)
- Online/adaptive scatter correction during synergistic reconstruction
- Deep learning-based direction field estimation or learned priors
- Extension to other challenging radionuclides (Lu-177, etc.)
- Clinical validation with larger patient cohorts and dose-outcome correlation

---

## 8. External Context and Related Work

### 8.1 Key Research Groups

- **UCL Institute of Nuclear Medicine (Thielemans, Hutton, Arridge):** STIR/SIRF frameworks, HKEM development (with Deidda), variance-reduced PET reconstruction, synergistic imaging
- **Ehrhardt (University of Bath / UCL):** dTV, variance-reduced optimisation (SAGA/SVRG) for PET, prior-aware preconditioning (PETRIC 2024 challenge winner)
- **Deidda (National Physical Laboratory, UK):** HKEM, triple-modality PET/SPECT/CT for Y-90
- **Holt (2014):** Original TNV formulation with extended Jacobians
- **Rigie & La Riviere (University of Chicago):** TNV for spectral CT
- **Knoll et al. (NYU):** TNV/TGV coupling for joint MR-PET
- **Wang & Qi (UC Davis):** Original kernel method (KEM) for PET

### 8.2 Software Ecosystem

- **STIR:** Open-source C++ library for PET/SPECT reconstruction (UCL/SyneRBI). Core computational engine.
- **SIRF:** Python/MATLAB interface on top of STIR (and Gadgetron for MR), part of CCP SyneRBI. Likely used for this thesis given the UCL affiliation and STIR references.
- **CIL (Core Imaging Library):** Provides optimisation algorithms (primal-dual, ADMM, gradient-based) integrated with SIRF.
- **SIMIND:** Monte Carlo simulation for SPECT (used for scatter estimation).
- **GATE/Geant4:** Monte Carlo simulation framework (used for bremsstrahlung range validation).
- **parallelproj:** GPU-accelerated projector framework integrated into STIR.

### 8.3 State of the Art (2024-2025)

**Post-SIRT dosimetry:**
- Kappadath et al. (Med Phys 2024): PET yields ~14.5% higher tumour mean dose than SPECT; comparable healthy liver doses
- Long axial FOV (LAFOV) PET/CT (e.g., Biograph Vision Quadra) increasing sensitivity for Y-90
- Voxel-based dosimetry increasingly adopted alongside (and in some centres replacing) single-compartment MIRD models, though single-compartment approaches remain widely used in routine practice
- Deep learning pipelines emerging for Y-90 bremsstrahlung scatter estimation

**Reconstruction methods:**
- PETRIC 2024 challenge (IEEE NSS/MIC Tampa): All algorithms used SIRF/STIR; winning approach used SVRG with prior-aware preconditioning
- Emerging deep learning approaches: beta-VAE constraints for synergistic PET/CT, diffusion posterior sampling for spectral CT
- The Deidda et al. 2023 triple-modality paper remains the only published work specifically addressing joint PET/SPECT/CT reconstruction for Y-90 SIRT using kernel methods

**This thesis fills the gap:** To the best of this literature review, no variational (TNV/dTV-based) approach has previously been applied to the triple-modality Y-90 PET/SPECT/CT problem, nor has synergy-aware preconditioning been studied for such coupled priors.

### 8.4 Key References (Selected)

| Reference | Contribution |
|-----------|-------------|
| Holt (IEEE TIP 2014) | Total Nuclear Variation and extended Jacobians |
| Rigie & La Riviere (PMB 2015) | TNV for spectral CT reconstruction |
| Knoll et al. (IEEE TMI 2017) | Nuclear-norm coupling within TGV for joint MR-PET |
| Ehrhardt & Betcke (SIAM 2016) | Directional Total Variation (dTV) |
| Wang & Qi (IEEE TMI 2015) | Kernel Expectation Maximisation (KEM) |
| Deidda et al. (EJNMMI Phys 2019/2022) | Hybrid KEM for Y-90 bremsstrahlung SPECT |
| Deidda et al. (EJNMMI Phys 2023) | Triple-modality PET/SPECT/CT reconstruction for Y-90 |
| Twyman et al. (IEEE TMI 2023) | SAGA/SVRG for 3D PET with RDP |
| Ehrhardt et al. (Frontiers 2025) | Variance reduction + prior-aware preconditioning for PET |
| Ahn et al. (IEEE TMI 2002) | BSREM framework |
| Lewis & Sendov (SIAM 2001) | Twice differentiable spectral functions |
| Fu & Qi (2010) | Residual correction for positron range blur |
| Levillain et al. (EJNMMI 2021) | International recommendations for personalised SIRT |
| Kappadath et al. (Med Phys 2024) | Y-90 PET/CT vs SPECT/CT dosimetry comparison |

---

## 9. Thesis Structure Summary

| Chapter | Title | Content |
|---------|-------|---------|
| Intro | Introduction | Clinical motivation, thesis overview |
| 1 | Physics and Measurement Models | PET/SPECT/CT physics, Y-90 imaging challenges |
| 2 | Penalised-Likelihood Reconstruction | MAP framework, regularisation review, optimisation algorithms, research gap |
| 3 | Improvements to Y-90 Bremsstrahlung System Model | Scatter correction, bremsstrahlung range blur, residual correction |
| 4 | Anatomy-Informed Deconvolution for PET PVC | Sandbox: RL, KRL, HKRL, RL-dTV deconvolution |
| 5 | w-dTNV: Theory and Optimisation | Novel prior formulation, differentiability, 6 preconditioner variants |
| 6 | w-dTNV for Joint Y-90 PET/SPECT Reconstruction | Phantom and clinical evaluation vs SHKEM |
| 7 | Discussion, Conclusion & Future Work | (Incomplete in current draft) |
| Appendix | Additional Work | (Placeholder in current draft) |
