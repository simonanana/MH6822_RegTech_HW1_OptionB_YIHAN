# MH6822 HW1 Designing a Jurisdiction-Aware RegTech Tool

**Name:** GUO YIHAN  
**Matriculation ID:** G2506255B  
**Email:** GUOY0065@ntu.edu.sg  

---

## Overview

**Option B — Architecture Design with Quantitative Demonstration**

This repository contains all deliverables for MH6822 Assignment 1. The project designs and partially implements a **Jurisdiction-Aware Credit Model Risk Governance Platform** — a RegTech tool that evaluates AI-driven credit-scoring models against two regulatory regimes simultaneously: the United States (OCC Bulletin 2026-13) and Singapore (MAS AI MRM Information Paper 2024, MAS AI Guidelines 2025, and the FEAT Principles with Veritas Toolkit v2.0).

**Regulated entity:** DBS Bank Limited  
**Domain:** AI Model Risk Management for Retail Credit Decisioning  
**Jurisdictions:** 🇺🇸 US (OCC 2026-13) × 🇸🇬 Singapore (MAS 2024/2025 AI MRM + FEAT)

---

## Repository Structure

```
├── Task1_Report_DBS_Selection_and_RegLandscape.pdf   # Task 1: entity selection, domain, regulatory landscape
├── Task2_Report_Values_Audit.pdf                     # Task 2: values audit — mission, perspective, risk vs compliance, cost of failure
├── Task3_Report_JurisdictionAware_Tool.pdf           # Task 3: full tool design report including architecture, metrics, and failure analysis
├── Task3_Notebook_RegTech_v4_fixed.ipynb             # Task 3: Python notebook — full quantitative implementation (bug-fixed v4)
├── Task3_Summary_OnePager.pdf                        # Task 3: one-page plain-language summary for senior management
├── Task3_Slides_Presentation.pdf                     # Task 3: senior management pitch deck (PDF)
├── README.md                                         # This file
```

---

## Task 1 — Selection and Research

**Regulated entity:** DBS Bank Limited (SGX: D05), the largest bank in Southeast Asia by total assets (~SGD 754 billion, FY2024). DBS satisfies the cross-jurisdictional requirement through its US presence: a Los Angeles representative office, broker-dealer operations via DBS Securities (USA) LLC (FINRA-regulated), and resolution-plan filings with the Federal Reserve and FDIC. DBS is also a Veritas Consortium participant and a direct adopter of the MAS FEAT Principles.

**Domain:** AI Model Risk Management applied to retail credit-scoring models — high-stakes, under explicit regulatory scrutiny in both jurisdictions, and well-suited to quantitative demonstration using standard credit risk metrics.

**Key regulatory divergences identified:**

| Dimension | 🇺🇸 US — OCC 2026-13 | 🇸🇬 SG — MAS 2024/2025 |
|---|---|---|
| GenAI coverage | Explicitly **excluded** | **Included** — all AI |
| Fairness standard | ECOA/Reg B — enumerated classes, EEOC 4/5ths rule (DI ≥ 0.80) | MAS FEAT — outcome-based, no fixed DI threshold |
| Local XAI | Not mandated | **Required** for adverse credit decisions (MAS 2024 p.14) |
| PSI retrain threshold | 0.25 | 0.20 (more conservative) |
| Independent validation | De-emphasised | Required for high-risk AI |
| Min Gini | 0.35 | 0.30 |
| IV/WoE documentation | Implied (conceptual soundness) | Explicit per-feature justification required |

---

## Task 2 — Values Audit

Answers four mandatory questions before any design:

1. **Company mission and profile:** ClearPath RegTech Pte. Ltd. — a Singapore-incorporated B2B software provider at early-growth stage (~25–40 staff), targeting Tier 1/2 financial institutions in Singapore and the US. Core competence: regulatory translation — encoding regulatory guidance into versioned, operational configuration objects.

