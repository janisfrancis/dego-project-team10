# Credit Application Governance Analysis
**DEGO 2606 — Data Ecosystems and Governance in Organizations**
**Nova SBE | MSc Business Analytics | Team 10**

---

## Team Members

| GitHub | Student ID | Role |
|---|---|---|
| [janisfrancis](https://github.com/janisfrancis) | 64674 | Data Engineer |
| [justin-rkr](https://github.com/justin-rkr) | 70770 | Data Scientist |
| [Fra73166](https://github.com/Fra73166) | 73166 | Governance Officer |
| [cxseiro](https://github.com/cxseiro) | 56517 | Product Lead |

---

## Project Overview

NovaCred is a fintech startup using machine learning to make automated credit decisions. Following a regulatory inquiry about potential discrimination in their lending practices, our team was engaged as a **Data Governance Task Force** to:

1. Audit the raw credit application dataset for data quality issues
2. Detect bias patterns in historical lending decisions
3. Propose governance controls to ensure GDPR and EU AI Act compliance

The analysis is structured across three notebooks, each corresponding to a team role. The full pipeline runs against a local MongoDB instance (`novacred` database).

---

## Repository Structure

```
dego-project-team10/
├── README.md
├── data/
│   └── raw_credit_applications.json
├── notebooks/
│   ├── 01-data-quality.ipynb
│   ├── 02-bias-analysis.ipynb
│   └── 03-privacy-demo.ipynb
├── src/
│   └── fairness_utils.py
└── presentation/
```

---

## How to Run

1. Start a local MongoDB instance on `mongodb://localhost:27017/`
2. Place `raw_credit_applications.json` in the `data/` folder
3. Run the notebooks **in order**: `01` → `02` → `03`
   - `01` ingests the raw data and writes a clean baseline to `novacred.clean_credit_applications`
   - `02` and `03` read from the clean collection

```bash
pip install pymongo pandas numpy scipy matplotlib seaborn
```

---

## Executive Summary

We audited 491 raw credit applications from NovaCred's lending system. The analysis uncovered significant data quality failures, statistically confirmed gender and age discrimination, and identified systemic governance gaps that constitute breaches of both the GDPR and the EU AI Act.

**The credit scoring system cannot legally operate in its current state.** It is classified as a High-Risk AI System under EU AI Act Annex III, point 5(b), and currently lacks the mandatory transparency, human oversight, and bias mitigation mechanisms required by law.

---

## 1. Data Quality Findings

**Notebook:** `notebooks/01-data-quality.ipynb`
**Raw dataset:** 491 records | **Clean baseline:** 485 records

We audited the dataset across four dimensions — Uniqueness, Consistency, Completeness, and Validity.

### Summary of Issues

| Dimension | Issue | Records Affected | % of Dataset |
|---|---|---|---|
| Uniqueness | Duplicate `_id` (RESUBMISSION flag) | 1 | 0.2% |
| Uniqueness | SSN identity collisions | 6 | 1.2% |
| Uniqueness | Missing SSN values | 5 | 1.0% |
| Consistency | Inconsistent DOB format (DD/MM/YYYY) | 100 | 20.4% |
| Consistency | Inconsistent DOB format (YYYY/MM/DD) | 55 | 11.2% |
| Consistency | Empty `date_of_birth` (app_350) | 1 | 0.2% |
| Consistency | Future-dated `processing_timestamp` | 2 | 0.4% |
| Completeness | Missing `processing_timestamp` | 430 | 87.6% |
| Completeness | Missing `loan_purpose` (undocumented field) | 442 | 90.0% |
| Completeness | Governance fields absent (all records) | 491 | 100% |
| Completeness | Empty email address | 3 | 0.6% |
| Completeness | Sensitive spending categories (Alcohol/Gambling/Adult Entertainment) | 22 | 4.5% |
| Validity | Debt-to-income ratio > 1.0 (app_402) | 1 | 0.2% |
| Validity | Negative `credit_history_months` | 2 | 0.4% |
| Validity | Negative `savings_balance` (app_290) | 1 | 0.2% |

### Key Remediation Steps

- **Uniqueness:** 11 violating records quarantined to `quarantine_uniqueness` collection
- **Consistency:** All date of birth strings normalised to ISO 8601 (`YYYY-MM-DD`); missing age for `app_350` imputed using dataset median
- **Completeness:** Future-dated timestamps quarantined; governance metadata gaps documented as systemic GDPR violations
- **Validity:** 4 records with impossible financial values removed from the active baseline

---

## 2. Bias Detection & Fairness Findings

**Notebook:** `notebooks/02-bias-analysis.ipynb`
**Clean baseline:** 485 records

We applied the **Four-Fifths Rule (80% Rule)** as the primary test for disparate impact. A Disparate Impact Ratio (DIR) below 0.80 constitutes prima facie evidence of discrimination.

### Summary of Findings

| # | Finding | Key Metric | Threshold | Status | Regulation |
|---|---|---|---|---|---|
| 1 | Gender disparate impact | DIR = 0.776 | 0.80 | **FAIL** | Four-Fifths Rule, AI Act Art. 10 |
| 2 | Age bias (Under 30) | DIR = 0.702 | 0.80 | **FAIL*** | AI Act Art. 10 |
| 3 | ZIP code as gender proxy | Chi-sq p < 0.001; conditional DIR = 0.693 | N/A | **FAIL** | AI Act Art. 10(2)(f), 10(3) |
| 4 | Intersectional bias | Young women DIR = 0.712; low-income women DIR = 0.599 | 0.80 | **FAIL** | AI Act Art. 10 |
| 5 | Opaque algorithm | 81% of rejections cite only `algorithm_risk_score` | N/A | **FAIL** | AI Act Art. 13, 14; GDPR Art. 22 |
| 6 | Spending categories as proxy | Chi-sq p = 0.789 (n=22) | 0.05 | Inconclusive | N/A |
| 7 | Financial parity across genders | All t-test p > 0.15 | 0.05 | **PASS** (supports findings 1–5) | — |

*Age DIR fails the threshold, but younger applicants have significantly weaker financial profiles (lower income, shorter credit history, lower savings — all p < 0.001). Some of the approval gap may reflect legitimate risk differences. See Finding 2 for full context.

### Key Results

**Finding 1 — Gender Bias:** Female applicants are approved at 51.4% vs. 66.2% for males — a 14.8 percentage point gap. The DIR of 0.776 falls below the 0.80 threshold. Critically, Finding 7 confirms that male and female applicants have statistically identical financial profiles (all p > 0.15), meaning the disparity cannot be explained by women having weaker creditworthiness.

**Finding 3 — Proxy Discrimination:** ZIP prefix 100xx (NYC area) is 88.5% male with a 64.6% approval rate; ZIP prefix 902xx (LA area) is 93.3% female with a 52.7% approval rate. The gender penalty persists even within the same ZIP prefix (conditional DIR = 0.693), confirming ZIP code as a proxy variable for gender.

**Finding 4 — Intersectional Bias:** Gender bias is most severe for young women (Under 30, DIR = 0.712) and low-income women (Q1 DIR = 0.670, Q2 DIR = 0.599). The algorithm disproportionately penalises women who are already economically vulnerable.

**Finding 5 — Algorithm Opacity:** 81% of rejections cite only `algorithm_risk_score` with no actionable explanation. Paradoxically, algorithmically rejected applicants have on average better financial profiles than those rejected for explicit financial reasons, suggesting the model relies on non-financial proxy features.

---

## 3. Privacy & Governance Findings

**Notebook:** `notebooks/03-privacy-demo.ipynb`

### PII Inventory

The dataset contains 5 direct identifier fields stored without any protection in the raw collection:

| Field | Sensitivity | GDPR Basis Required | Issue |
|---|---|---|---|
| `full_name` | High | Art. 6(1)(b) | No pseudonymisation |
| `email` | High | Art. 6(1)(b) | No pseudonymisation; 3 records empty |
| `ssn` | Critical | Art. 6(1)(b) | Stored in plain text |
| `ip_address` | Medium | Art. 6(1)(f) | No legitimate credit purpose |
| `date_of_birth` | High | Art. 6(1)(b) | No pseudonymisation |

### Critical GDPR Violations

- **Art. 5(1)(c) — Data Minimisation:** `ip_address` has no legitimate value for credit assessment and must be removed
- **Art. 9 — Special Categories:** `spending_behavior` contains lifestyle tags (Alcohol, Gambling, Adult Entertainment) that can infer health status and addictions. Processing these without explicit consent is unlawful
- **Art. 5(1)(e) — Storage Limitation:** No `retention_until` field exists across any of the 491 records — a complete absence of data lifecycle management
- **Art. 22 — Automated Decision-Making:** 81% of rejections are made with no human review and no meaningful explanation

### Technical Safeguard Demonstrated

SHA-256 pseudonymisation was applied to SSN, email, and full name fields in `03-privacy-demo.ipynb`, demonstrating Privacy by Design (GDPR Art. 25). This does not constitute full anonymisation — the data remains subject to GDPR — but significantly reduces re-identification risk in the event of a breach.

---

## 4. Governance Recommendations

### Immediate Actions

1. **Suspend algorithm-only rejections** — Automated rejections based solely on `algorithm_risk_score` are non-compliant with AI Act Art. 13 and GDPR Art. 22 until the model is audited and made explainable
2. **Remove ZIP code** from model features to eliminate the confirmed proxy discrimination channel

### Short-Term

3. **Implement a Bias-Gate** — Block model deployment if DIR for any protected attribute falls below 0.80 (CI/CD pipeline check)
4. **Pseudonymise all PII** at ingestion — Apply SHA-256 hashing to SSN, email, and full name before data enters the analytics zone
5. **Replace `algorithm_risk_score`** with specific, interpretable rejection reasons for every applicant

### Medium-Term

6. **Establish a Credit Review Committee** — Human-in-the-loop review for all algorithm-based rejections, complying with GDPR Art. 22 and AI Act Art. 14
7. **Implement data retention policy** — Rejected applications purged after 5 years (GDPR Art. 5(1)(e))
8. **Adopt intersectional fairness testing** — Evaluate DIR not only for single protected attributes but for their cross-tabulations (gender × age, gender × income)

### Ongoing

9. **Deploy a bias monitoring dashboard** tracking DIR metrics on a rolling basis in production
10. **Implement immutable audit logs** recording model version, input features, and decision timestamps (AI Act Art. 12)

---

## Regulatory Framework

| Regulation | Articles | Relevance |
|---|---|---|
| EU AI Act | Annex III, 5(b) | Credit scoring = High-Risk AI System |
| EU AI Act | Art. 10 | Training data must be free from discriminatory bias |
| EU AI Act | Art. 12 | Logging and traceability requirements |
| EU AI Act | Art. 13 | Transparency and explainability |
| EU AI Act | Art. 14 | Human oversight |
| GDPR | Art. 5 | Data minimisation, storage limitation |
| GDPR | Art. 6 & 9 | Lawful basis for processing |
| GDPR | Art. 22 | Rights related to automated decision-making |
| GDPR | Art. 25 | Privacy by Design |
| EU Employment Equality Directive | 2000/78/EC | Age as a protected characteristic |