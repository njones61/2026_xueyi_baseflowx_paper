# Pybaseflow Manuscript Refactoring Plan

## Overview

This plan covers the refactoring of `pybaseflow_manuscript_v06.tex` for submission to **MDPI Hydrology** as an **Article** (or optionally a **Technical Note**). The paper presents the `pybaseflow` Python package for baseflow separation. The package underwent an extensive overhaul in March 2026, and the manuscript must be updated accordingly.

---

## 1. MDPI Hydrology Guidelines Summary

- **Article type**: "Article" (standard research paper) or "Technical Note" (MDPI explicitly states Technical Notes can describe "a new software tool or computational method"). Most Python package papers in MDPI Hydrology/Water use the Article type.
- **Structure**: MDPI uses IMRAD but published software papers adapt it freely (see examples below).
- **Required elements**: Abstract, Keywords, Data Availability Statement, Author Contributions, Funding, Conflicts of Interest.
- **Code availability**: Must provide repository link; archiving with DOI (Zenodo) is strongly encouraged.
- **No dedicated software paper template** -- authors adapt the standard template.

---

## 2. How Other Python Package Papers Are Structured (MDPI)

### Hydrostats (MDPI Hydrology, 2018) -- 11 pages
1. Introduction
2. Hydrostats Python Package (Background, Development Methods, Use, then one subsection per module)
3. Conclusions

### Ratingcurve (MDPI Hydrology, 2024) -- 9 pages
1. Introduction
2. Parameterization (math)
3. Calibration (methodology)
4. Usage (examples)
5. Benchmarking Results
6. Conclusions

### swmm_api (MDPI Water, 2025) -- 18 pages
1. Introduction
2. Design and Functionality (subsections per module, plus dependencies, docs, limitations)
3. Example Applications
4. Benchmarks
5. Conclusions

### Common pattern across all successful software papers:
- Introduction with **statement of need** and comparison to existing tools
- Software design organized **by module/capability**, not by IMRAD
- **Usage examples** with real data
- **Validation or benchmarking**
- Explicit mention of **limitations and future work**
- **Code/data availability**

---

## 3. Current Manuscript Problems

### Structural Issues
- [ ] Conclusions (Sec 4) appears **before** Discussion (Sec 5) -- must be reversed or merged
- [ ] Discussion is only 2 paragraphs (just URLs) -- essentially empty
- [ ] Section 3 titled "Examples and Discussion" but contains only examples
- [ ] No statement of need or comparison with existing tools in the Introduction

### Content Gaps
- [ ] Only Chapman method is demonstrated in detail; 16 other methods are merely listed
- [ ] No quantitative validation or benchmarking
- [ ] No description of the `estimate` module (recession coefficient calculation)
- [ ] No mention of the generalized recursive digital filter framework (key intellectual contribution of the overhaul)
- [ ] Utility functions section describes functions that no longer exist (`geo2imagexy`, `exist_ice`, `kge`, `format_method`)

### Package Overhaul Changes Not Reflected
- [ ] Package renamed from `baseflow` to `pybaseflow`
- [ ] 4 new methods added: PART, CMB (tracer-based), IHACRES, BFlow
- [ ] New modules: `tracer.py`, `io.py` (USGS NWIS retrieval)
- [ ] Unified `_recursive_digital_filter()` core with generalized equation
- [ ] Removed: geospatial functions, `thawed.npz`, `flow_duration_curve`, `kge`, batch orchestrators
- [ ] Modern packaging (`pyproject.toml` replaces `setup.py`)
- [ ] Bundled sample data (`fish_river.csv`) with `load_sample_data()`
- [ ] New documentation site (MkDocs)
- [ ] `HydRun` removed (was duplicate of `lh_multi`)

### Reference Issues
- [ ] Several citations in Table 1 point to wrong papers (ref13, ref17, ref18 for HYSEP methods)
- [ ] Duplicate reference: ref15 and ref22 are both Ladson (2013)
- [ ] Missing references for new methods (PART, CMB, IHACRES, BFlow)
- [ ] Need to add references for: Rutledge (1998), Stewart et al. (2007), Jakeman & Hornberger (1993), Arnold & Allen (1999)

