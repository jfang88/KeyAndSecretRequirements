# Secrets, Keys, and Cryptography Policy v1.1

**Organization Type:** Critical Financial Services Provider  
**Document Type:** Enterprise Policy  
**Version:** 1.1  
**Classification:** Internal Restricted  
**Review Cycle:** Annual or material change  
**Approval:** Security Steering Committee / CISO  
**Effective:** 2026-03-23  
**Companion:** Secrets, Keys, and Cryptography Technical Standards v1.4

---

## Change History

| Version | Date | Author | Status | Changes |
|---------|------|--------|--------|---------|
| 0.1 | 2026-03-23 | Security Architecture | Draft | Initial policy aligned to Cap. 653, HKMA, NIST CSF 2.0 Govern function |
| 1.0 | 2026-04-01 | Security Architecture | Approved | Final version with board review comments incorporated |

---

## Table of Contents

1. [Purpose and Scope](#1-purpose-and-scope)
2. [Policy Authority](#2-policy-authority)
3. [Universes of Coverage](#3-universes-of-coverage)
4. [Policy Principles](#4-policy-principles)
5. [Control Objectives](#5-control-objectives)
6. [Roles and Responsibilities](#6-roles-and-responsibilities)
7. [Exceptions and Risk Acceptance](#7-exceptions-and-risk-acceptance)
8. [Assurance and Evidence](#8-assurance-and-evidence)
9. [Compliance Mapping](#9-compliance-mapping)
10. [References](#10-references)

---

## 1. Purpose and Scope

This policy establishes the enterprise governance requirements for secrets management, cryptographic key management, and cryptography usage to satisfy Cap. 653 critical infrastructure obligations, HKMA/SFC financial services requirements, and NIST CSF 2.0 enterprise cybersecurity framework.

**Scope:** All production and non-production environments including on-premises, VMware, Huawei private cloud, Kubernetes clusters, and public cloud services.

**Out of Scope:** End-user password composition (governed by IAM policy); proprietary HSM firmware internals.

Technical standards and platform implementations are defined in companion documents.

---

## 2. Policy Authority

This policy derives authority from:

1. **Hong Kong Cap. 653** (Protection of Critical Infrastructures Ordinance) and associated Code of Practice — statutory requirements for Category A financial services infrastructure operators.
2. **SFC cybersecurity requirements and guidance** — primary regulatory expectations for licensed activities, including applicable SFC guidelines, circulars, and conduct requirements.
3. **HKMA guidance** — informative benchmark material only unless separately adopted, contractually required, or made applicable by another obligation.
4. **NIST Cybersecurity Framework 2.0** — enterprise cybersecurity governance model.
5. **Enterprise Risk Appetite** — approved by Board Risk Committee.

**Precedence:** Cap. 653 obligations and applicable SFC requirements take precedence. HKMA references are informative benchmark requirements unless separately adopted by the enterprise.

---

## 3. Universes of Coverage

| Universe | Definition | Examples | Policy Driver |
|----------|------------|----------|---------------|
| **Secrets** | Sensitive values granting access/auth/authz | API keys, service passwords, tokens, client secrets | Cap. 653 access control; HKMA client protection |
| **Cryptographic Keys** | Parameters for crypto algorithms | DEKs/KEKs, signing keys, HMAC keys | Cap. 653 cryptography; NIST 800-57 lifecycle |
| **Cryptography** | Algorithms/modules/protocols | AES-256, ECDSA, TLS 1.3, HSM/KMS | Cap. 653 cryptography; NIST approved lists |

**Overlap:** Secrets often encrypted by crypto keys; certificates combine public crypto key + private key secret.

---

## 4. Policy Principles

1. **Cap. 653 Compliance First** — All secrets/keys/crypto controls must satisfy statutory management plan, risk assessment, audit, access control, cryptography, logging, and incident response obligations.
2. **Integrated Lifecycle Governance** — Secrets, keys, and cryptography follow unified lifecycle principles from design through destruction.
3. **Least Privilege and Separation** — No single individual controls secret/key lifecycle end-to-end; privileged access separated.
4. **Crypto Strength** — Only enterprise-approved algorithms, key sizes, and FIPS-equivalent modules for production.
5. **Auditability** — All access, rotation, policy changes logged with 6-month minimum retention per Cap. 653.
6. **Crypto-Agility** — Designs support algorithm parameter replacement without re-architecture.
7. **Regional Compliance** — PRC commercial cryptography (SM-series) where business/regulatory requirements apply.
8. **Evidence-Based Assurance** — Design, implementation, and operating evidence maintained for regulatory audit.

---

## 5. Control Objectives (NIST CSF 2.0)

### 5.1 Govern (GV)
**Objective:** Establish organizational accountability, policy direction, and risk management for secrets/keys/crypto.

- Board/senior mgmt endorsement of security management plan (Cap. 653).
- Named owner for secrets/keys/crypto standards.
- Approved algorithm/profile baseline published.
- Exception process with risk acceptance and time bounds.

### 5.2 Identify (ID)  
**Objective:** Know what secrets/keys/crypto assets exist, who owns them, and what depends on them.

- Complete inventory of secrets, keys, crypto usage.
- Dependency mapping (consumers/producers).
- Data classification driving protection level.

### 5.3 Protect (PR)
**Objective:** Protect confidentiality, integrity, and lifecycle of secrets/keys using approved crypto.

- Approved storage (encrypted/vaulted).
- Least privilege access to secrets/keys/services.
- Automated rotation/revocation where feasible.
- Crypto modules meet enterprise baseline.

### 5.4 Detect (DE)
**Objective:** Detect misuse, anomalies, and policy violations.

- Comprehensive logging of access/usage/policy changes.
- Anomaly detection (volume/geography/abuse).
- Monitoring of rotation failures/cert expiries.

### 5.5 Respond (RS)
**Objective:** Contain and eradicate compromise impact.

- Playbooks for secret/key compromise.
- Blast radius assessment procedures.
- Regulatory notification triggers.

### 5.6 Recover (RC)
**Objective:** Restore trusted operations post-incident/migration.

- Rekey/secret re-issuance procedures.
- Trust recovery (PKI/trust anchor rollover).
- Profile migration planning.

---

## 6. Roles and Responsibilities

| Role | Responsibilities |
|------|------------------|
| **Board Risk Committee** | Policy endorsement; risk appetite for exceptions |
| **CISO** | Policy ownership; exception escalation |
| **Security Architecture** | Standards maintenance; profile approval; design review |
| **Cryptography Team** | PKI/KMS/HSM operations; algorithm baseline |
| **Cloud Security** | Cloud platform mappings; shared responsibility |
| **Platform Teams** | Secrets platform operation (CSMS/Vault) |
| **IAM Team** | Access governance for secrets services |
| **SOC** | Monitoring/alerting/incident response |
| **Application Owners** | Secret consumer compliance; dependency documentation |
| **Internal Audit** | Assurance testing; evidence review |

**Separation Rule:** No individual performs request + approval + execution + evidence closure for production secrets/keys.

---

## 7. Exceptions and Risk Acceptance

All deviations require formal risk acceptance.

### 7.1 Minimum Fields

| Field | Required |
|-------|----------|
| Exception ID | Unique |
| Impacted Controls | Standards Doc #2 IDs |
| Business Justification | Why non-compliant |
| Risk Assessment | Confidentiality/integrity/availability/regulatory impact |
| Compensating Controls | Interim measures |
| Approver | Security Architecture + Risk Owner |
| Expiration | Max 12 months |

### 7.2 Review Cadence

- High-risk: Quarterly
- Medium-risk: Semi-annual
- Low-risk: Annual

---

## 8. Assurance and Evidence

Cap. 653 requires documented evidence of design, implementation, and operating effectiveness.

### 8.1 Evidence Categories

| Category | Examples |
|----------|----------|
| **Design** | Architecture diagrams; secret/key flow; crypto profile selection |
| **Inventory** | Secret/key register exports |
| **Implementation** | Platform config; KMS policy; Vault ACLs |
| **Operations** | Rotation logs; access logs; monitoring dashboards |
| **Assurance** | Audit reports; pen test results; exception register |

### 8.2 Testing Requirements

- Annual control testing
- Incident/posture review
- Platform certification evidence

---

## 9. Compliance Mapping

### 9.1 Cap. 653 / SFC

| Cap. 653 Category | Policy Coverage |
|-------------------|-----------------|
| Management Plan | Policy endorsement; standards reference |
| Risk Assessment | Exception process |
| Access Control | Least privilege; logging |
| Cryptography | Approved profiles |
| Logging | 6-month retention |
| Incident Response | Playbooks/notification |

### 9.2 HKMA benchmark status

HKMA materials are retained as informative benchmark references to strengthen control design, evidence expectations, and sector alignment, but they are not treated as primary policy obligations unless separately adopted.

### 9.3 NIST CSF 2.0 Functions

Mapped in Section 5.

---

## 10. References

1. Hong Kong Cap. 653 Protection of Critical Infrastructures Ordinance
2. SFC cybersecurity requirements, circulars, guidelines, and Code of Conduct materials applicable to licensed activities
3. HKMA TM-G-1 and related HKMA cybersecurity guidance (informative benchmark only)
4. NIST Cybersecurity Framework 2.0
5. Secrets, Keys, and Cryptography Technical Standards v1.4
6. Secrets Management Implementation Guide v1.1
7. Cryptography Management Implementation Guide v1.0

---

*This policy must be reviewed annually or upon material regulatory/technical change.*

## References

### Internal Documents

| Doc Ref | Document | Version |
|---|---|---|
| DOC-01 | Secrets, Keys, and Cryptography Policy | v1.1 |
| DOC-02 | Secrets, Keys, and Cryptography Technical Standards | v1.4 |
| DOC-03 | Secrets Management Implementation Guide | v1.1 |
| DOC-04 | Cryptography Management Implementation Guide | v1.0 |

### External Sources

| Source ID | Reference | URL |
|---|---|---|
| EXT-01 | SFC Cybersecurity topic page | [https://www.sfc.hk/en/Regulatory-functions/Intermediaries/Supervision/Search-regulations-by-topic/Cybersecurity](https://www.sfc.hk/en/Regulatory-functions/Intermediaries/Supervision/Search-regulations-by-topic/Cybersecurity) |
| EXT-02 | NIST Cybersecurity Framework 2.0 | [https://www.nist.gov/cyberframework](https://www.nist.gov/cyberframework) |
| EXT-03 | NIST SP 800-57 Part 1 Rev. 5 | [https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final) |
| EXT-04 | CSA Cloud Controls Matrix (CCM) | [https://cloudsecurityalliance.org/research/cloud-controls-matrix](https://cloudsecurityalliance.org/research/cloud-controls-matrix) |
| EXT-05 | OWASP Secrets Management Cheat Sheet | [https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) |