2. **Whose perspective does the tool serve?** Primarily the second-line risk and compliance function (model risk management teams, CCOs), but designed to protect consumer interests — local SHAP explanations and fairness monitoring go beyond what many paying clients would require. Tensions between bank efficiency, regulatory robustness, and consumer fairness are explicitly acknowledged.

3. **Genuine risk measurement vs documentation compliance:** PSI monitoring (not mandated by name in either regulation) and IV/WoE analysis (auditable numeric feature justification) are included because they improve substantive risk detection. A uniform automated sign-off workflow was excluded precisely because it serves documentation over substance.

4. **Who bears the cost if the tool gets it wrong?** False negatives harm consumers. False positives harm business lines. Fairness mismeasurement harms protected groups. Jurisdictional misconfiguration — applying US rules where SG rules should apply — produces confidently wrong compliance assertions and harms consumers, compliance officers, and senior management simultaneously.

---

## Task 3 — Tool Design: Jurisdiction-Aware Credit MRM Platform

### What the Platform Does

The platform ingests model outputs (PD scores, feature vectors) and:
- Computes credit model performance metrics: ROC-AUC, Gini, KS (exact, `roc_curve`-based), Brier Score, PR-AUC, ECE (Expected Calibration Error)
- Conducts **IV/WoE feature analysis** on raw features for auditable, per-feature regulatory justification
- Computes **percentile-bin PSI** over rolling time windows for distribution drift detection
- Evaluates **fairness metrics** (DI, DPD, EOD) for Sex and Age, under both US 4/5ths rule and MAS FEAT outcome monitoring
- Generates **SHAP** (global + local per-applicant) and **LIME** (model-agnostic independent check) explanations
- Applies **versioned JurisdictionConfig objects** (US and SG) to classify each check as PASS / WARN / ESCALATE
- Produces **jurisdiction-specific adverse action notice drafts** (CFPB Reg B format vs MAS FEAT/PDPA format)
- Outputs a structured **model governance card** with explicit failure modes and limitations

### What the Platform Does NOT Do

By explicit design choice:
- Does **not** make credit decisions
- Does **not** incorporate macroeconomic stress scenarios
- Does **not** cover Basel III capital model validation
- Does **not** implement OSFI E-23 (Canada) or EU AI Act
- Does **not** handle real-time scoring — batch inference only
- Does **not** govern GenAI under the US regime, consistent with OCC 2026-13's explicit exclusion
- Does **not** replace human judgement — three explicit human gates are built into the workflow

### Six-Layer Architecture

| Layer | Component | Function |
|---|---|---|
| 1. Data Ingestion | Feature + Score Loader | Feature vectors, PD scores, population metadata |
| 2. Feature Analysis | IV/WoE Module | Per-feature predictive power; flags leakage (IV > 0.5) |
| 3. Model Evaluation | Metrics Module | KS, Gini, AUC, Brier, PR-AUC, ECE, PSI, fairness metrics |
| 4. Explainability | SHAP + LIME Module | Global OCC / local per-applicant MAS 2024 + LIME FEAT T2 |
| 5. Jurisdiction Rule Engine | JurisdictionConfig Registry | Versioned, hot-swappable regulatory parameters per jurisdiction |
| 6. Reporting & Workflow | Report + Notice Generator | Jurisdiction-tagged reports, adverse action notices, governance card |

### Quantitative Implementation (v4, all bugs corrected)

**Dataset:** German Credit Dataset (UCI/Kaggle, N=1,000). Synthetic labels constructed with a transparent, feature-weighted scoring function calibrated to a 70/30 good/bad split, grounded in Basel II expected-loss theory. Fixed seed (42) for full reproducibility.

**Four candidate models** compared via stratified 5-fold CV: Logistic Regression, Random Forest, Gradient Boosting, LightGBM.  
**Champion (v4):** Logistic Regression (CV-AUC = 0.9460) — expected result given the linear data-generating process; documented in the governance card as a prototype artifact, not a defect.