### Figure Issues
- [ ] Figures 1-2 are screenshots of code -- should use LaTeX `lstlisting` instead
- [ ] Figures 5-7 are Colab UI screenshots -- low value for a journal paper
- [ ] No comparative visualization across methods
- [ ] No figure showing the method taxonomy / generalized filter framework
- [ ] Duplicate image files in `figures/` directory (image1-11.png alongside fig1-9b.png)

---

## 4. Proposed New Paper Structure

Based on the successful patterns from Hydrostats, Ratingcurve, and swmm_api papers, the recommended structure is:

### Section 1: Introduction
- Baseflow definition and importance in hydrology
- Overview of separation approaches (digital filters, graphical, recession, tracer-based)
- **Statement of need**: why a unified Python package? Gap analysis vs existing tools:
  - Original `baseflow` package (Xie et al., 2020) -- research code, not maintained as a general tool
  - `hydrosep` (R) -- limited to a few methods
  - WHAT (web tool) -- not scriptable
  - USGS HYSEP/PART -- Fortran, not easily integrated into modern workflows
- What pybaseflow offers: 17 methods across 4 paradigms, unified API, open source, pip-installable

### Section 2: Theoretical Framework
- The generalized recursive digital filter equation: `b[t] = alpha*b[t-1] + beta*(Q[t] + gamma*Q[t-1])`, with `b[t] <= Q[t]`
- **Table**: Show how each method maps to specific alpha, beta, gamma values (this is the key insight from the overhaul)
- Brief mathematical description of each paradigm:
  - 2.1 Recursive digital filters (gamma=0 family): Chapman-Maxwell, Boughton, Eckhardt, EWMA, Furey-Gupta, WHAT
  - 2.2 Recursive digital filters (gamma=1 family): Lyne-Hollick, Chapman, Willems
  - 2.3 Hybrid filters (variable gamma): IHACRES
  - 2.4 Graphical methods: Fixed interval, Sliding interval, Local minimum (HYSEP), UKIH, PART
  - 2.5 Tracer-based methods: CMB
  - 2.6 Recession analysis: BFlow, BN77, strict_baseflow

### Section 3: Software Design and Implementation
- 3.1 Package architecture (module overview diagram)
  - `separation.py` -- all separation methods
  - `estimate.py` -- recession coefficient estimation, parameter calibration
  - `tracer.py` -- CMB and Eckhardt calibration bridge
  - `io.py` -- USGS NWIS data retrieval
  - `utils.py` -- preprocessing utilities
- 3.2 Unified filter core (`_recursive_digital_filter`)
- 3.3 Performance optimization (Numba JIT compilation)
- 3.4 Data input and preprocessing
  - `fetch_usgs()` for NWIS API access
  - `clean_streamflow()` for quality control
  - Bundled sample data
- 3.5 Dependencies and installation
- 3.6 Documentation (MkDocs site, API reference, method guides)

### Section 4: Usage Examples
- 4.1 Quick start: loading data and running a separation
- 4.2 Comparing multiple methods on the same dataset
- 4.3 Parameter estimation (recession coefficient, BFImax calibration)
- 4.4 Tracer-based separation and CMB-to-Eckhardt calibration bridge
- Use typeset code listings (not screenshots)
- Include publication-quality figures from the docs/assets

### Section 5: Validation and Benchmarking
- 5.1 Cross-method comparison on representative catchments
- 5.2 Comparison with published results (e.g., reproduce results from Eckhardt 2005, Sloto & Crouse 1996)
- 5.3 Verification against USGS PART/HYSEP outputs (if feasible)
- Include quantitative metrics (BFI values, correlation between methods)

### Section 6: Discussion
- Comparison with existing tools and packages
- Strengths: unified framework, breadth of methods, modern Python ecosystem integration
- Limitations: daily timestep only, single-station focus, parameter uncertainty
- Future work: automated method selection, spatial baseflow mapping, additional tracer methods

### Section 7: Conclusions
- Summary of contributions
- Availability statement

### Data Availability Statement
- GitHub repository URL
- ReadTheDocs documentation URL
- Zenodo DOI (create archive before submission)
- Sample data provenance

---

## 5. Action Items by Priority

