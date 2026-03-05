# NovaCred Data Retention & Disposal Policy

**Version:** 1.1  
**Status:** Mandatory Draft  
**Prepared by:** Governance Officer  

---

## 1. Purpose

This policy ensures NovaCred complies with the **GDPR Principle of Storage Limitation (Art. 5(1)(e))**, stating that personal data shall be kept for no longer than is necessary for the purposes for which the personal data are processed.

---

## 2. Retention Schedule

| Data Category | Retention Period | Legal & Governance Justification |
|---------------|-----------------|----------------------------------|
| Approved Loan Data | 10 Years post-settlement | Financial audit requirements and tax records |
| Rejected Applications | 5 Years | Necessary to defend against discrimination claims (Audit Trail) |
| Behavioral Data (Art. 9) | Immediate Purge | Categories like "Alcohol" identified in spending_behavior must be removed |
| Metadata (IP Addresses) | 30 Days | Temporary security logs only |

---

## 3. Data Protection Mechanisms

As demonstrated in the `03-privacy-demo.ipynb` notebook, the following technical safeguards are mandatory:

- **Pseudonymization (Art. 25):** Unique identifiers like SSNs must be hashed using SHA-256 at the ingestion layer.
- **Minimization (Art. 5):** Non-essential fields (e.g., IP Address) must be dropped before the Analytics Zone.

---

## 4. Secure Disposal (Art. 17 GDPR)

NovaCred must support the **"Right to be Forgotten"** through:

1. **Hard Deletion:** Permanent removal from all databases within 30 days of retention expiration.
2. **Anonymization:** Data kept for long-term research must have all identifiers irreversibly destroyed.

> Note: This policy must be reviewed every six months by the Credit Review Committee.