**Six bugs identified and corrected from v3 → v4:**
1. Double-encoding of Saving/Checking Account features (feature leakage)
2. Inaccurate KS via histogram-CDF approximation → replaced with exact `roc_curve`-based KS
3. Inflated PSI via fixed [0,1] bins → replaced with percentile-based bins (industry standard)
4. IV/WoE computed on derived ordinal encodings → moved to raw features
5. Incorrect 4/5ths floor (min rate × 0.80 → reference group rate × 0.80, per ECOA)
6. ECE listed in report but not implemented → now computed for all four models

### Compliance Dashboard (5-Panel Executive View)

`Task3_Notebook_RegTech_v4_fixed.ipynb` generates a single executive dashboard with:
- **Panel A:** Performance vs Thresholds (ROC-AUC, Gini, KS, 1−Brier vs US/SG minima)
- **Panel B:** Fairness by Sex (Approval Rate + TPR with correct US 4/5ths floor)
- **Panel C:** PSI Drift Monitor (Q1–Q6 with SG 0.20 and US 0.25 retrain thresholds)
- **Panel D:** Jurisdiction Decision Matrix (PASS/WARN/FAIL side-by-side, US vs SG)
- **Panel E:** Divergence Heatmap (magnitude of regulatory gap across six dimensions)

---

## Data Note

The prototype uses the **German Credit Dataset (UCI/Kaggle)** as an open, reproducible benchmark. DBS realism is injected via:
- (a) Bad-rate calibration consistent with DBS retail NPL profile (~30% subprime training distribution)
- (b) Regulatory mapping to MAS/PDPA/FEAT and OCC 2026-13 requirements
- (c) Veritas 2.0 FEAT methodology used by Singapore banks including DBS

In production, this platform would ingest Singapore Credit Bureau (CBS) scores, DBS internal model outputs, and MAS macro stress scenario data.

---

## Key References

- OCC Bulletin 2026-13 (17 Apr 2026): https://www.occ.treas.gov/news-issuances/bulletins/2026/bulletin-2026-13.html
- MAS AI MRM Information Paper (Dec 2024): https://www.mas.gov.sg/publications/monographs-or-information-paper/2024/artificial-intelligence-model-risk-management
- MAS AI Guidelines Consultation Paper (Nov 2025): https://www.mas.gov.sg/news/media-releases/2025/mas-guidelines-for-artificial-intelligence-risk-management
- MAS FEAT Principles (2018): https://www.mas.gov.sg/news/media-releases/2018/mas-and-abs-launch-principles-for-fair-responsible-and-transparent-use-of-ai
- Veritas Toolkit v2.0: https://github.com/veritas-toolkit
- FDIC DBS Resolution Plan: https://www.fdic.gov/system/files/2024-07/dbs-165-1312.pdf
- arXiv: Information-Theoretic Framework for WoE/IV (2025): https://arxiv.org/html/2408.03497v3
- PMC: LightGBM + SHAP for Credit Scoring Transparency (2024): https://pmc.ncbi.nlm.nih.gov/articles/PMC11318906

---

## Dependencies

```
python >= 3.10
scikit-learn
lightgbm
shap
lime
fairlearn
pandas
numpy
matplotlib
seaborn
```

Install all dependencies:
```bash
pip install scikit-learn lightgbm shap lime fairlearn pandas numpy matplotlib seaborn
```

---

## How to Run

1. Download `german_credit_data.csv` from [Kaggle](https://www.kaggle.com/datasets/uciml/german-credit) and place it in the working directory.
2. Open `Task3_Notebook_RegTech_v4_fixed.ipynb` in Jupyter or Google Colab.
3. Run all cells sequentially. All outputs (metrics, figures, dashboard, governance card, adverse action notices) are generated automatically.
4. The compliance dashboard is saved as `compliance_dashboard_v4.png`.

---

*MH6822 Regulatory Technology — NTU, May 2026*