### Phase 1: Update content to match overhauled package (CRITICAL)
1. Rename all occurrences of `baseflow` package to `pybaseflow`
2. Update Table 1 (separation methods): add PART, CMB, IHACRES, BFlow; remove HydRun
3. Remove Table 2 (utility functions) or replace with updated functions -- remove `geo2imagexy`, `exist_ice`, `kge`, `format_method`; add `fetch_usgs`, `load_sample_data`
4. Add description of the generalized digital filter equation and alpha/beta/gamma mapping table
5. Add content for new modules: `tracer.py`, `io.py`, `estimate.py`
6. Fix all incorrect citations in Table 1
7. Add missing references for new methods

### Phase 2: Restructure the paper
8. Adopt the new section structure (see Section 4 above)
9. Move/rewrite Introduction to include statement of need and gap analysis
10. Create a proper Theoretical Framework section
11. Merge old Sections 4-5 into a proper Discussion + Conclusions flow
12. Add a Validation/Benchmarking section (even if preliminary)

### Phase 3: Improve figures and code presentation
13. Replace screenshot figures (fig1, fig2) with LaTeX `lstlisting` code blocks
14. Remove Colab UI screenshots (fig5, fig6, fig7) -- replace with more informative figures
15. Add a method taxonomy diagram (the 4-paradigm classification)
16. Add multi-method comparison figure(s) -- use the publication-quality figures from `docs/assets/figures/`
17. Clean up `figures/` directory (remove duplicate `image*.png` files)

### Phase 4: Polish
18. Expand the abstract to reflect the overhauled package
19. Update keywords: add "digital filter", "recession analysis", "tracer", "USGS", "open source"
20. Fix duplicate reference (ref15/ref22 both Ladson 2013)
21. Convert bibliography to `.bib` file for cleaner management
22. Add ORCID IDs for all authors
23. Create Zenodo archive and add DOI
24. Proofread for consistency in package naming and terminology

---

## 6. Estimated Scope

| Item | Effort |
|------|--------|
| Content updates for package overhaul | Major -- new tables, new sections, new references |
| Paper restructuring | Major -- rewrite/reorganize most sections |
| New figures | Moderate -- can reuse figures from docs/assets |
| Validation section | Moderate-to-Major -- depends on depth of benchmarking |
| Reference cleanup | Minor |
| Polish and formatting | Minor |

Target length: **12-18 pages** (consistent with swmm_api at 18pp and Hydrostats at 11pp).

---

## 7. Reusable Assets from pybaseflow `docs/` Folder

The overhauled package documentation at `https://github.com/BYU-Hydroinformatics/pybaseflow/tree/master/docs` contains publication-quality figures, fully formatted equations, complete citations, and well-written narrative prose that can be directly adapted for the paper. This significantly reduces the writing effort.

### 7.1 Figures (21 publication-quality PNGs in `docs/assets/figures/`)

All generated from `docs/generate_figures.py` using Fish River (USGS 01013500, 2019-2020) sample data.

