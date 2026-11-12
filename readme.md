# OLYMPUS TECH 4.0    Project Report
**Team:** The gentlemen  
**Competition:** Olympus Tech 4.0 Data Science Problem Statement (Cosmosoc)

---

## Problem Statement

Following the corruption of the AI supercomputer **EDEN**, humanity's emergency relocation initiative lost access to critical astronomical tracking pipelines, historical planetary archives, and the core settlement-viability prediction algorithm. Our team (**The gentlemen**) executed a three-stage recovery and reconstruction mission to restore these capabilities:

1. **Part One - Signal in the Static (Astronomical Image Processing):** We analyzed a sequence of 20 noisy Flexible Image Transport System (FITS) astronomical frames to isolate a faint, uncatalogued moving body. Using sky-level normalization, masked median-reference image subtraction, and matched-filter point-spread function (PSF) modeling, we detected and tracked the object across all 20 frames. The object follows a gentle quadratic trajectory from (39.80, 44.67) px to (65.53, 53.86) px with a 2D root-mean-square (RMS) positional residual of 0.213 px (closely approaching the noise floor of 0.206 px) and a shift-and-stack combined significance of 35.7σ.
2. **Part Two - Fragments of a Lost Archive (Data Restoration & Exploratory Analysis):** We audited and cleaned 6,400 corrupted planetary records from a nearby galaxy, resolving unit-scale anomalies, text-encoding fragmentation, and missing values via domain-informed chemical and physical constraints. Through visual and statistical analysis, we evaluated five core astrobiological research questions and conducted an independent investigation into surface hydrology, demonstrating that **hazard-free environments** (mean index = 83.9) and **global surface oceans** (mean index = 65.4, +14.9 points over dry worlds) are the strongest drivers of planetary habitability.
3. **Bonus Task - Rebuilding EDEN (Predictive Machine Learning Architecture):** To reconstruct EDEN's decision engine (`viable` ⟺ `settlement_index` ≥ 60), we engineered 176 domain-specific astrophysics, atmospheric chemistry, and orbital mechanics features. We trained a 10-fold cross-validated, 12-model dual-objective stacking ensemble (combining CatBoost, LightGBM, XGBoost, HistGradientBoosting, ExtraTrees, and Neural MLPs across both regression and classification objectives). Our Level-2 meta-ensemble achieved an Out-of-Fold (OOF) **ROC-AUC of 0.94748**, **Accuracy of 88.84%**, and **F1-Score of 0.8643**, classifying 653 (40.81%) of the 1,600 unseen test worlds as viable for human settlement.

---

## Part One: Signal in the Static

### 1.1 Problem Context & Objective
Before going rogue, EDEN captured 20 sequential FITS images of a dense stellar field containing a faint, moving astronomical source obscured by background starlight, sensor readout noise, and cosmic-ray hits. Our objective was to calibrate and visualize the raw frames, remove the static stellar field via difference imaging, establish a statistically rigorous detection pipeline, reconstruct the object's kinematic trajectory, and verify its authenticity against sensor artefacts.

---

### 1.2 Methodology

#### Step 1: FITS Extraction, Calibration, and Raw Visualization
- All 20 FITS frames were extracted, naturally sorted in chronological frame order, and cast into 64-bit floating-point arrays.
- Missing or corrupted pixel values (`NaN`/`Inf`), where present, were imputed using the spatial median of the corresponding frame.
- Each frame was plotted with calibrated $(x, y)$ pixel axes, a uniform grayscale intensity stretch, and a labeled colorbar reporting flux in Analog-to-Digital Units ($ADU$). Because the target's flux is comparable to local sky background fluctuations, it cannot be reliably tracked by visual inspection of raw frames alone.

#### Step 2: Two-Pass Masked Median Reference Construction & Difference Imaging
- **Background Harmonization:** Small frame-to-frame sky background offsets were measured via robust sigma-clipped statistics and subtracted prior to stacking.
- **Masked Median Stacking:** A standard temporal median across aligned frames removes moving sources while preserving static stars. However, in a short 20-frame sequence, a slow-moving source can leave a faint positive bias ("ghost trail") in the median stack, which subsequently degrades SNR during subtraction. To prevent this, we implemented a **two-pass masked median reference**:
  1. An initial median image was constructed to identify the preliminary path of the moving object and locate sharp 1-pixel cosmic-ray spikes.
  2. In the second pass, pixels along the preliminary trajectory and confirmed sharp artefacts were masked out before computing the temporal median pixel-by-pixel:

