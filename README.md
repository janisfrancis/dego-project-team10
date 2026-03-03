# DEGO Project - Team 10

## Team Members
- [janisfrancis] | 64674
- [justin-rkr] | 70770
- [Fra73166] | 73166
- [cxseiro] | 56517

## Project Description
Credit scoring bias analysis for DEGO course.

## Structure
- ‘data /‘ - Dataset files
- ‘notebooks /‘ - Jupyter analysis notebooks
- ‘src /‘ - Python source code
- ‘reports /‘ - Final deliverables

Governance & Compliance Assessment (NovaCred)

As the Governance Task Force, we have audited the credit application pipeline against GDPR and the EU AI Act frameworks.

1. Privacy & Data Protection (GDPR)

Our audit identified high-risk personal data (PII) including SSNs, emails, and IP addresses.

Data Minimization (Art. 5):We found that ip_address is currently collected but not utilized in the risk scoring. We recommend removing this field to reduce exposure. 

Privacy by Design (Art. 25): We demonstrated SHA-256 hashing for SSNs in 03-privacy-demo.ipynb to ensure pseudonymization at the ingestion layer.

Automated Decision-Making (Art. 22): The current system lacks a "Human-in-the-loop" mechanism. We propose a mandatory manual review for all "High Risk" rejections.

2. Algorithmic Fairness (AI Act)

The system is classified as High-Risk AI under Annex III of the EU AI Act.

Bias Findings: Our analysis (see 02-bias-analysis.ipynb) detected a Disparate Impact Ratio (DIR) below 0.8 for gender and age groups.

Proxy Discrimination: ZIP codes were found to correlate strongly with protected attributes. We recommend removing geographic indicators from the training features.

3. Actionable Recommendations for CTO

Implement an Audit Trail: Every decision made by the algorithm must be logged with the specific feature weights used (Transparency requirement).

Bias Monitoring Gate: Integrate a pre-deployment check that blocks any model with a DIR < 0.8.

Data Retention Policy: Establish a 5-year deletion cycle for rejected applications to comply with storage limitation principles.

Human Oversight: Appoint a Compliance Officer to review the "Algorithm Rejection" cases weekly.