| # | Filename | Content | Suggested Paper Section |
|---|----------|---------|------------------------|
| 1 | `overview_comparison.png` | All 17 methods applied simultaneously with BFI annotations | Sec 4 (Examples) or Sec 5 (Validation) -- the "hero" figure |
| 2 | `filter_families.png` | gamma=0 vs gamma=1 families side-by-side, zoomed to a peak | Sec 2 (Theoretical Framework) -- illustrates taxonomy |
| 3 | `eckhardt.png` | Eckhardt filter: streamflow + baseflow fill + quickflow | Sec 2.1 (Digital Filters) |
| 4 | `chapman_maxwell.png` | Chapman-Maxwell filter separation | Sec 2.1 |
| 5 | `chapman.png` | Chapman (1991) filter separation | Sec 2.1 |
| 6 | `boughton.png` | Boughton filter separation | Sec 2.1 |
| 7 | `ewma.png` | EWMA filter separation | Sec 2.1 |
| 8 | `furey.png` | Furey-Gupta physically-based filter | Sec 2.1 |
| 9 | `willems.png` | Willems filter separation | Sec 2.1 |
| 10 | `ihacres.png` | IHACRES filter separation | Sec 2.3 (Hybrid Filters) |
| 11 | `ihacres_sensitivity.png` | IHACRES sensitivity to alpha_s (0, -0.3, -0.6, -0.9) showing transition between families | Sec 2.3 or Discussion -- key insight figure |
| 12 | `lyne_hollick_passes.png` | LH filter with 1, 2, 3 passes overlaid | Sec 2.2 (gamma=1 family) |
| 13 | `bflow.png` | BFlow: 3 LH passes + alpha factor + baseflow days | Sec 2.6 (Recession Analysis) |
| 14 | `fixed_interval.png` | HYSEP fixed interval -- staircase pattern | Sec 2.4 (Graphical Methods) |
| 15 | `sliding_interval.png` | HYSEP sliding interval -- smooth envelope | Sec 2.4 |
| 16 | `local_minimum.png` | HYSEP local minimum -- turning points + interpolation | Sec 2.4 |
| 17 | `ukih.png` | UKIH smoothed minima with 0.9 test | Sec 2.4 |
| 18 | `part.png` | PART recession-based method -- anchor points + log-space interpolation | Sec 2.4 |
| 19 | `cmb.png` | Two-panel: (top) CMB baseflow separation, (bottom) specific conductance with end-member lines | Sec 2.5 (Tracer Methods) |
| 20 | `recession_coefficient.png` | Strict baseflow days highlighted + fitted recession coefficient | Sec 3 (Parameter Estimation) |
| 21 | `bn77.png` | Brutsaert-Nieber drought flow points on hydrograph | Sec 2.6 |

**Recommendation for the paper**: Not all 21 figures should go in the paper. Suggested selection (8-10 figures):
1. `overview_comparison.png` -- multi-method comparison (essential)
2. `filter_families.png` -- taxonomy illustration (essential)
3. One representative per paradigm: `eckhardt.png` (gamma=0), `lyne_hollick_passes.png` (gamma=1), `ihacres_sensitivity.png` (hybrid bridge), `part.png` (graphical), `cmb.png` (tracer)
4. `recession_coefficient.png` -- parameter estimation
5. A new figure: package architecture/module diagram (create for Sec 3)

### 7.2 Equations (ready to paste into LaTeX)

All equations below are documented in the `docs/methods/` markdown files with MathJax and can be converted directly to LaTeX.

**Core framework** (from `overview.md`, `digital-filters.md`):
- Generalized recursive digital filter: $b_t = \alpha\,b_{t-1} + \beta(Q_t + \gamma\,Q_{t-1})$, $b_t \leq Q_t$

**Digital filter coefficients** (from `digital-filters.md`) -- each method's alpha, beta, gamma mapping:

| Method | alpha | beta | gamma | Source param |
|--------|-------|------|-------|-------------|
| Chapman-Maxwell | $\frac{a}{2-a}$ | $\frac{1-a}{2-a}$ | 0 | recession coeff $a$ |
| Boughton | $\frac{a}{1+C}$ | $\frac{C}{1+C}$ | 0 | $a$, calibration $C$ |
| Eckhardt | $\frac{(1-\text{BFI}_\text{max})a}{1-a\cdot\text{BFI}_\text{max}}$ | $\frac{(1-a)\text{BFI}_\text{max}}{1-a\cdot\text{BFI}_\text{max}}$ | 0 | $a$, $\text{BFI}_\text{max}$ |
| EWMA | $1-e$ | $e$ | 0 | smoothing $e$ |
| Furey-Gupta | $a - A(1-a)$ | $A(1-a)$ | 0 (lagged) | $a$, aquifer $A$ |
| Lyne-Hollick | $\beta_p$ | $\frac{1-\beta_p}{2}$ | 1 | filter $\beta_p$ |
| Chapman (1991) | $\frac{3a-1}{3-a}$ | $\frac{1-a}{3-a}$ | 1 | recession coeff $a$ |
| Willems | $\frac{a-v}{1+v}$ | $\frac{v}{1+v}$ | 1 | $a$, quickflow frac $w$ |
| IHACRES | $\frac{a}{1+C}$ | $\frac{C}{1+C}$ | $\alpha_s$ | $a$, $C$, slow-flow $\alpha_s$ |

**Key algebraic relationships** (from `digital-filters.md`):
- Boughton-Eckhardt equivalence: $C = \frac{(1-a)\cdot\text{BFI}_\text{max}}{1-\text{BFI}_\text{max}}$
- IHACRES reduces to Boughton when $\alpha_s = 0$, approaches gamma=1 family as $\alpha_s \to -1$

