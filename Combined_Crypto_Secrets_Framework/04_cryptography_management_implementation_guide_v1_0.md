# Cryptography Management Implementation Guide v1.0

**Organization Type:** Critical Financial Services Provider  
**Document Type:** Implementation Guide  
**Version:** 1.0  
**Classification:** Internal Restricted  
**Review Cycle:** Annual  
**Companion Standards:** Secrets, Keys, and Cryptography Technical Standards v1.4

**Status:** Approved Implementation Guide

---

## Table of Contents

1. [Purpose and Scope](#1-purpose-and-scope)
2. [Implementation Model](#2-implementation-model)
3. [Platform Mapping](#3-platform-mapping)
4. [Huawei Cloud KMS](#4-huawei-cloud-kms)
5. [AWS KMS](#5-aws-kms)
6. [VMware vHSM](#6-vmware-vhsm)
7. [CloudHSM / Dedicated HSM](#7-cloudhsm--dedicated-hsm)
8. [Shared Responsibility Matrices](#8-shared-responsibility-matrices)
9. [Key Custody Models](#9-key-custody-models)
10. [Migration and Onboarding](#10-migration-and-onboarding)
11. [Troubleshooting and Operations](#11-troubleshooting-and-operations)

**Appendices:**
A. [API Examples and Configuration](#appendix-a-api-examples-and-configuration)
B. [Control Mapping to Standards Doc #2](#appendix-b-control-mapping-to-standards-doc-2)

---

## 1. Purpose and Scope

This guide provides platform-specific implementation patterns for **cryptographic key management only** (generation, protection, usage, rotation of DEKs/KEKs/signing keys).

**Secrets management** (API keys/service credentials) is covered in the companion **Secrets Management Implementation Guide**.

**Reference:** Secrets, Keys, and Cryptography Technical Standards v1.4 (Standards Doc #2) for lifecycle requirements, approved profiles, and control IDs.

### 1.1 Objectives

- Provide tested key management patterns for each platform.
- Map platform capabilities to Standards Doc #2 controls (PR-KMS, PR-ALG).
- Document BYOK/CMK/HYOK custody models.
- Enable consistent cryptographic key handling across hybrid infrastructure.

### 1.2 Platforms Covered

| Platform | Key Service | Custody Model | Status |
|----------|-------------|---------------|--------|
| Huawei Cloud | KMS | CMK/BYOK | Primary private cloud |
| AWS | KMS | CMK/BYOK | Public cloud primary |
| VMware | vHSM | vHSM cluster | On-prem/VMware |
| Multi-cloud | CloudHSM | Dedicated HSM | High-value keys |

---

## 2. Implementation Model

### 2.1 Core Principles

All implementations must satisfy Standards Doc #2:

- **Key Generation**: CSPRNG/HSM-backed
- **Custody**: Enterprise-controlled (CMK/BYOK/HYOK)
- **Usage**: API-only (no export for high-value)
- **Rotation**: Cryptoperiod-based
- **Audit**: Key policy/usage events to SIEM

### 2.2 Custody Decision Framework

| Sensitivity | Recommended Custody |
|-------------|---------------------|
| Tier 1 (roots/signing) | Dedicated HSM/vHSM |
| Tier 2 (KEKs) | CloudHSM/KMS CMK |
| Tier 3 (DEKs) | KMS BYOK |
| Tier 4 | Provider-managed (exception) |

---

## 3. Platform Mapping

| Capability | Huawei KMS | AWS KMS | vHSM | CloudHSM |
|------------|------------|---------|------|----------|
| Key Types | Symmetric | Symmetric/Asym | PKCS#11 | PKCS#11 |
| BYOK Import | Yes | Yes | Yes | Yes |
| Rotation | Manual/API | Automatic | Manual | Manual |
| Audit | Cloud logs | CloudTrail | vHSM logs | HSM logs |
| FIPS | Yes | FIPS mode | FIPS validated | FIPS validated |

---

## 4. Huawei Cloud KMS

### 4.1 Overview

Huawei KMS provides customer-managed keys (CMK) for data protection and CSMS secret encryption.

**Maps to Standards Doc #2:**
- Custody: CMK → PR-KMS-01
- Rotation: API → PR-KMS-03
- Audit: Cloud logs → DE-CRY-01

### 4.2 Core Patterns

**Pattern 1: CMK Creation**
```bash
aliyun kms CreateKey \\
  --KeyUsage ENCRYPT_DECRYPT \\
  --Description "Payments DEK" \\
  --KeySpec SYMMETRIC_DEFAULT
```

**Pattern 2: BYOK Import**
```
aliyun kms ImportKeyMaterial \\
  --KeyId {cmk-id} \\
  --EncryptedKeyMaterial {wrapped-material} \\
  --WrappingAlgorithm RSAES_OAEP_SHA_256 \\
  --WrappingKeyHashAlgorithm SHA384
```

### 4.3 Integration Points

- **CSMS Secrets**: Secrets encrypted by KMS CMK
- **CCE/Object Storage**: Runtime encryption
- **Logs**: Cloud Log Service → SIEM

---

## 5. AWS KMS

### 5.1 Overview

AWS KMS for symmetric/asymmetric keys with automatic rotation and CloudTrail audit.

### 5.2 Core Patterns

**Pattern 1: CMK w/ Rotation**
```json
{{
  "KeyUsage": "ENCRYPT_DECRYPT",
  "KeySpec": "SYMMETRIC_DEFAULT",
  "EnableKeyRotation": true
}}
```

**Pattern 2: BYOK**
```
aws kms import-key-material \\
  --key-id alias/my-key \\
  --encrypted-key-material fileb://wrapped.bin \\
  --expiration-model KEY_MATERIAL_EXPIRES \\
  --expiration-time 2026-12-31T00:00:00Z
```

**Multi-Region Replication:**
```
aws kms replicate-key \\
  --key-id arn:...:primary-region \\
  --replica-region us-west-2
```

---

## 6. VMware vHSM

### 6.1 Overview

VMware-validated Hardware Security Module for vSphere/NSX with PKCS#11 interface.

### 6.2 Core Patterns

**vHSM Cluster Setup:**
```
HSM partition creation command:
pkcs11-tool --module /usr/lib/softhsm/libsofthsm2.so \\
  --login --pin 1234 --init-token --label "vHSM-Prod"
```

**PKCS#11 Integration:**
```
Application PKCS#11 configuration example:
library=/opt/vhsm/libvhsm_pkcs11.so
slot=0
userpin=secret
```

**Key Generation:**
```
pkcs11-tool --module libvhsm_pkcs11.so \\
  --login --keypairgen --key-type EC:secp384r1 \\
  --label "TLS-Signing-Key"
```

---

## 7. CloudHSM / Dedicated HSM

### 7.1 Overview

Dedicated HSM partitions (AWS CloudHSM, Azure Dedicated HSM) for Tier 1 keys.

### 7.2 Core Patterns

**AWS CloudHSM Cluster:**
```
aws cloudhsmv2 create-cluster \\
  --subnet-ids subnet-123 subnet-456 \\
  --hsm-type hsm1.medium
```

**PKCS#11 Key Gen:**
```
pkcs11-tool --module /opt/cloudhsm/lib/libcloudhsm_pkcs11.so \\
  --login --so-pin {sopass} \\
  --keypairgen --mechanism ECDSA-KEY-PAIR-GEN \\
  --key-type EC:prime256v1
```

---

## 8. Shared Responsibility Matrices

### 8.1 Huawei KMS

| Control | Enterprise | Huawei |
|---------|------------|---------|
| Key Policy | CMK config | KMS service |
| Key Material | BYOK custody | Generate/rotate |
| Audit Logs | SIEM integration | Raw Cloud Logs |

### 8.2 AWS KMS

| Control | Enterprise | AWS |
|---------|------------|---------|
| Rotation Policy | Enable/disable | Automatic execution |
| Grants/Policies | IAM/KMS policy | Enforcement |
| CloudTrail | Subscription | Log delivery |

---

## 9. Key Custody Models

| Model | Use Case | Platforms | Standards Ref |
|-------|----------|-----------|---------------|
| **CMK** | Tier 2/3 DEKs | All cloud KMS | PR-KMS-02 |
| **BYOK** | Import existing | KMS w/ wrapping | PR-KMS-04 |
| **HYOK** | Export prohibited | vHSM/CloudHSM | PR-KMS-01 |
| **Provider-Managed** | Tier 4 only | Exception | GV-CRY-04 |

---

## 10. Migration and Onboarding

1. **Inventory existing keys** (type/location/custody)
2. **Map to custody model** above
3. **Test key operations** (encrypt/decrypt/rotate)
4. **Cutover** w/ dual-key validation
5. **Update dependents** (CSMS policies, app config)

---

## 11. Troubleshooting and Operations

**Common Issues:**
| Issue | Platform | Resolution |
|-------|----------|------------|
| KMS quota exceeded | Huawei/AWS | Request increase |
| vHSM cluster fail | VMware | Backup/restore partition |
| BYOK unwrap fail | All | Check wrapping algo/key size |

**Monitoring:**
- Key policy changes
- Rotation failures
- Grant/usage anomalies

---

## Appendix A. API Examples and Configuration

**Huawei KMS Rotate Key Material:**
```bash
aliyun kms ScheduleKeyDeletion \\
  --KeyId {cmk-id} \\
  --PendingWindowInDays 7
```

**AWS KMS Grant Example:**
```json
{{
  "GranteePrincipal": "arn:aws:iam::account:role/app-role",
  "Operations": ["Decrypt", "GenerateDataKey"]
}}
```

## Appendix B. Control Mapping to Standards Doc #2

| Platform Pattern | Standards Doc #2 Control |
|------------------|--------------------------|
| Huawei KMS CMK | PR-KMS-01 (Custody) |
| AWS Auto Rotation | PR-KMS-03 (Rotation) |
| vHSM PKCS#11 | PR-KMS-05 (Modules) |
| CloudHSM Partition | PR-KMS-01 (Tier 1) |

---

*References Standards Doc #2 for full technical requirements.*

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
| EXT-01 | AWS Key Management Service Documentation | [https://docs.aws.amazon.com/kms/](https://docs.aws.amazon.com/kms/) |
| EXT-02 | AWS CloudHSM Documentation | [https://docs.aws.amazon.com/cloudhsm/](https://docs.aws.amazon.com/cloudhsm/) |
| EXT-03 | Huawei Cloud KMS for Encryption | [https://support.huaweicloud.com/intl/en-us/usermanual-dew/dew_01_0094.html](https://support.huaweicloud.com/intl/en-us/usermanual-dew/dew_01_0094.html) |
| EXT-04 | NIST SP 800-57 Part 1 Rev. 5 | [https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final) |
| EXT-05 | CSA Cloud Controls Matrix (CCM) | [https://cloudsecurityalliance.org/research/cloud-controls-matrix](https://cloudsecurityalliance.org/research/cloud-controls-matrix) |