$$I_{\text{ref}}(x, y) = \operatorname{median}_{t \notin \mathcal{M}(x,y)} \left( I_{\text{cal}}(x, y, t) \right)$$

- **Difference Image Generation:** Each calibrated frame was subtracted from the clean reference image:

$$D(x, y, t) = I_{\text{cal}}(x, y, t) - I_{\text{ref}}(x, y)$$

  Static background stars cancel out to zero mean (leaving only localized photon shot-noise residuals), whereas the moving source emerges clearly as a localized positive point-spread function (PSF) shifting position across frames.

#### Step 3: Noise Estimation, Matched-Filter Detection, and Artefact Rejection
- **Robust Noise Floor Estimation:** For each difference frame, background noise $\sigma_{\text{bg}}$ was estimated using the Median Absolute Deviation ($MAD$) scaled to an equivalent Gaussian standard deviation ($\sigma \approx 1.4826 \times MAD$), preventing bright residuals or the target itself from inflating the noise metric.
- **Spatial Morphology & Cosmic-Ray Filtering:** Candidate detections were required to exceed a noise-based threshold and span multiple connected pixels consistent with the optical PSF width. Sharp single-pixel spikes were isolated using a sharpness/Laplacian ratio criterion. While an initial unrefined pass flagged 28 sharp candidates (7 of which overlapped the true object's core), our refined spatial-frequency discriminator isolated exactly **3 true single-pixel cosmic-ray/hot-pixel artefacts**, with **0 false flags on the target**.
- **Sub-Pixel Astrometric Centroiding:** In each frame, the object's exact $(x_t, y_t)$ coordinates and integrated flux were measured by fitting a 2D circular Gaussian PSF integrated over finite pixel areas.

#### Step 4: Kinematic Trajectory Modeling
We evaluated three polynomial motion models  Linear (constant velocity), Quadratic (constant acceleration), and Cubic (jerk)  across the 20 frames ($t \in \{0, 1, \dots, 19\}$) using the Bayesian Information Criterion ($BIC$) and 2D RMS positional error:

$$x(t) = x_0 + v_x t + \frac{1}{2} a_x t^2, \qquad y(t) = y_0 + v_y t + \frac{1}{2} a_y t^2$$

---

### 1.3 Key Findings & Quantitative Results

#### Kinematic Model Selection & Trajectory Parameters
The quadratic motion model provided the optimal balance between physical explanatory power and parsimony, reducing the 2D RMS error by **71%** compared to a straight-line fit:

| Motion Model | 2D RMS Error ($px$) | Bayesian Information Criterion ($BIC$) | Physical & Statistical Interpretation |
| :--- | :---: | :---: | :--- |
| **Linear** | 0.736 px | 597.6 | Underfits; fails to capture vertical curvature |
| **Quadratic (Selected)** | **0.213 px** | **66.0** | **Optimal balance of goodness-of-fit and model simplicity** |
| **Cubic** | 0.205 px | 69.3 | Over-parameterized; marginal RMS drop penalized by $BIC$ |

- **Start Coordinates (Frame 1):** $(39.80, 44.67)\text{ px}$
- **End Coordinates (Frame 20):** $(65.53, 53.86)\text{ px}$
- **Mean Apparent Speed:** 1.46 px/frame
- **Mid-Sequence Velocity Vector:** $v_x = 1.357 \pm 0.007\text{ px/frame}$, $v_y = 0.477 \pm 0.007\text{ px/frame}$
- **Measured Acceleration Vector:**
  - Horizontal acceleration: $a_x = -0.0013 \pm 0.0026\text{ px/frame}^2$ (statistically consistent with zero)
  - Vertical acceleration: $a_y = +0.0470 \pm 0.0028\text{ px/frame}^2$ (highly significant detection at $>16\sigma$, explaining the gentle upward curvature of the path)

#### Signal-to-Noise Ratio (SNR) & Photometry
- **Difference Image Background Noise:** $\sigma_{\text{bg}} = 2.80 \pm 0.02\text{ ADU/pixel}$
- **Mean Integrated Source Flux:** 124.9 ADU
- **Peak-Pixel SNR:** Mean of 5.6 per frame (light is distributed across the PSF footprint)
- **Matched-Filter SNR:** Ranges from **8.7 to 13.2** across individual frames, with a **mean matched-filter SNR of 11.03**.

#### Statistical Validation & Reliability Verification
To prove that the recovered track could not arise from sensor noise or subtraction residuals, we executed three independent validation experiments:

1. **Synthetic Source Injection & Recovery Monte Carlo:**
   Artificial PSF sources matching the target's flux profile were injected into source-free regions of the difference frames. Recovery experiments yielded a 1D scatter of $\sigma_x = 0.141\text{ px}$ and $\sigma_y = 0.149\text{ px}$, corresponding to a theoretical **2D noise-limited astrometric floor of 0.206 px** (with <0.01 px systematic bias). Our observed trajectory RMS of **0.213 px** matches this theoretical limit to within 0.007 px, proving that residual scatter is pure background photon noise. Furthermore, completeness testing showed a 7% recovery rate at $\text{SNR}=3$, 54% at $\text{SNR}=5$, 97% at $\text{SNR}=7$, and **100% recovery for $\text{SNR} \ge 8$**  placing our target ($\text{SNR} \in [8.7, 13.2]$) firmly in the 100% completeness regime.
2. **Negative-Image False-Alarm Test:**
   Running the identical detection pipeline on inverted difference frames ($-D(x,y,t)$) yielded an average of only **0.10 false positive candidates per frame**, none of which formed a kinematically linkable multi-frame trajectory.
3. **Shift-and-Stack Co-Addition Test:**
   Shifting all 20 difference frames along the recovered quadratic trajectory and stacking them produced a coherent point source at an empirical significance of **35.7σ**. By contrast, stacking along random noise trajectories peaked at **2.6σ**, and stacking along the exact same spatial coordinates in **reverse chronological order** yielded **-0.6σ** (pure noise).

#### Pipeline Refinement Summary
Comparing our final two-pass masked reference and pixel-integrated PSF fitting pipeline against our initial baseline analysis demonstrated consistent gains across all metrics:

| Diagnostic Metric | Initial Baseline Pipeline | Final Refined Pipeline | Relative Improvement |
| :--- | :---: | :---: | :---: |
| **2D Trajectory RMS Error** | 0.242 px | **0.213 px** | **12.0% reduction** |
| **Leave-One-Out (LOO) CV RMS** | 0.285 px | **0.253 px** | **11.2% reduction** |
| **Median Astrometric Uncertainty** | 0.155 px | **0.138 px** | **11.0% reduction** |
| **Trajectory $R^2$ ($X$ / $Y$)** | 0.99970 / 0.99510 | **0.99969 / 0.99676** | **$Y$-axis fit substantially improved** |
| **Mean Matched-Filter SNR** | 10.30 | **11.03** | **$+7.1\%$ SNR gain** |

#### Physical Nature & Limitations
The point-like PSF morphology and smooth, curved apparent motion against fixed stars are characteristic of a solar-system minor body (such as a main-belt asteroid, near-Earth object, or distant comet core) or an artificial satellite/spacecraft exhibiting parallax or orbital curvature. However, key limitations include: (1) the short 20-frame baseline, (2) localized photon noise elevation when the object passes near a bright background star, and (3) the absence of World Coordinate System (WCS) headers, exposure timestamps, and telescope plate scale ($\text{arcsec/pixel}$), which prevents converting pixel velocities into physical orbital elements.

---

## Part Two: Fragments of a Lost Archive

### 2.1 Problem Context & Objective
EDEN's planetary archive contained 6,400 labeled training records and 1,600 unlabeled test records detailing physical, orbital, atmospheric, and ecological parameters alongside EDEN's continuous `settlement_index` (0 to 100). Because EDEN corrupted the database before going offline, the raw records suffered from severe unit-scale inconsistencies, typographical fragmentation, missing fields, and multi-label string concatenations. Our mission was to restore the dataset to analytical integrity, answer five fundamental habitability research questions, and formulate an independent scientific investigation.

---

### 2.2 Data Cleaning & Restoration Methodology

A systematic audit of the raw archive revealed six major classes of data corruption, each resolved through deterministic, physics- and chemistry-informed restoration rules:

1. **Categorical Taxonomy & String Harmonization:**
   - *Star Types:* Consolidated synonymous labels (`"red dwarf"`, `"Red dwarf (M-type)"`, `"M-type main sequence"`, `"M-type red dwarf"`) into canonical classes (`M-type red dwarf`, `White dwarf remnant`, `Binary system (K+K)`, `Binary system (F+G)`, `Binary system (G+M)`, `Subdwarf`, etc.).
   - *Planet Types:* Standardized compound and casing variations into canonical archetypes (`terrestrial`, `super-Earth`, `ocean world`, `tidally locked`, `rogue planet`, `desert world`, `volcanic world`, `ice world`, `gas giant moon`, `relic/artificial world`), while preserving secondary attributes (e.g., `"tidally locked super-Earth"`) via binary indicator flags.
   - *Water Presence, Surface Composition, & Hazards:* Normalized free-text strings into structured categories (`global ocean`, `seas`, `subsurface`, `trace`, `none` for hydrology; `rocky`, `volcanic`, `ice`, `crystalline`, `metallic`, `sandy` for lithosphere; and standardized hazard classes). Boolean `rings` indicators (`"yes"`, `"true"`, `"1"`, `"no"`, `"false"`, `"0"`) were unified to binary $\{0, 1\}$.
2. **Atmospheric String Parsing & Cross-Column Imputation:**
   - The `atmosphere_primary_gases` column contained semi-structured text (e.g., `"N2 78%, O2 21%, Ar 0.9%, Trace 0.1%"`). We built regular-expression parsers to extract exact percentage concentrations for 14 distinct chemical species (N₂, O₂, Ar, CO₂, CH₄, H₂O, He, H₂, SO₂, Ne, C₂H₆, NH₃, H₂S, CO) plus `Trace` gases.
   - Whenever dedicated numeric columns (`oxygen_percent`, `co2_percent`, `methane_percent`) were missing (`NaN`), we cross-imputed them directly from the parsed `atmosphere_primary_gases` string, falling back to trace baseline medians (0.05% for O₂/CH₄, 0.15% for CO₂) only if absent from both sources.
3. **Unit-Scale & Transform Corruption Recovery:**
   - *Cloud Coverage Fractional Scaling:* A subset of records stored `cloud_coverage_percent` as a decimal fraction in $[0, 1]$ instead of $[0, 100]$. Values $\le 1.0$ were rescaled by $\times 100$ into a unified percentage scale.
   - *Axial Tilt Normalization:* Certain records encoded `axial_tilt_deg` as a fraction of Earth's obliquity ($\le 1.0$) rather than degrees. Values $\le 1.0$ were unified via $\times 23.5^\circ$.
   - *Logarithmic Orbital Period Corruption:* A subset of non-rogue planets recorded `orbital_period_days` in natural-log units (0 < P ≤ 10) rather than linear days. We unified these records via $\exp(P)$ while maintaining a logarithmic transformation across all planets for scale invariance.
4. **Astrophysical Zero-Constraints for Rogue Worlds:**
   - Missing `orbital_distance_au` and `orbital_period_days` values on `rogue planet` records are physically meaningful (unbound to any host star) rather than random missing data; these were deterministically set to 0.0.
5. **Group-Conditional Median Imputation:**
   - Remaining missing continuous variables (`orbital_distance_au`, `orbital_period_days`, `day_length_hours`, `axial_tilt_deg`, `escape_velocity_kms`) were imputed using the median of each planet's canonical `planet_type` cohort.

---

### 2.3 Findings from the Five Core Research Questions

#### Q1: How does the recorded primary hazard relate to settlement potential?
Primary environmental hazards exert one of the strongest influences on EDEN's settlement index:
- **Top-Performing Hazard Groups:**
  - **None (Hazard-Free):** Mean score = **83.9** (100% viable at $\ge 60$; $n = 9$)
  - **Seasonal Storms:** Mean score = **79.1** (100% viable; $n = 3$)
  - **Thermal Vents:** Mean score = **73.7** (100% viable; $n = 6$)
  - **Unknown Anomaly:** Mean score = **63.9** (56% viable; $n = 1,315$)
- **Lowest-Performing Hazard Groups:**
  - **Temperature Extremes:** Mean score = **54.8** (29% viable; $n = 41$)
  - **Radiation:** Mean score = **55.5** (30% viable; $n = 1,241$)
  - **Toxic Air:** Mean score = **56.1** (31% viable; $n = 829$)
  - **Seismic Activity:** Mean score = **57.2** (37% viable; $n = 1,392$)
- **Conclusion:** Worlds free of major hazards or experiencing predictable, localized phenomena (such as seasonal storms or geothermal vents) achieve high settlement viability. Conversely, pervasive life-threatening conditions  specifically thermal extremes, ionizing radiation, and toxic atmospheres  depress viability rates below 32%.

#### Q2: How do atmospheric gas levels relate to settlement potential?
Atmospheric composition follows a non-linear physiological optimality curve rather than a monotonic trend:
- **Oxygen (O₂):** Worlds within the Earth-like breathable band of **20–25% O₂** achieve the highest average settlement score of **66.9**, compared to just **53.7** for anoxic worlds (0–1% O₂). Increasing oxygen above 25% yields diminishing returns and increased combustion risk without further boosting scores.
- **Carbon Dioxide (CO₂):** High CO₂ concentrations (>70%) drop the average settlement index to **54.2**, reflecting runaway greenhouse heating and hypercapnic toxicity. Low-to-moderate CO₂ bands perform substantially better.
- **Methane (CH₄):** Atmospheres dominated by methane (>50%) average **53.2**, whereas trace or low-methane worlds score significantly higher.
- **Conclusion:** Habitability depends on a balanced gas mixture - specifically moderate O₂ paired with low asphyxiant/greenhouse gas loads (CO₂ and CH₄).

#### Q3: How does stellar multiplicity relate to settlement potential?
Grouping planetary systems by host star configuration reveals a consistent stability gradient:

| Stellar System Architecture | Planet Count ($n$) | Mean Settlement Index | Viable Share ($\text{Index} \ge 60$) |
| :--- | :---: | :---: | :---: |
| **Single-Star Systems** | 4,010 | **60.1** | **45.0%** |
| **Binary-Star Systems** | 1,841 | 58.1 | 39.0% |
| **Triple-Star Systems** | 306 | 56.7 | 33.0% |
| **Other / Complex Companion** | 243 | 55.7 | 29.0% |

- **Conclusion:** Single-star systems provide the highest average settlement score (60.1) and highest viability rate (45%). Multi-star architectures introduce gravitational perturbations and complex radiative flux cycles that moderately reduce settlement viability.

#### Q4: How do moons and rings relate to settlement potential?
- **Moon Count:** Planets orbited by **one moon** achieve the highest well-supported average score (**62.2**), followed closely by **two moons** (**60.7**), whereas moonless worlds average **56.6** (consistent with the stabilizing effect of a large moon on planetary axial obliquity). However, across the entire range of moon counts, the linear correlation with settlement index is weak ($r = -0.09$), fluctuating erratically at higher moon counts where sample sizes are small.
- **Planetary Rings:** Worlds **without rings** average **59.4**, outscoring **ringed planets** (**56.8**) by 2.6 points.
- **Conclusion:** While having 1–2 moons and no ring system is mildly favorable, both features are secondary modifiers compared to atmospheric breathability, water, and radiation hazards.

#### Q5: How do rogue and tidally locked planets compare with other planets?
Comparing rotationally or orbitally constrained worlds against standard orbiting planets highlights clear environmental penalties:

| Planetary Group | Planet Count ($n$) | Mean Settlement Index | Viable Share ($\text{Index} \ge 60$) |
| :--- | :---: | :---: | :---: |
| **Standard Orbiting Planets** | 5,127 | **59.8** | **44.0%** |
| **Tidally Locked Planets** | 736 | 56.6 | 37.1% |
| **Rogue Planets** | 537 | 56.7 | 33.0% |

- **Conclusion:** Both rogue planets (which lack stellar insolation and rely solely on internal geothermal/radiogenic heating) and tidally locked worlds (which experience permanent scorching day sides and frozen night sides) score approximately 3.1–3.2 points lower on average and exhibit 7–11% lower viability rates than standard worlds. Nevertheless, roughly one-third of these worlds still achieve viability ($\ge 60$) when compensated by thick atmospheres, subsurface oceans, or geothermal energy.

---

### 2.4 Independent Investigation: Surface Hydrology as a Primary Screening Criterion

#### Research Question & Hypothesis
*How strongly does the physical state and accessibility of water relate to planetary settlement potential?*  
We hypothesized that **accessible surface liquid water** (`global ocean` and `seas`) provides a massive positive shift in settlement viability compared to sequestered (`subsurface`), frozen (`surface ice`), or absent (`none`) water regimes.

#### Empirical Findings
Stratifying the cleaned archive across hydrological categories revealed one of the largest effect sizes in the entire dataset:

| Water Availability State | Mean Settlement Index | Viable Share ($\text{Index} \ge 60$) | Hydrological Regime |
| :--- | :---: | :---: | :--- |
| **Global Ocean** | **65.4** | **65.0%** | Abundant accessible liquid surface water |
| **Surface Seas** | **63.8** | **62.0%** | Regional accessible liquid surface water |
| **No Recorded Water (`None`)** | 50.5 | 11.0% | Desiccated / arid lithosphere |
| **Surface Ice Only** | 47.9 | 10.0% | Cryogenic locked water |

- **Effect Size:** Worlds with a **global ocean** outscore dry worlds (`none`) by **$+14.9\text{ points}$** (65.4 vs. 50.5) and are **nearly 6× more likely to be viable for settlement** (65% vs. 11%).
- **Conclusion:** Accessible liquid surface water acts as a foundational prerequisite for both biological sustainability and climate regulation (via thermal inertia). While high radiation or toxic gases can still render an ocean world unviable, hydrological accessibility serves as the single most effective first-pass screening filter for colonizable worlds.

---

## Bonus Task: Rebuilding EDEN

### 3.1 Problem Formulation & Target Structure
To rebuild EDEN's lost decision engine, we trained a predictive machine learning pipeline on the 6,400 labeled planets to classify the 1,600 unseen test worlds into binary settlement viability (`viable_planet`):

$$\text{viable} = \mathbb{I}(\text{settlement index} \ge 60.0)$$

In the training archive, **2,708 planets (42.31%)** are viable (`True` / 1) and **3,692 planets (57.69%)** are non-viable (`False` / 0).

Rather than discarding the continuous `settlement_index` values during training by treating the problem purely as binary classification, we formulated a **Dual-Objective Learning Paradigm**:
1. **Regression Branch:** Models trained directly on the continuous `settlement_index` preserve fine-grained distance-to-threshold information (e.g., distinguishing a borderline planet with index 59.8 from a hostile world with index 25.0).
2. **Classification Branch:** Models trained directly on binary log-loss focus sharp decision boundaries specifically around the critical index = 60.0 threshold.

---

### 3.2 Domain Feature Engineering (176 Features)

We engineered **176 features** spanning eight scientific and statistical domains while enforcing strict data-leakage prevention:

1. **Astrophysical & Planetary Interior Proxies:**
   - Using normalized escape velocity $\hat{v}_e = v_e / 11.186\text{ km/s}$ and surface gravity $g$ (in Earth $g$), we derived dimensionless planetary radius and mass proxies from Newtonian gravitation ($v_e = \sqrt{2GM/R}$ and $g = GM/R^2$):

$$R_{\text{proxy}} = \frac{\hat{v}_e^2}{g}, \qquad M_{\text{proxy}} = \frac{\hat{v}_e^4}{g}, \qquad \text{Ratio}_{v_e/g} = \frac{v_e}{g}$$

2. **Orbital Mechanics & Keplerian Consistency:**
   - Using orbital semi-major axis $a$ ($\text{AU}$) and unified orbital period $P_{\text{yr}}$ ($\text{years}$), we derived a host-star mass proxy via Kepler's Third Law ($M_\star \approx a^3 / P_{\text{yr}}^2$), expected Keplerian period $P_{\text{Kepler}} = 365.25 \cdot a^{1.5}$, and effective stellar insolation flux:

$$S_{\text{eff}} = \frac{1 - \text{Albedo}}{a^2 + 0.05}$$

3. **Atmospheric Partial Pressures & Physiological Ratios:**
   - By Dalton's Law, physiological gas toxicity and breathability depend on **partial pressure** ($P_i = P_{\text{atm}} \times f_i$) rather than volume percentage alone. We computed partial pressures P(O₂), P(CO₂), P(CH₄), and P(N₂) (in atm), total toxic gas sum (CO₂ + CH₄ + SO₂ + H₂S + NH₃ + CO), total inert buffer sum (N₂ + Ar + He + Ne), and a composite breathability ratio:

$$\text{Breathability Ratio} = \frac{\text{O}_2 \%}{\sum \text{Toxic Gas} \% + 1.0}$$

4. **Earth-Similarity Deviations & Habitability Window Flags:**
   - Absolute deviations from Earth baselines: |Temp - 15°C|, |g - 1.0|, |Patm - 1.0|, |%O₂ - 21.0%|, estimated diurnal temperature range ($T_{\text{mean}} \pm 0.5\Delta T$), and boolean indicators for liquid-water thermal bands (-10°C to 45°C), comfortable gravity (0.5–1.3g), comfortable pressure (0.4–2.2 atm), and breathable oxygen (12–26%).
5. **Composite Domain Indices:**
   - `shielding_score` (`2 * Magnetic_ord - Radiation_ord - Storm_ord - Seismic_ord`), `resource_score` (combining `rare_mineral_index`, `energy_ord`, and `water_ord`), and an analytical prior `eden_heuristic_index`.
6. **Non-Linear Physical Interactions & Group Aggregations:**
   - Multiplicative couplings (`O2 * g`, `O2 * axial_tilt`, `g * axial_tilt`, `O2 * water_ord`), high-order categorical interaction pairs (`star_planet_pair`, `water_surface_pair`), frequency encodings, and group-conditional Z-scores and mean-differences of key numerical variables across `planet_type`, `star_type`, and `water_presence`.
7. **Archive Corruption & Missingness Metadata:**
   - Binary missingness flags (`isna_*`), rounding precision indicators (`*_is_round2`), and unit-regime flags (`cloud_is_fractional`, `orbital_period_is_logged`, `axial_tilt_gt1`) to capture structural sub-populations in EDEN's corrupted database.
8. **Out-of-Fold Bayesian Smoothed Target Encoding:**
   - Computed strictly **inside each cross-validation fold** (with smoothing parameter $m = 15$) across all 14 categorical features for both continuous (`*_te_reg`) and binary (`*_te_cls`) targets, preventing any target leakage into validation predictions.

---

### 3.3 Model Architecture: 12-Model Dual-Objective Stacking Ensemble

We implemented a **10-Fold Stratified Cross-Validation** scheme (`RANDOM_SEED = 42`) training **6 diverse algorithm families $\times$ 2 objectives = 12 Level-1 Base Models**:

1. **CatBoost (`cat_reg`, `cat_cls`):** Symmetric oblivious trees (`depth=6`, `iterations=2500`, `lr=0.03`, `l2_leaf_reg=4.5`) leveraging native ordered target statistics on all 14 categorical features.
2. **LightGBM (`lgb_reg`, `lgb_cls`):** Leaf-wise gradient boosted trees (`num_leaves=31`, `max_depth=6`, `lr=0.02`, `colsample_bytree=0.72`, `subsample=0.8`, $\text{L1}=1.2, \text{L2}=2.5$).
3. **XGBoost (`xgb_reg`, `xgb_cls`):** Depth-wise histogram GBDTs (`max_depth=5`, `lr=0.02`, `min_child_weight=22`, native experimental categorical splitting enabled).
4. **Scikit-Learn HistGradientBoosting (`hgb_reg`, `hgb_cls`):** Fast binned histogram trees (`max_leaf_nodes=31`, `max_depth=6`, `l2_regularization=2.5`) operating on numeric and OOF target-encoded representations.
5. **ExtraTrees (`et_reg`, `et_cls`):** Randomized bagging ensembles (`n_estimators=300`, `max_depth=16`, `max_features=0.65`) providing high variance reduction and uncorrelated split diversity.
6. **Multi-Layer Perceptron Neural Networks (`mlp_reg`, `mlp_cls`):** Deep feed-forward neural architecture (`hidden_layer_sizes=(128, 64)`, ReLU activations, $\text{L2 }\alpha=2.0$, inputs scaled via outlier-resistant `RobustScaler` clipped to $[-8, 8]$).

#### Level-2 Meta-Ensembling & Calibration
- **Meta-Regressor (`RidgeCV`):** Combines out-of-fold predictions from the 6 base regressors using L2-regularized linear stacking ($\alpha \in [10^{-1}, 10^4]$).
- **Meta-Classifier (`LogisticRegressionCV`):** Stacks the 6 base classifier probabilities alongside sigmoid-calibrated probabilities from the 6 base regressors:

$$p_{\text{reg}, k} = \sigma\!\left(\frac{\hat{y}_{\text{reg}, k} - 60.0}{2.2}\right), \quad \text{where } \sigma(z) = \frac{1}{1 + e^{-z}}$$

- **Optimal Dual-Branch Blending:** Finally, the meta-regressor prediction $\hat{y}_{\text{reg}}$ and meta-classifier probability $p_{\text{cls}}$ are combined via a prior-regularized grid search over blend weight $\alpha$, temperature scale $s$, and classification threshold $\tau$:

$$P(\text{viable}) = \alpha \cdot \sigma\!\left(\frac{\hat{y}_{\text{reg}} - 60.0}{s}\right) + (1 - \alpha) \cdot p_{\text{cls}}$$

---

### 3.4 Model Performance & Validation Results

#### Out-of-Fold (OOF) Performance Across All 12 Base Models & Meta-Ensemble

| Model Identifier | Algorithm Family & Objective | OOF RMSE | OOF $R^2$ | OOF Accuracy | OOF F1-Score | OOF ROC-AUC |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| `cat_reg` | CatBoost Regressor | **4.3530** | **0.8369** | 88.625% | - | 0.94739 |
| `hgb_reg` | HistGradientBoosting Regressor | 4.4998 | 0.8257 | 87.812% | - | 0.94375 |
| `lgb_reg` | LightGBM Regressor | 4.5238 | 0.8238 | 87.812% | - | 0.94277 |
| `et_reg` | ExtraTrees Regressor | 4.8142 | 0.8005 | 86.500% | - | 0.93378 |
| `xgb_reg` | XGBoost Regressor | 4.8892 | 0.7942 | 87.016% | - | 0.93676 |
| `mlp_reg` | Neural MLP Regressor | 5.2004 | 0.7672 | 85.219% | - | 0.92637 |
| `cat_cls` | CatBoost Classifier | - | - | 88.141% | 0.8551 | 0.94474 |
| `xgb_cls` | XGBoost Classifier | - | - | 87.703% | 0.8501 | 0.94050 |
| `lgb_cls` | LightGBM Classifier | - | - | 87.484% | 0.8466 | 0.93898 |
| `hgb_cls` | HistGradientBoosting Classifier | - | - | 87.266% | 0.8447 | 0.93792 |
| `et_cls` | ExtraTrees Classifier | - | - | 86.141% | 0.8287 | 0.92897 |
| `mlp_cls` | Neural MLP Classifier | - | - | 86.047% | 0.8303 | 0.92845 |
| **Meta-Regressor** | `RidgeCV` Stack (6 Regressors) | 4.3533 | 0.8368 | - | - | - |
| **Meta-Classifier** | `LogisticRegressionCV` Stack | - | - | - | - | 0.94719 |
| **Final V2 Meta-Ensemble** | **Dual-Objective Calibrated Blend** | **4.3533** | **0.8368** | **88.8438%** | **0.8643** | **0.94748** |

#### Key Observations from Model Evaluation
1. **Regression Outperforms Pure Classification:** Across nearly every model family, training on the continuous `settlement_index` and thresholding at 60.0 outperformed training solely on binary labels (e.g., `cat_reg` achieved 88.625% accuracy and 0.94739 AUC vs. `cat_cls` at 88.141% accuracy and 0.94474 AUC). Consequently, the optimal meta-blend assigned **$\alpha = 0.80$ weight to the regression stack** (with sigmoid temperature scale $s = 1.8$) and **0.20 weight to the classification stack** at decision threshold $\tau = 0.5240$.
2. **Confusion Matrix & Error Balance:**
   On the 6,400 out-of-fold validation predictions, the final ensemble produced:

   | | Predicted Non-Viable (0) | Predicted Viable (1) |
   | :--- | :---: | :---: |
   | **Actual Non-Viable (0)** | **TN = 3,412** | **FP = 280** |
   | **Actual Viable (1)** | **FN = 434** | **TP = 2,274** |

   - **Specificity (True Negative Rate):** 92.42% (3,412 / 3,692) - High protection against false-positive colonization of hostile worlds.
   - **Precision (Positive Predictive Value):** 89.04% (2,274 / 2,554) - Nearly 9 out of 10 worlds recommended by our model are genuinely viable.
   - **Sensitivity (Recall):** 83.97% (2,274 / 2,708).
3. **Unseen Test Set Predictions (`The_gentlemen_predicted.csv`):**
   - Applied to the 1,600 unseen test planets, the ensemble predicted **653 viable worlds (40.81%)** and **947 non-viable worlds (59.19%)**, closely matching the true training prior of 42.31% and OOF positive rate of 39.91%.