**Graphical methods** (from `graphical-methods.md`):
- HYSEP drainage area: $N = A^{0.2}$ (A in sq miles)
- Sliding interval: $b_t = \min_{j \in [t-(N^*-1)/2,\, t+(N^*-1)/2]} Q_j$
- UKIH turning point test: $0.9\,Q_{\min,i} < Q_{\min,i-1}$ and $0.9\,Q_{\min,i} < Q_{\min,i+1}$
- PART log-space interpolation: $\log_{10}(b_t) = \log_{10}(b_{t_0}) + \frac{t-t_0}{t_1-t_0}[\log_{10}(b_{t_1}) - \log_{10}(b_{t_0})]$

**Recession analysis** (from `recession-analysis.md`):
- BFlow exponential recession: $Q_b(t) = Q_b(0)\cdot e^{-\alpha t}$; baseflow days $= \ln(10)/\alpha$
- Brutsaert-Nieber recession slope: $S(t) = \frac{Q(t-1)-Q(t+1)}{2}$

**Tracer methods** (from `tracer-methods.md`):
- CMB mass balance: $Q(t)\cdot SC(t) = Q_b(t)\cdot SC_{BF} + Q_r(t)\cdot SC_{RO}$
- Solved: $Q_b(t) = Q(t)\cdot\frac{SC(t) - SC_{RO}}{SC_{BF} - SC_{RO}}$

### 7.3 Complete Citation List from Docs (21 references)

These are extracted from the docs narrative and can be used to build the `.bib` file. References marked **(NEW)** are not in the current manuscript.

1. Arnold, J.G. and Allen, P.M. (1999). Automated methods for estimating baseflow and ground water recharge from streamflow records. *JAWRA*, 35(2), 411--424. **(NEW -- BFlow)**
2. Barnes, B.S. (1939). The structure of discharge-recession curves. *Trans. AGU*, 20(4), 721--725. **(NEW -- PART threshold origin)**
3. Boughton, W.C. (1993). A hydrograph-based model for estimating water yield of ungauged catchments. *Inst. Eng. Aust.*, 93/14, 317--324.
4. Chapman, T.G. (1991). Comment on "Evaluation of automated techniques for base flow and recession analyses." *WRR*, 27(7), 1783--1784.
5. Chapman, T.G. and Maxwell, A.I. (1996). Baseflow separation -- comparison of numerical methods with tracer experiments. *Hydrol. Water Resour. Symp.*, Hobart, 539--545.
6. Cheng, L., Zhang, L. and Brutsaert, W. (2016). Automated selection of pure base flows from regular daily streamflow data. *J. Hydrol. Eng.*, 21(11), 06016008. **(NEW -- bn77)**
7. Eckhardt, K. (2005). How to construct recursive digital filters for baseflow separation. *Hydrol. Processes*, 19, 507--515.
8. Furey, P.R. and Gupta, V.K. (2001). A physically based filter for separating base flow from streamflow time series. *WRR*, 37(11), 2709--2722.
9. Jakeman, A.J. and Hornberger, G.M. (1993). How much complexity is warranted in a rainfall-runoff model? *WRR*, 29(8), 2637--2649. **(NEW -- IHACRES)**
10. Lim, K.J. et al. (2005). Automated web GIS based hydrograph analysis tool, WHAT. *JAWRA*, 41(6), 1407--1416.
11. Lyne, V. and Hollick, M. (1979). Stochastic time-variable rainfall-runoff modelling. *Inst. Eng. Aust. Natl. Conf.*, 89--93.
12. Nathan, R.J. and McMahon, T.A. (1990). Evaluation of automated techniques for base flow and recession analyses. *WRR*, 26(7), 1465--1473. **(NEW -- LH parameterization)**
13. Rutledge, A.T. (1998). Computer programs for describing the recession of ground-water discharge... *USGS WRIR 98-4148*. **(NEW -- PART)**
14. Sloto, R.A. and Crouse, M.Y. (1996). HYSEP: A computer program for streamflow hydrograph separation and analysis. *USGS WRIR 96-4040*.
15. Spongberg, M.E. (2000). Spectral analysis of base flow separation with digital filters. *WRR*, 36(3), 745--752. **(NEW -- multi-pass LH)**
16. Stewart, M.T., Cimino, J. and Ross, M. (2007). Calibration of base flow separation methods with streamflow conductivity. *Groundwater*, 45(1), 17--27. **(NEW -- CMB)**
17. Tularam, G.A. and Ilahee, M. (2008). Exponential smoothing method of base flow separation. *Am. J. Environ. Sci.*, 4(2), 136--144.
18. UKIH (1980). *Low Flow Studies*. Institute of Hydrology, Wallingford, UK. **(NEW)**
19. Willems, P. (2009). A time series tool to support multi-criteria performance evaluation. *Environ. Model. Softw.*, 24(3), 311--321.
20. Xie, J. et al. (2020). Evaluation of typical methods for baseflow separation in the contiguous United States. *J. Hydrology*, 583, 124628.
21. Zhang, J. et al. (2021). Can the two-parameter recursive digital filter really be calibrated by the conductivity mass balance method? *HESS*, 25, 1747--1760. **(NEW -- CMB calibration caveats)**

