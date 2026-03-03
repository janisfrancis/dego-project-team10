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





⚖️ 4. Data Governance & Regulatory Compliance

As the Governance Officer, I conducted a regulatory audit of the NovaCred ecosystem. Our findings indicate that the current machine learning pipeline presents significant legal risks under the GDPR and the EU AI Act.

4.1 Key Findings & Risks

AI Act Classification: The credit scoring system is a High-Risk AI System (Annex III, 5b). It currently lacks the mandatory transparency and human oversight mechanisms required by law.

Privacy Gaps (GDPR):

Art. 9 Violation: We identified the collection of sensitive behavioral data (e.g., "Alcohol" spending), which constitutes an unlawful processing of health-related inferences without explicit consent.

Art. 5 Failure: The system fails the Data Minimization principle by collecting metadata (IP addresses) that are irrelevant to credit risk assessment.

Proxy Discrimination: The use of zip_code as a feature facilitates indirect discrimination, as it correlates with protected groups identified in our Bias Analysis.

4.2 Implemented & Proposed Controls

Pseudonymization Pipeline: We demonstrated the hashing of Social Security Numbers (SSN) using SHA-256 to ensure Privacy by Design (Art. 25).

Fairness Gate: We mandate the removal of zip_code from the training set and the implementation of a Bias-Gate (ensuring DIR > 0.8) before model deployment.

Human-in-the-Loop (HITL): We propose the creation of a manual review process for any loan rejection to comply with Art. 22 GDPR (Automated individual decision-making).

Audit Trail: Every algorithmic decision must now be logged with specific feature importance scores to satisfy the Transparency (Art. 13 AI Act) requirement.

Detailed analysis and technical demonstrations are available in notebooks/03-privacy-demo.ipynb.