### 7.4 Docs-to-Paper Section Mapping

This table shows which doc files map to which proposed paper sections, so content can be adapted efficiently.

| Proposed Paper Section | Primary Doc Source | What to Reuse |
|------------------------|--------------------|---------------|
| **Sec 1: Introduction** | `index.md` (opening para, "Background") | Baseflow definition, 4-paradigm summary, package motivation |
| **Sec 2: Theoretical Framework** | `methods/overview.md` | Generalized filter equation, taxonomy, "Choosing a method" discussion |
| **Sec 2.1: Digital Filters (gamma=0)** | `methods/digital-filters.md` (first half) | Chapman-Maxwell, Boughton, Eckhardt, EWMA, Furey equations + prose |
| **Sec 2.2: Digital Filters (gamma=1)** | `methods/digital-filters.md` (second half) | Lyne-Hollick, Chapman, Willems equations + prose |
| **Sec 2.3: Hybrid Filters** | `methods/digital-filters.md` (IHACRES section) | IHACRES as bridge between families; alpha_s sensitivity |
| **Sec 2.4: Graphical Methods** | `methods/graphical-methods.md` | HYSEP (fixed/slide/local), UKIH, PART algorithms with full step-by-step |
| **Sec 2.5: Tracer Methods** | `methods/tracer-methods.md` | CMB mass balance derivation, end-member estimation, limitations |
| **Sec 2.6: Recession Analysis** | `methods/recession-analysis.md` | BFlow procedure, BN77 elimination criteria (C3-C9) |
| **Sec 3: Software Design** | `installation.md`, `api.md` | Dependencies, module structure, Numba strategy, Python version support |
| **Sec 3 (preprocessing)** | `guides/usgs-data.md` | fetch_usgs() usage, Fish River sample data rationale |
| **Sec 4: Usage Examples** | `guides/parameter-estimation.md` | Recession coefficient estimation, BFImax table (0.80/0.50/0.25) |
| **Sec 4 (CMB example)** | `guides/cmb-calibration.md` | 3-step CMB-to-Eckhardt calibration workflow |
| **Sec 6: Discussion** | `methods/overview.md` ("Choosing a method"), `methods/tracer-methods.md` (caveats) | Method trade-offs, CMB daily-vs-annual scale issue (Zhang et al. 2021: NSE 0.91-1.00 annually but -0.30 daily) |

### 7.5 Key Intellectual Contributions Documented in Docs (highlight in paper)

These insights from the docs deserve prominent placement in the paper as they represent the intellectual contribution beyond just wrapping methods:

1. **Generalized filter unification**: All 11 recursive digital filters are special cases of a single 3-parameter equation. This is implemented as a single `_recursive_digital_filter()` function; named methods are thin wrappers that compute their specific alpha/beta/gamma.

2. **Boughton-Eckhardt equivalence**: The docs prove algebraically that Eckhardt's filter is a reparameterization of Boughton's, with $C = \frac{(1-a)\cdot\text{BFI}_\text{max}}{1-\text{BFI}_\text{max}}$. This means Eckhardt subsumes Chapman-Maxwell (when $\text{BFI}_\text{max}=0.5$).

3. **IHACRES as a bridge**: With variable gamma ($\alpha_s$), IHACRES spans the continuum between the gamma=0 and gamma=1 filter families. The `ihacres_sensitivity.png` figure demonstrates this visually.

4. **CMB-to-Eckhardt calibration bridge**: Tracer-derived baseflow can calibrate the Eckhardt filter's BFImax parameter, linking physical measurements to digital filter parameters. The Zhang et al. (2021) caveat about daily-scale performance should be discussed.

5. **PART as first Python implementation**: The docs note this is the first Python implementation of Rutledge's (1998) PART method, previously only available in Fortran.

---

## 8. Full Reference List for New Manuscript

Combining current manuscript references (corrected) with new references from the docs:

1. Arnold, J.G. and Allen, P.M. (1999). Automated methods for estimating baseflow and ground water recharge from streamflow records. *JAWRA*, 35(2), 411--424.
2. Barnes, B.S. (1939). The structure of discharge-recession curves. *Trans. AGU*, 20(4), 721--725.
3. Boughton, W.C. (1993). A hydrograph-based model for estimating water yield of ungauged catchments. *Inst. Eng. Aust.*, 93/14, 317--324.
4. Chapman, T.G. (1991). Comment on "Evaluation of automated techniques for base flow and recession analyses." *WRR*, 27(7), 1783--1784.
5. Chapman, T.G. and Maxwell, A.I. (1996). Baseflow separation -- comparison of numerical methods with tracer experiments. *Hydrol. Water Resour. Symp.*, Hobart, 539--545.
6. Cheng, L., Zhang, L. and Brutsaert, W. (2016). Automated selection of pure base flows from regular daily streamflow data. *J. Hydrol. Eng.*, 21(11), 06016008.
7. Eckhardt, K. (2005). How to construct recursive digital filters for baseflow separation. *Hydrol. Processes*, 19, 507--515.
8. Furey, P.R. and Gupta, V.K. (2001). A physically based filter for separating base flow from streamflow time series. *WRR*, 37(11), 2709--2722.
9. Jakeman, A.J. and Hornberger, G.M. (1993). How much complexity is warranted in a rainfall-runoff model? *WRR*, 29(8), 2637--2649.
10. Lim, K.J. et al. (2005). Automated web GIS based hydrograph analysis tool, WHAT. *JAWRA*, 41(6), 1407--1416.
11. Lyne, V. and Hollick, M. (1979). Stochastic time-variable rainfall-runoff modelling. *Inst. Eng. Aust. Natl. Conf.*, 89--93.
12. Nathan, R.J. and McMahon, T.A. (1990). Evaluation of automated techniques for base flow and recession analyses. *WRR*, 26(7), 1465--1473.
13. Rutledge, A.T. (1998). Computer programs for describing the recession of ground-water discharge and for estimating mean ground-water recharge and discharge from streamflow records -- Update. *USGS WRIR 98-4148*.
14. Sloto, R.A. and Crouse, M.Y. (1996). HYSEP: A computer program for streamflow hydrograph separation and analysis. *USGS WRIR 96-4040*.
15. Spongberg, M.E. (2000). Spectral analysis of base flow separation with digital filters. *WRR*, 36(3), 745--752.
16. Stewart, M.T., Cimino, J. and Ross, M. (2007). Calibration of base flow separation methods with streamflow conductivity. *Groundwater*, 45(1), 17--27.
17. Tularam, G.A. and Ilahee, M. (2008). Exponential smoothing method of base flow separation. *Am. J. Environ. Sci.*, 4(2), 136--144.
18. UKIH (1980). *Low Flow Studies*. Institute of Hydrology, Wallingford, UK.
19. Willems, P. (2009). A time series tool to support multi-criteria performance evaluation. *Environ. Model. Softw.*, 24(3), 311--321.
20. Xie, J. et al. (2020). Evaluation of typical methods for baseflow separation in the contiguous United States. *J. Hydrology*, 583, 124628.
21. Zhang, J. et al. (2021). Can the two-parameter recursive digital filter really be calibrated by the conductivity mass balance method? *HESS*, 25, 1747--1760.
