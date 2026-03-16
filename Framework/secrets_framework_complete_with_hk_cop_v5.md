# Secrets and Cryptographic Key Management Framework

**Organization Type:** Critical Financial Services Provider  
**Document Type:** Architecture and Standards  
**Version:** 1.0  
**Classification:** Internal Restricted  
**Review Cycle:** Annual or on material change

---

## Table of Contents

1. [Purpose and Scope](#1-purpose-and-scope)
2. [Governing Standards and Frameworks](#2-governing-standards-and-frameworks)
3. [Definitions and Secret Type Taxonomy](#3-definitions-and-secret-type-taxonomy)
4. [Lifecycle Model](#4-lifecycle-model)
5. [Secret Type Requirements](#5-secret-type-requirements)
6. [Service Account Framework](#6-service-account-framework)
7. [SSH Key Pair Framework](#7-ssh-key-pair-framework)
8. [Inventory and Metadata Schema](#8-inventory-and-metadata-schema)
9. [Encryption Requirements](#9-encryption-requirements)
10. [Trust Relationships and Authentication Models](#10-trust-relationships-and-authentication-models)
11. [Secrets Storage and Platform Solutions](#11-secrets-storage-and-platform-solutions)
12. [Key Rotation and Renewal Framework](#12-key-rotation-and-renewal-framework)
13. [Key Escrow and Recovery](#13-key-escrow-and-recovery)
14. [Abuse Detection and Anomaly Monitoring](#14-abuse-detection-and-anomaly-monitoring)
15. [Emergency Compromise Handling and Incident Response](#15-emergency-compromise-handling-and-incident-response)
16. [Audit, Logging and Accountability](#16-audit-logging-and-accountability)
17. [CI/CD Secrets Management](#17-cicd-secrets-management)
18. [Compliance Mapping](#18-compliance-mapping)
19. [Roles and Responsibilities](#19-roles-and-responsibilities)
20. [References](#20-references)
A. [Appendix A. Sample Integrations](#appendix-a-huawei-cyberark-and-kubernetes-implementation-profiles)

---

## 1. Purpose and Scope

This framework defines the architecture, control requirements, operational procedures, and technical standards for managing secrets and cryptographic keys across the enterprise.

It applies to all environments, including production, disaster recovery, staging, development, end-user platforms, infrastructure platforms, cloud services, CI/CD systems, security tooling, and third-party integrations.

Where a Hong Kong designated authority issues a sector-specific code of practice, that sectoral code should be read together with this framework and take precedence for any additional sector-specific obligations.

### 1.1 Objectives

- Protect authentication secrets, API keys, service credentials, certificates, SSH identities, and cryptographic keys.
- Establish a full lifecycle model from request through destruction.
- Define inventory, ownership, trust relationships, rotation, retirement, escrow, audit, and emergency response requirements.
- Standardize technical platforms for secure storage, issuance, retrieval, rotation, and monitoring.
- Align implementation and governance to NIST, OWASP, PCI DSS, and ISO-aligned control expectations.

### 1.2 In Scope

- API keys and bearer secrets.
- Username and password secrets for service use.
- OAuth2 and OIDC client secrets.
- Symmetric encryption keys, including DEKs, KEKs, and HMAC keys.
- Asymmetric keys, including TLS, mTLS, signing, and certificate authority keys.
- Service accounts and workload identities.
- SSH key pairs and SSH certificates.
- Break-glass credentials and escrow materials.
- Metadata, audit logs, policies, source and destination mappings, and operational evidence.

### 1.3 Out of Scope

- End-user password composition policy except where user credentials interact with privileged platforms.
- Proprietary HSM firmware internals.

---

## 2. Governing Standards and Frameworks

The framework is based on recognized guidance for cryptographic key management, secrets handling, auditability, and resilience.

| Standard / Framework | Relevance to this framework |
|---|---|
| NIST SP 800-57 Part 1 Rev. 5 | Primary source for key lifecycle, cryptoperiods, protection, archival, and destruction. |
| NIST SP 800-57 Part 2 | Governance and organizational practices for key management programs. |
| NIST SP 800-130 | Architectural model for cryptographic key management systems. |
| NIST SP 800-12 Rev. 1 | Introductory NIST security handbook guidance, including cryptography, key management, and foundational control rationale. |
| NIST SP 800-53 Rev. 5 | Core control catalog for access, audit, crypto, monitoring, incident response, and separation of duties. |
| NIST CSF 2.0 | Business-level structure across Govern, Identify, Protect, Detect, Respond, and Recover. |
| OWASP Key Management Cheat Sheet | Practical key storage, wrapping, trust store, rotation, and destruction guidance. |
| OWASP Secrets Management Cheat Sheet | Secret lifecycle, CI/CD, metadata, in-memory handling, and operational implementation patterns. |
| OWASP Logging Cheat Sheet | Logging content, integrity, privacy, and tamper-resistance guidance. |
| OWASP CI/CD Security Cheat Sheet | Pipeline hardening, runner isolation, approval and attribution controls. |
| PCI DSS v4.0 | Financial-sector cryptographic key management and audit expectations, especially for protected data environments. |
| ISO 27001 aligned controls | Asset management, access control, cryptography, secure operations, and vendor oversight. |
| FIPS 140-3 | Validation baseline for cryptographic modules and HSMs used for critical material. |
| NIST SP 800-63 Digital Identity Guidelines | Identity proofing, authentication, federation, authenticator lifecycle, and digital identity assurance guidance relevant to machine and service identities. |
| Hong Kong OCCICS Code of Practice (PCI(CS) Ordinance) | Regulatory baseline for CCS governance, management plans, risk assessment, audit, incident response, access control, cryptography, logging, and documentation expectations for CI operators in Hong Kong. |

### 2.1 Hong Kong regulatory overlay

For organizations governed in Hong Kong, the **Code of Practice pursuant to the Protection of Critical Infrastructures (Computer Systems) Ordinance** should be treated as a regulatory overlay on top of this standard. The Code is described as practical guidance rather than subsidiary legislation, but the Commissioner may use it as the reference point when assessing compliance and issuing directions.

| Hong Kong CoP topic | Practical implication for this standard |
|---|---|
| Board or senior-management endorsement and review | The computer-system security management plan should be formally endorsed and reviewed at least once every two years, and earlier after material changes. |
| Cross-reference and alternative controls | Where this framework is implemented through multiple policies and standards, maintain clear cross-references, and document compensating controls when exact requirements cannot be met. |
| Asset inventory and architecture documentation | Maintain up-to-date inventories, diagrams, upstream and downstream dependencies, asset owners, and supporting documentation for critical systems. |
| Access control and privileged access | Enforce least privilege, annual review of access rights, unique user IDs, separate privileged accounts or just-in-time elevation, and MFA where appropriate. |
| Cryptography and key management | Manage cryptographic keys across generation, storage, archive, retrieval, distribution, retirement, and destruction, and store keys separately from encrypted information. |
| Logging and retention | Log critical events including access to password and key files, privileged use, access-right changes, and audit-policy changes; retain logs for at least six months and protect them from alteration or deletion. |
| Remote access and network security | Encrypt remote sessions, implement MFA and logging, segment networks by trust level, and monitor critical nodes with IDS or IPS. |
| Incident readiness | Maintain emergency response plans, risk assessments, security audits, and notification processes for incidents and material changes. |

### 2.2 Control families most used

| Area | Main standards linkage |
|---|---|
| Inventory and ownership | NIST CM-8, AC-2, NIST CSF ID.AM |
| Access control and least privilege | NIST AC-2, AC-3, AC-5, AC-6 |
| Authenticator and digital identity lifecycle | NIST SP 800-63, NIST IA-5 |
| Authenticator management | NIST IA-5 |
| Key establishment and management | NIST SP 800-12, NIST SP 800-57, NIST SC-12, SC-13 |
| Data and key protection | NIST SC-28 |
| Audit and accountability | NIST AU-2, AU-3, AU-6, AU-9, AU-11, AU-12 |
| Monitoring and abuse detection | NIST SI-4, NIST CSF DE.CM |
| Incident response | NIST IR-4, IR-6, NIST CSF RS.MI |
| Backup, recovery, escrow | NIST CP-9, MP-6, RC.RP |

---

## 3. Definitions and Secret Type Taxonomy

A **secret** is a sensitive value that must be protected against unauthorized disclosure because possession or knowledge of that value can grant access, authenticate an identity, authorize an action, or enable other security-relevant operations. In practice, OWASP treats passwords, API keys, tokens, certificates, and cryptographic keys as secrets that require managed storage, access control, rotation, metadata, and audit. [web:46]

A **cryptographic key** is formally defined by NIST as a parameter used with a cryptographic algorithm that determines the algorithm's operation so that an entity with knowledge of the key can reproduce, reverse, or verify the operation, while one without the key cannot. NIST also describes cryptographic keys as values used to control cryptographic operations such as encryption, decryption, signature generation, or signature verification. [web:179][web:181]

In this framework, a **key** is therefore a specialized type of secret used for cryptographic operations, while a **secret** is the broader category that also includes passwords, bearer tokens, client secrets, and similar sensitive credentials. [web:46][web:179]

A **credential** is the secret or key material presented to authenticate a user, machine, workload, or service, and NIST SP 800-63 is used in this framework to distinguish identities, authenticators, federation, and authenticator lifecycle management from broader application-secret and data-encryption-key management. [web:160][web:170]

### 3.0 Why secrets and key management matter

NIST SP 800-12 states that the proper management of cryptographic keys is essential to the effective use of cryptography for security, and that the security of information protected by cryptography directly depends on the protection afforded to keys. It also states that key management includes the procedures and protocols, both manual and automated, used throughout the life cycle of keys, including generation, distribution, storage, entry, use, destruction, and archiving. [web:175]

NIST SP 800-53 SC-12 requires organizations to establish and manage cryptographic keys for required cryptography employed within information systems, which makes key management a control requirement and not just an implementation preference. [web:41]

OWASP's secrets-management guidance treats secrets as high-value operational assets because they grant access to systems and services, and it recommends centralized control, metadata, auditing, rotation, and automation to reduce exposure and operational error. [web:46]

If a secret is disclosed, the affected control is often **bypassed rather than attacked**, because an attacker can authenticate or authorize directly with the exposed value instead of exploiting a software flaw or cryptographic weakness. That is why secrets, machine-identity credentials, trust anchors, and high-assurance keys should be treated as high-value assets in threat models and as Tier-0 or equivalent material in zero-trust designs. [web:46][web:175][web:160]

Secret leakage is also a common breach path when secrets are hard-coded into scripts or code, committed to repositories, stored in insecure locations, or logged in plaintext. External guidance on secret leakage highlights hardcoding, public-repository exposure, insecure environment storage, and plaintext logging as recurring causes of compromise, which aligns with OWASP's emphasis on centralized secret handling and lifecycle control. [web:178][web:182][web:46]

For organizations governed in Hong Kong, the OCCICS Code of Practice reinforces this importance by requiring documented computer-system security management plans, access control, cryptographic protection, logging, risk assessments, audits, and incident handling for covered systems. Even though the Code is not a deep technical control catalog, it makes clear that these controls must be documented, governed, and evidenced. [page:5][page:6]

### 3.1 Secret classes

| Secret class | Typical examples | Primary use | Security characteristics |
|---|---|---|---|
| API secrets | API keys, webhook secrets, HMAC shared secrets | Service-to-service or external API authentication | Usually bearer or shared secret; tightly scoped and rotated frequently |
| Credential secrets | Service passwords, database credentials, client secrets | Authentication to systems or platforms | Prefer dynamic issuance over long-lived storage |
| Symmetric keys | AES DEKs, KEKs, HMAC keys | Encryption and integrity | Strong lifecycle and key hierarchy required |
| Asymmetric keys | TLS private keys, code signing keys, mTLS client keys | Authentication, signing, and encryption | Private keys require strongest protection |
| Trust anchors | CA roots, intermediate CA keys, SSH CA keys | Establish trust relationships | Limited issuance paths and strict custody |
| Machine identity material | Service accounts, workload tokens, SSH key pairs | Identity of systems and automation | Prefer short-lived, attested, or signed identities |
| Recovery materials | Escrow shards, emergency credentials | Recovery and continuity | Multi-party control and separate storage |

### 3.2 Secret sensitivity tiers

| Tier | Description | Examples | Baseline controls |
|---|---|---|---|
| Tier 1 - Critical | Compromise would materially impact core financial services, customer funds, or enterprise trust | CA private keys, KEKs, HSM master material, signing keys | HSM residency, no plaintext export, dual control, threshold recovery |
| Tier 2 - High | Compromise would provide privileged system or data access | Service account secrets, SSH private keys, OAuth client secrets, API keys | Central secret store, short TTL where possible, strong monitoring |
| Tier 3 - Medium | Sensitive but narrower blast radius | Lower-risk environment credentials, internal non-privileged tokens | Central storage, rotation, audit, no source code exposure |
| Tier 4 - Low | Sensitive configuration with limited security impact | Non-privileged configuration secrets | Managed storage and baseline audit |

### 3.3 Secret type matrix

| Secret type | Example | Source | Destination | Trust model | Default rotation |
|---|---|---|---|---|---|
| API key | Vendor payment API key | Payments platform | External provider | Bearer or HMAC with scope restriction | 90 days |
| OAuth client secret | OIDC confidential client secret | Application | Identity provider | OIDC / OAuth2 | 90 days |
| DB credential | Dynamic DB login | Vault | Database | Password or mTLS | 30 days or shorter TTL |
| Symmetric DEK | AES-256 data key | Crypto service | Data stores | Wrapped by KEK | 365 days or event driven |
| KEK | AES wrapping key | HSM | Secret store / crypto service | HSM-enforced | 730 days |
| TLS private key | RSA / EC server key | HSM or PKI engine | TLS endpoint | PKI chain | Certificate-driven |
| SSH identity | SSH cert or key pair | Vault SSH CA | Hosts / systems | SSH CA trust | 1 hour to 24 hours if cert-based |
| Service account token | Workload identity token | Cloud IAM / Vault | Internal service | OIDC / WIF / AppRole | Minutes to 1 hour |
| Escrow shard | Shamir shard | Key ceremony | Custodian | M-of-N threshold | Reviewed annually |

### 3.4 Identity, account, and cryptographic material distinctions

The framework distinguishes between an **account or principal**, the **credential or key used to authenticate as that principal**, and the **cryptographic key used to protect data**. This distinction is important because the lifecycle, custody model, monitoring pattern, and compromise response are different for each class of material.

| Class | What it represents | Typical examples | Primary purpose | Lifecycle emphasis | Handling standard |
|---|---|---|---|---|---|
| Service account | A non-human principal with permissions in a platform or application | Cloud IAM service account, DB service account, Kubernetes service account, application technical user | Authorization boundary and ownership of privileges | Provisioning, approval, periodic entitlement review, deprovisioning | Govern as an identity object with owner, scope, review date, and role mapping |
| Machine identity credential | The material a system uses to prove it is a known workload or host | mTLS client certificate and private key, SSH host certificate, workload identity token, AppRole secret ID, signed instance identity | Authentication of a workload, VM, node, or service | Issuance, trust validation, renewal, revocation, attestation, runtime protection | Prefer short-lived, attested, or certificate-based identity rather than long-lived static shared secrets |
| Service authentication secret | A secret that lets a workload authenticate to another system on behalf of its service account | Client secret, service password, API token, bootstrap secret, database password | Service-to-service authentication | Rotation, lease TTL, dependency mapping, emergency replacement | Central store, scoped delivery, short lifetime where possible |
| Data protection key | A cryptographic key used to encrypt, decrypt, or integrity-protect information | DEK, KEK, HMAC key, tokenization key | Confidentiality and integrity protection | Cryptoperiod management, wrapping hierarchy, archival, destruction, re-encryption | HSM or KMS anchored, strict key hierarchy, store separately from protected data |
| Trust anchor or issuing key | A key used to establish or delegate trust | CA root key, intermediate CA key, SSH CA key, code-signing root | Trust establishment and chain validation | Ceremony-based custody, issuance governance, emergency rollover | Tier 1 controls, dual control, threshold recovery, restricted use |
| Recovery material | Material used only for continuity or emergency recovery | Escrow shard, break-glass credential, unseal share | Recovery and continuity | Ceremony, testing, physical or split storage, post-use rotation | Never used for day-to-day runtime access |

#### Practical distinction rules

NIST SP 800-63 is especially useful for the identity side of this framework because it separates identity proofing, authentication, federation, and authenticator lifecycle management, which helps distinguish identities and authenticators from general-purpose application secrets or data-encryption keys.

- A **service account** is the identity object that owns permissions.
- A **machine identity credential** is the proof that the workload is that identity.
- A **service authentication secret** is often what the workload presents to another system after it has assumed or been bound to the service account.
- A **data protection key** does not identify the workload; it protects data or verifies integrity.
- A **trust anchor** should not be treated like a routine application secret because compromise affects all dependent identities or certificates.

#### Example mappings

| Example | Identity object | Authentication material | Protected function |
|---|---|---|---|
| Kubernetes workload calling an internal API | Kubernetes service account | OIDC service account token or mTLS client certificate | Authenticates workload to Vault, API gateway, or mesh |
| ECS virtual machine in Huawei VPC | VM or workload identity | IAM token, instance-bound credential, or retrieved bootstrap secret | Authenticates VM to CSMS, database, or internal services |
| Application using a DB login | Service account in database | Dynamic DB password or client certificate | Authenticates application to database |
| Storage encryption service | Crypto service identity | Service cert or workload token | Uses DEK and KEK to protect stored data |
| Internal PKI issuing TLS certs | CA service identity | CA private key under HSM control | Establishes trust for issued certificates |

---

## 4. Lifecycle Model

All secrets and keys must follow a controlled lifecycle with explicit ownership and evidence.

### 4.1 Enterprise lifecycle phases

```mermaid
flowchart LR
    A[Need identified] --> B[Design and approval]
    B --> C[Generate or issue]
    C --> D[Register in inventory]
    D --> E[Distribute securely]
    E --> F[Activate]
    F --> G[Use and monitor]
    G --> H[Rotate or renew]
    H --> I[Deactivate]
    I --> J[Archive if required]
    J --> K[Destroy or zeroize]
    G --> L[Compromise event]
    L --> M[Revoke and contain]
    M --> H
```

### 4.2 Lifecycle requirements by phase

| Phase | Mandatory activities |
|---|---|
| Need identified | Business need documented, data classification known, owner assigned, trust relationship defined |
| Design and approval | Algorithm and key type selected, source and destination mapped, platform pattern selected |
| Generate or issue | CSPRNG or HSM-backed generation, metadata bound, cryptoperiod assigned |
| Register in inventory | Unique ID, owner, environment, source, destination, trust scope, storage location, sensitivity tier |
| Distribute securely | TLS 1.3 or equivalent secure transport, wrapped or lease-based delivery, no plaintext tickets or chat transfer |
| Activate | Access policy enforced, consumer validated, alerting enabled |
| Use and monitor | JIT retrieval, in-memory minimization, logging, anomaly monitoring |
| Rotate or renew | Scheduled and event-driven rotation, dependent systems updated, test and rollback defined |
| Deactivate | New issuance blocked, legacy decrypt or verify permitted only where needed |
| Archive | Archived only where justified by business, legal, or decryptability needs |
| Destroy or zeroize | Secure destruction per media and platform type, evidence captured |
| Compromise handling | Immediate revocation, blast radius review, cascading rotation, incident record |

### 4.3 State model

| State | Description |
|---|---|
| Requested | Awaiting design approval and owner confirmation |
| Generated | Secret exists but is not yet active |
| Active | Used in production or live service path |
| Suspended | Temporarily blocked pending review or incident |
| Deprecated | Replaced by a newer version; read-only or drain phase |
| Archived | Retained only for decrypt, verify, legal, or recovery needs |
| Destroyed | Removed or zeroized with evidence |
| Compromised | Explicit incident state requiring emergency handling |

---

## 5. Secret Type Requirements

This section defines operational controls by secret type.

### 5.1 API keys and shared application secrets

| Requirement area | Standard |
|---|---|
| Generation | Minimum 256-bit entropy from approved CSPRNG |
| Storage | Central secret store only; never in code, tickets, chat, or unencrypted config |
| Scope | Limit by API, action, IP range, environment, and account |
| Delivery | Returned once or retrieved at runtime via SDK or agent |
| Rotation | 30 to 90 days based on provider support and risk |
| Monitoring | Baseline volume, geolocation, ASN, method, and failure patterns |
| Emergency action | Immediate revoke and vendor notification where applicable |

### 5.2 Credentials and passwords used by systems

| Requirement area | Standard |
|---|---|
| Pattern | Prefer dynamic credentials over static passwords |
| Length and randomness | 20+ characters for non-human secrets, generated not human chosen |
| Interactive login | Disabled for service accounts unless exception approved |
| Rotation | 30 to 90 days if static; minutes to hours if dynamic |
| Storage | Vault or equivalent; no environment variable for persistent privileged credentials |
| Recovery | Re-issue rather than disclose historical value |

### 5.3 Symmetric keys

| Requirement area | Standard |
|---|---|
| Algorithm | AES-256 for encryption, HMAC-SHA-256 or stronger for integrity |
| Hierarchy | DEK wrapped by KEK, KEK protected by HSM master material |
| Export | KEKs and master keys non-exportable wherever possible |
| Rotation | DEKs annually or sooner, KEKs every two years or sooner |
| Re-encryption | Defined process for rekey and data re-encryption |
| Destruction | Zeroize keys and revoke associated wrapping relationships |

### 5.4 Asymmetric keys

| Requirement area | Standard |
|---|---|
| Algorithms | RSA-4096 or EC P-384 for critical use; Ed25519 acceptable for SSH identity where approved |
| Generation | Prefer HSM or CA engine generation |
| Private key exposure | Private keys must not be exported from HSM for Tier 1 use cases |
| Public distribution | Through trust store or certificate repository |
| Renewal | New key pair for renewals unless managed exception |
| Revocation | CRL, OCSP, or equivalent trust-store update process |

### 5.5 Key hierarchy

```mermaid
flowchart TD
    MK[Master key in HSM] --> KEK[Key encryption key]
    KEK --> DEK[Data encryption key]
    DEK --> DATA[Protected data]
    KEK --> SECRETSTORE[Wrapped secrets in vault or store]
```

---

## 6. Service Account Framework

Service accounts are non-human identities used by applications, automation, CI/CD jobs, and infrastructure components.

### 6.1 Requirements

- One service account per workload or bounded function.
- No shared service accounts across unrelated services.
- No standing interactive login unless explicitly approved.
- Prefer short-lived tokens or dynamic credentials over static secrets.
- Every service account must have an owner, purpose, environment, and access review date.

### 6.2 Service account types

| Type | Preferred auth model | Typical TTL |
|---|---|---|
| Cloud workload identity | OIDC / workload identity federation | Minutes to 1 hour |
| Kubernetes workload | Service account token with projected short-lived token or Vault auth | 1 hour |
| Database automation account | Vault dynamic DB secret | 15 minutes to 24 hours |
| Broker or middleware account | mTLS cert or short-lived token | 24 hours to 90 days depending on platform |
| Legacy system account | Static secret with exception | 30 to 90 days |

### 6.3 Lifecycle diagram

```mermaid
flowchart LR
    A[Request new service account] --> B[Business justification and owner]
    B --> C[Security and IAM approval]
    C --> D[Provision identity and policy]
    D --> E[Register inventory and monitoring]
    E --> F[Runtime use]
    F --> G[Periodic review]
    G --> H[Rotate or renew]
    H --> I[Deprovision when no longer needed]
    F --> J[Anomaly detected]
    J --> K[Suspend or revoke]
    K --> H
```

### 6.4 Mandatory controls

| Control | Requirement |
|---|---|
| Ownership | Named owner and backup owner required |
| Scope | Resource-level least privilege only |
| Review | Review every 90 days, earlier for privileged accounts |
| Dormancy | Flag no-use >30 days for decommission review |
| Monitoring | Log all auth, role assumption, token issuance, and privileged operations |
| Secrets pattern | Prefer issued tokens or dynamic passwords, not hard-coded values |

---

## 7. SSH Key Pair Framework

SSH identity for systems and services must move from static long-lived keys to short-lived SSH certificates wherever feasible.

### 7.1 Target architecture

```mermaid
flowchart LR
    A[User or service] -->|Authenticate to Vault| B[Vault SSH CA]
    B -->|Issue short-lived SSH certificate| A
    A -->|Present cert| C[Target host]
    C -->|Validate against trusted CA public key| D[Access granted or denied]
```

### 7.2 SSH use cases

| Use case | Recommended pattern | TTL |
|---|---|---|
| Operator administrative access | SSH certificate issued after MFA and policy check | 1 hour |
| CI/CD deployment to hosts | Per-job SSH certificate | Less than 1 hour |
| Service-to-system trust | Host or service SSH certificate | 1 to 24 hours |
| Legacy device without cert support | Static key pair under exception | Max 90 days |

### 7.3 SSH control requirements

| Control | Requirement |
|---|---|
| Key type | Ed25519 preferred for SSH identity unless compatibility requires RSA-4096 |
| CA trust | Hosts trust CA public key, not many individual static keys |
| Principals | Certificate principals must be explicit and non-wildcard |
| Private key handling | No storage in code repos or shared home directories |
| Agent use | Use ephemeral agents and clear keys after session or job completion |
| Incident handling | Revoke or rotate CA and redistribute trust promptly for widespread compromise |

---

## 8. Inventory and Metadata Schema

A complete inventory is mandatory for lifecycle management, blast-radius analysis, and audit evidence.

### 8.1 Required metadata fields

| Field | Required | Description |
|---|---|---|
| key_id | Yes | Globally unique identifier |
| name | Yes | Human-readable secret name |
| type | Yes | API, credential, symmetric, asymmetric, service account, SSH, escrow |
| subtype | Yes | More specific classification |
| owner_team | Yes | Responsible team |
| owner_contact | Yes | Accountable individual or shared mailbox |
| environment | Yes | Production, DR, staging, development |
| source | Yes | Where the secret originates |
| destination | Yes | Consuming systems or services |
| trust_model | Yes | OIDC, mTLS, PKI, HMAC, AppRole, etc. |
| storage_location | Yes | Vault path, KMS ARN, HSM partition, or similar |
| algorithm | Conditional | Required for cryptographic keys |
| sensitivity_tier | Yes | Tier 1 to Tier 4 |
| created_at | Yes | Creation timestamp |
| expires_at | Yes | Expiry or next review timestamp |
| last_rotated | Yes | Last completed rotation |
| rotation_policy | Yes | Schedule or event condition |
| status | Yes | Requested, active, deprecated, archived, destroyed, compromised |
| escrow_reference | Conditional | Recovery material linkage |
| in_memory_protection | Conditional | HSM-only, enclave, mlock, ephemeral, standard |
| compliance_tags | Yes | PCI, SOX, ISO, etc. |

### 8.2 Inventory operating requirements

- Daily reconciliation across Vault, KMS, PKI, CMDB, and cloud IAM.
- Discovery scanning for hard-coded secrets in repositories, images, and configuration stores.
- Owner recertification every 90 days for production entries.
- Mandatory dependency maps for Tier 1 and Tier 2 secrets.
- Automatic orphan detection for secrets with no owner, no consumer, or no recent activity.

### 8.3 Relationship diagram

```mermaid
erDiagram
    OWNER ||--o{ SECRET : owns
    SECRET ||--o{ CONSUMER : used_by
    SECRET ||--o{ ROTATION_POLICY : governed_by
    SECRET ||--o{ AUDIT_EVENT : generates
    SECRET ||--o| ESCROW_RECORD : may_have
    SECRET {
      string key_id
      string name
      string type
      string subtype
      string owner_team
      string environment
      string source
      string trust_model
      string status
    }
    CONSUMER {
      string consumer_id
      string destination
      string system_type
    }
    ROTATION_POLICY {
      string policy_id
      string frequency
      string trigger_type
    }
    AUDIT_EVENT {
      string event_id
      string event_type
      string timestamp
    }
    ESCROW_RECORD {
      string escrow_id
      string threshold
    }
```

### 8.4 Operational inventory for acquisition and rotation methods

In enterprise environments, the inventory must record not only **what** the secret or key is, but also **how it is obtained, how it is rotated, who coordinates it, and what operational dependency exists**. OWASP recommends storing metadata such as intended consumers, purpose, type, and when manual rotation is required, and NIST key-management guidance emphasizes inventories, accountability, audit, and documented distribution methods. [web:46][web:32][web:154]

#### 8.4.1 Acquisition and rotation method classification

| Operational pattern | Typical examples | How obtained | Rotation method | Documentation requirement |
|---|---|---|---|---|
| Platform-issued dynamic secret | Vault DB credential, short-lived cloud token | Retrieved programmatically at runtime from secret platform | Automatic by lease renewal or re-issuance | Record platform, auth method, TTL, and consuming workload |
| Internal automated secret | Internal API key generated by enterprise platform | Generated by internal secret manager, PKI, or KMS | Scheduled automation or event-driven workflow | Record job, trigger, rollback path, and dependency list |
| External party managed secret | Vendor API key, partner certificate, external client secret | Received from external portal, secure channel, or partner workflow | External-party process, with internal calendar and owner follow-up | Record external owner, portal URL, contact path, issue date, expiry, and coordination lead |
| Console-regenerated secret | API gateway key, SaaS console secret, partner portal token | Created or regenerated through admin console | Manual or semi-manual console action | Record console URL, admin role, exact steps, approval path, and evidence link |
| Manual coordinated certificate or key | mTLS certificate exchange, SSH trust onboarding, bilateral key ceremony | Coordinated with another internal or external team | Manual renewal with change window and validation steps | Record counterparties, trust boundary, validation steps, notice period, and rollback plan |
| Static exception secret | Legacy app password or device secret with no API | Manual placement or limited scripting | Manual rotation | Record exception approval, compensating controls, storage path, and retirement target date |

#### 8.4.2 Required operational inventory fields

| Field | Purpose |
|---|---|
| Acquisition method | Distinguishes runtime retrieval, API issuance, portal download, manual exchange, or console generation |
| Rotation method | Distinguishes automatic, event-driven, scheduled manual, vendor-driven, or emergency-only rotation |
| Rotation owner | Names the accountable internal team for execution and evidence capture |
| Coordinating party | Identifies external vendor, internal platform owner, or partner team if coordination is required |
| Rotation interface | Stores API endpoint, vault path, console URL, runbook name, or ticket workflow reference |
| Notice period | Captures how much lead time is required for partner or business coordination |
| Validation method | Records how the new secret is tested before promotion, such as connectivity test, handshake test, or application smoke test |
| Rollback method | Documents whether the previous version remains valid, for how long, and how fallback is executed |
| Evidence location | Links to ticket, pipeline run, job ID, ceremony record, or audit log query |
| Dependency set | Lists impacted applications, APIs, gateways, partners, hosts, or certificates |

#### 8.4.3 Source and rotation register template

| ID | Secret / Key | Source type | How obtained | Rotation type | Rotation owner | Coordination required | Interface / system | Validation | Rollback | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| SK-001 | payments-db-cred | Internal dynamic | Vault DB engine | Automatic lease re-issue | Platform team | No | `vault://database/payments` | DB login test | Previous lease overlap | Runtime only |
| SK-002 | partner-api-key | External vendor | Partner portal download | Manual scheduled | Payments team | Yes, vendor | Vendor console URL | Test API call | Previous key valid for 7 days | External dependency |
| SK-003 | gateway-client-secret | Internal console | API gateway admin console | Semi-manual regenerate | API platform team | No | Gateway admin UI | Client auth test | Prior version drain window | Needs CAB window |
| SK-004 | bilateral-mtls-cert | External coordinated | CA or partner exchange | Manual coordinated renewal | Security engineering | Yes, counterparty | PKI portal + ticket | TLS handshake and app test | Old cert overlaps until cutover | Formal runbook |
| SK-005 | legacy-device-password | Static exception | Manual secure entry | Manual | Infra ops | Maybe | Local device workflow | Login test | Manual restore | Exception tracked |

#### 8.4.4 Documentation requirements for design documents

Every solution design document should include a **Secrets and Keys Register** plus an **Operational Rotation Matrix**. OWASP API inventory guidance recommends documenting integrated services, their role, exchanged data, and sensitivity, which is directly applicable to external-party secrets, API gateway credentials, and partner trust relationships. [web:140][web:46]

The design document should state, for each secret or key, whether the lifecycle is fully automated, partially automated, externally coordinated, or manual by exception. It should also identify which secrets are internal-only, externally issued, console-generated, or ceremony-based so operations teams know how to rotate them without discovering the process during an incident. [web:46][web:32]


---

## 9. Encryption Requirements

All secrets must be encrypted at rest. Protection in transit and, for sensitive classes, in memory is also required.

### 9.1 Encryption at rest

| Tier | Requirement |
|---|---|
| Tier 1 | HSM-backed, non-exportable where supported, AES-256 or platform equivalent |
| Tier 2 | Secret store encryption with HSM-backed master key or KMS CMK |
| Tier 3 | Managed encryption with enterprise-approved key management |
| Tier 4 | Managed platform encryption minimum baseline |

### 9.2 Encryption in transit

- TLS 1.3 minimum for secret retrieval and management traffic unless documented platform limitation exists.
- Mutual TLS required for privileged service-to-service secret retrieval paths where feasible.
- No plaintext transmission over email, ticketing tools, instant messaging, or shell history.

### 9.3 Encryption and protection in memory

| Sensitivity | Requirement |
|---|---|
| Tier 1 | Cryptographic operation should occur fully inside HSM or enclave whenever possible |
| Tier 1 and Tier 2 | Use `mlock()` or platform equivalent to prevent swap for temporary in-memory key material |
| Tier 1 and Tier 2 | Zeroize memory after use and avoid immutable string types for key bytes |
| Tier 2 | Prefer sidecar or agent with short-lived in-memory cache over application-persistent copies |
| Tier 3 | Minimize retention time and avoid debug or crash dump exposure |

### 9.4 Memory handling by implementation model

| Model | Requirement |
|---|---|
| HSM-backed signing or unwrap | Preferred for private keys, KEKs, and trust anchors |
| Nitro enclave / SGX / equivalent | Approved for workloads needing protected in-memory operations |
| Agent sidecar in RAM-backed volume | Approved for service secrets and certificates |
| Direct application memory | Allowed only when higher-assurance patterns are impractical |

---

## 10. Trust Relationships and Authentication Models

Trust models define how systems prove identity before receiving or using secrets.

### 10.1 Approved trust models

| Trust model | Typical use cases | Notes |
|---|---|---|
| PKI / certificate chain | TLS, mTLS, signing, SSH CA trust | Best for strong machine identity and trust path validation |
| OIDC / OAuth2 | Workload identity, CI/CD federation, cloud IAM | Prefer short-lived tokens and audience restrictions |
| HMAC shared secret | API request signing, webhook validation | Strong protection required because both parties share material |
| Vault AppRole | Legacy or non-cloud machine auth to Vault | Secret ID must be wrapped and tightly distributed |
| Workload identity federation | Cloud-native machine identity | Preferred over static cloud access keys |
| Threshold trust | Escrow, break-glass, recovery | Used only for recovery and governance workflows |

### 10.2 Trust flow

```mermaid
flowchart LR
    A[Workload or user] --> B[Authenticate with approved method]
    B --> C[Policy evaluation]
    C --> D[Issue short-lived token or certificate]
    D --> E[Retrieve or use secret]
    E --> F[Audit and telemetry]
    C --> G[Deny]
    G --> F
```

### 10.3 Trust requirements

- Every secret must document the trust model used to obtain it.
- Every machine identity must bind to a concrete workload, service, namespace, project, or host.
- No wildcard trust relationship for privileged secrets.
- Trust anchors must be inventoried, reviewed, and protected with Tier 1 controls.

---

## 11. Secrets Storage and Platform Solutions

The enterprise should standardize on a primary secret platform and limited secondary platform patterns.

### 11.1 Preferred platform pattern

| Capability | Preferred solution |
|---|---|
| Centralized secret storage | HashiCorp Vault Enterprise or equivalent enterprise secret platform |
| HSM-backed custody | CloudHSM, dedicated HSM, or equivalent integrated via PKCS#11 |
| Cloud-native secrets | AWS Secrets Manager, Azure Key Vault, or GCP Secret Manager where aligned to platform strategy |
| Cryptographic key operations | Cloud KMS and/or Vault transit engine with HSM anchoring |
| PKI and SSH certificate issuance | Vault PKI and SSH engines or enterprise CA platform |

### 11.2 Platform capabilities required

| Capability | Requirement |
|---|---|
| Auth methods | OIDC, AppRole, cert auth, cloud identity integration |
| Dynamic secrets | Database, cloud, and service credential issuance |
| Leases | Short-lived tokens and renewable leases |
| Versioning | Support current, pending, previous, and deprecated versions |
| Rotation | Automated rotation hooks or native rotation capability |
| Audit | Structured, immutable-capable audit streams |
| HA and DR | Multi-node availability and cross-region replication |
| Policy | Fine-grained RBAC or ABAC with namespaces or clear segregation |

### 11.3 Reference architecture

```mermaid
flowchart TB
    subgraph VaultCluster[Vault or enterprise secret platform]
        V1[Active node]
        V2[Standby node]
        V3[DR replica]
    end
    HSM[HSM or CloudHSM] --> VaultCluster
    Apps[Applications and services] --> VaultCluster
    CICD[CI/CD pipelines] --> VaultCluster
    Ops[Operators and admins] --> VaultCluster
    VaultCluster --> SIEM[SIEM and immutable log archive]
    VaultCluster --> CMDB[Inventory and metadata system]
```

### 11.4 Handling approaches by secret type

| Secret type | Recommended handling approach |
|---|---|
| API keys | Vault KV or managed secret store with versioning and scoped distribution |
| Dynamic DB credentials | Vault dynamic secret engine |
| Encryption keys | HSM-backed KMS or Vault transit with HSM anchoring |
| TLS certs | PKI engine with automated issuance and renewal |
| SSH identity | Vault SSH CA or equivalent certificate authority model |
| Break-glass credentials | Separate protected store with multi-party recovery and offline backup |

---

## 12. Key Rotation and Renewal Framework

Rotation must be both scheduled and event-driven.

### 12.1 Rotation principles

- Rotate on cryptoperiod expiry.
- Rotate immediately on compromise, suspected compromise, or personnel risk event.
- Maintain version history sufficient for drain, rollback, or decrypt continuity.
- Test rotation before full promotion where systems depend on continuity.

### 12.2 Rotation schedule

| Secret type | Standard rotation |
|---|---|
| Internal API keys | 30 days |
| External or vendor API keys | 90 days or provider maximum lower limit |
| Static service account passwords | 30 to 90 days |
| Dynamic service credentials | Per lease or under 24 hours |
| OAuth client secrets | 90 days |
| SSH certificates | 1 hour to 24 hours |
| Static SSH keys under exception | 90 days maximum |
| TLS server certs | Renew before expiry, typically 30 days before |
| DEKs | 365 days or on significant data reclassification |
| KEKs | 730 days or on event |
| CA or signing keys | According to PKI or signing policy, typically 1 to 2 years |

### 12.3 Rotation flow

```mermaid
flowchart LR
    A[Rotation trigger] --> B[Generate new secret or key]
    B --> C[Stage as pending version]
    C --> D[Test connectivity or cryptographic function]
    D --> E[Promote to current]
    E --> F[Drain old version]
    F --> G[Revoke and destroy old version]
    G --> H[Update inventory and audit evidence]
```

### 12.4 Rotation failure handling

| Condition | Requirement |
|---|---|
| Test failure | Do not promote pending version |
| Production rollback | Defined drain and rollback period required |
| Repeated failure | Escalate to security and application owner |
| Expired secret still in use | Raise incident and prioritize remediation |

### 12.5 Operational rotation considerations

Rotation is an operational process as much as a cryptographic one. OWASP notes that manual rotation is error-prone and should be automated where possible, while NIST-oriented key-management guidance recognizes manual distribution, automated distribution, and key agreement as distinct operational models that need accountability and audit coverage. [web:46][web:32]

#### 12.5.1 Rotation execution models

| Execution model | When to use | Operational considerations |
|---|---|---|
| Fully automated | Dynamic secrets, internally issued certs, platform-generated API credentials | Requires tested automation, health checks, rollback, and monitoring of failed rotations |
| Semi-automated | Console-regenerated keys or keys updated by runbook with API assistance | Requires step-by-step runbook, dual control where privileged, and evidence capture |
| External coordinated | Partner credentials, third-party API keys, bilateral certificates | Requires notice period, named counterpart, test window, and explicit overlap strategy |
| Manual exception | Legacy platforms or non-API devices | Requires exception approval, compensating controls, shorter review cycle, and retirement plan |
| Emergency rotation | Suspected compromise or urgent platform issue | Requires pre-approved playbook, priority contact list, and blast-radius mapping |

#### 12.5.2 Minimum runbook content

| Runbook element | What to document |
|---|---|
| Trigger | Scheduled date, expiry threshold, incident, or vendor notice |
| Preconditions | Required approvals, access roles, maintenance window, and backup status |
| Source of new value | Whether the new secret is generated internally, regenerated in console, received from partner, or pulled from vault |
| Change steps | Exact sequence to create, stage, test, promote, and retire |
| Downstream updates | Which applications, gateways, jobs, hosts, or certificates must be updated |
| Validation | Functional test, handshake test, authentication test, and monitoring check |
| Rollback | Conditions for rollback, validity period of prior version, and restoration steps |
| Evidence | Ticket ID, job ID, screenshots where allowed, audit log references, and approver |

#### 12.5.3 Internal versus external ownership model

| Secret relationship | Internal owner responsibility | Additional requirement |
|---|---|---|
| Internal-only secret | Own creation, rotation, validation, and retirement | Automate by default |
| Externally issued secret | Track issue source, validity, and external contact | Maintain partner coordination record and notice periods |
| Shared trust secret or certificate | Coordinate cutover timing with all relying parties | Maintain overlap window and counterparty validation evidence |
| Console-managed secret | Track which console and admin role can regenerate | Maintain screenshots or procedural evidence where permitted |
| Manual exception secret | Track why automation is not possible | Maintain compensating controls and retirement target |


---

## 13. Key Escrow and Recovery

Escrow is permitted only where justified by business, legal, recovery, or decryptability requirements.

### 13.1 Escrow principles

- Not every secret should be escrowed.
- Escrow is most applicable to keys whose loss would prevent lawful recovery of protected data or operation of critical trust systems.
- Escrowed material must be protected with controls at least as strong as operational custody.

### 13.2 Escrow applicability

| Secret type | Escrow policy |
|---|---|
| HSM master or unseal material | Mandatory threshold recovery |
| KEKs protecting critical datasets | Mandatory if loss would prevent recovery |
| CA root and code-signing keys | Mandatory with strong governance |
| SSH CA private key | Prefer HSM backup and rotation planning; strict recovery controls |
| API keys and standard service credentials | Re-issue preferred; escrow generally not required |
| Dynamic credentials | Escrow not required |

### 13.3 Recovery ceremony flow

```mermaid
flowchart LR
    A[Declare recovery event] --> B[Verify incident authority]
    B --> C[Assemble approved custodians]
    C --> D[Retrieve threshold shares]
    D --> E[Reconstruct recovery material in controlled environment]
    E --> F[Restore platform or decrypt archived material]
    F --> G[Rotate all accessed recovery-sensitive material]
    G --> H[Document ceremony and archive evidence]
```

### 13.4 Custodian requirements

| Requirement | Standard |
|---|---|
| Threshold model | Minimum 3-of-5 for critical recovery material |
| Separation | Custodians from different teams and locations |
| Storage | Tamper-evident and geographically separated |
| Testing | Annual recovery validation |
| Evidence | Signed ceremony record and post-event rotation |

---

## 14. Abuse Detection and Anomaly Monitoring

Secrets management is incomplete without detection of misuse, abuse, or drift from expected behavior.

### 14.1 Abuse scenarios

| Secret type | Abuse case |
|---|---|
| API keys | Sudden volume spike, access from new ASN, method misuse |
| Service accounts | Login from unexpected IP or region, dormant account reuse |
| SSH identities | Cert issuance for unexpected principal, login to non-approved hosts |
| Symmetric keys | Unusual decrypt or unwrap activity volume |
| Vault admin actions | Unexpected audit-device changes, auth method changes, namespace creation |
| Recovery materials | Access outside formal ceremony |

### 14.2 Detection architecture

```mermaid
flowchart TB
    Vault[Vault audit logs] --> SIEM[SIEM / analytics]
    Cloud[Cloud IAM and KMS logs] --> SIEM
    SSH[SSH auth logs] --> SIEM
    CICD[CI/CD logs] --> SIEM
    Runtime[Runtime and host telemetry] --> SIEM
    SIEM --> Alert[Alerting and response automation]
    Alert --> SOCTeam[SOC investigation]
    Alert --> AutoContain[Auto revoke or quarantine where approved]
```

### 14.3 Detection controls

| Control | Requirement |
|---|---|
| Baselines | Profile normal use by secret, service, IP, time, and operation |
| Correlation | Join identity, secret, source, destination, and policy outcome |
| Real-time alerting | P1 and P2 conditions routed to SOC and on-call responders |
| Auto containment | Approved for clearly malicious events such as revoked-token use |
| Review | Weekly rule tuning and quarterly baseline review |

---

## 15. Emergency Compromise Handling and Incident Response

Every secret type requires a documented compromise playbook.

### 15.1 Response stages

```mermaid
flowchart LR
    A[Detect] --> B[Validate]
    B --> C[Contain]
    C --> D[Revoke]
    D --> E[Rotate dependents]
    E --> F[Recover service]
    F --> G[Post-incident review]
```

### 15.2 Minimum playbook requirements

| Step | Requirement |
|---|---|
| Detect | Alert from SIEM, operator, platform, or vendor |
| Validate | Confirm event and determine scope |
| Contain | Disable access, suspend account, isolate affected paths |
| Revoke | Revoke secret, certificate, lease, token, or trust path |
| Rotate dependents | Replace all linked or downstream credentials |
| Recover | Validate service health and trust restoration |
| Review | Root cause, timeline, lessons learned, control updates |

### 15.3 Revocation SLAs

| Secret type | Revocation target |
|---|---|
| API key or service credential | 5 to 30 minutes |
| SSH certificate or SSH identity | 15 minutes |
| TLS cert or trust path | 15 minutes to 4 hours depending on platform |
| DEK or application data key | 30 minutes to 4 hours |
| KEK or higher trust anchor | Emergency procedure with executive escalation |

---

## 16. Audit, Logging and Accountability

Logging is mandatory for lifecycle evidence, investigations, control assurance, and abuse detection.

### 16.1 Events that must be logged

| Category | Examples |
|---|---|
| Secret lifecycle | Create, read, issue, renew, rotate, revoke, destroy |
| Auth events | Successful and failed authentication, token issuance, token renewal |
| Policy events | Create, modify, delete policy or auth method |
| Crypto operations | Encrypt, decrypt, sign, verify, wrap, unwrap |
| Platform admin | Unseal, seal, namespace changes, mount enable or disable |
| Recovery events | Break-glass invocation, escrow access, recovery ceremony steps |
| Detection events | Deny decisions, anomaly triggers, abuse flags |

### 16.2 Audit record fields

| Field | Description |
|---|---|
| event_id | Unique identifier |
| timestamp | ISO 8601 UTC time |
| actor_identity | User, service, role, or pipeline |
| source_ip | Request origin |
| auth_method | How the actor authenticated |
| secret_id | Inventory-linked identifier |
| operation | Requested action |
| outcome | Success, denied, error |
| policy_matched | Policy or control decision reference |
| request_id | Correlation identifier |
| environment | Environment or namespace |

### 16.3 Audit pipeline

```mermaid
flowchart LR
    A[Vault and secret platforms] --> B[Secure log shippers]
    C[Cloud IAM and KMS logs] --> B
    D[SSH and host logs] --> B
    E[CI/CD logs] --> B
    B --> F[SIEM hot tier]
    F --> G[Immutable warm and cold archive]
    F --> H[Correlation rules and dashboards]
```

### 16.4 Logging requirements

| Control | Requirement |
|---|---|
| Content | Structured JSON or equivalent machine-readable format |
| Redaction | Never log plaintext secrets, private keys, access tokens, or escrow material |
| Integrity | Tamper detection and immutable storage for retained audit evidence |
| Retention | 90 days hot, 1 year warm, 7 years cold unless stricter regulation applies |
| Access | Read access limited to security, compliance, and approved investigators |
| Audit of auditing | Changes to audit configuration must themselves be logged and alerted |

---

## 17. CI/CD Secrets Management

CI/CD platforms are privileged control points and must be treated as production systems.

### 17.1 Secure delivery patterns

| Pattern | Suitability |
|---|---|
| Runtime retrieval by workload identity | Preferred pattern |
| Pipeline injects secret at deploy time | Exception for legacy platforms |
| Encrypted secret in source control | Acceptable only when decryption occurs at destination and keys are separated by environment |

### 17.2 CI/CD controls

| Control | Requirement |
|---|---|
| Runner isolation | Separate production runners and prevent cross-tenant reuse |
| Identity | Use OIDC or equivalent short-lived identity to obtain deployment permissions |
| Approvals | Require review for pipeline definition changes and production secret access changes |
| Logging | Attribute every run to user, repo, branch, commit, and job |
| Output hygiene | Mask logs and scan logs and artifacts for accidental secret exposure |
| Environment separation | Unique secrets and trust boundaries per environment |

### 17.3 Reference flow

```mermaid
flowchart LR
    A[Developer commit] --> B[Pull request review]
    B --> C[Pipeline execution]
    C --> D[Short-lived pipeline identity]
    D --> E[Deploy workload metadata]
    E --> F[Runtime workload starts]
    F --> G[Workload retrieves secret directly]
    G --> H[Use in memory only]
    C --> I[Pipeline audit logs]
    G --> I
```

---

## 18. Compliance Mapping

This framework can be mapped to core external standards for audit, risk, and regulator-facing evidence.

### 18.1 Crosswalk

| Framework area | NIST 800-53 | NIST CSF 2.0 | OWASP | PCI / ISO aligned area |
|---|---|---|---|---|
| Inventory and ownership | CM-8, AC-2 | ID.AM | Secrets metadata and inventory guidance | Asset and cryptography inventory |
| Key lifecycle and custody | SC-12, SC-13, SC-28 | PR.DS, PR.PS | Key and secrets lifecycle guidance | PCI key management and cryptography |
| Service account governance | AC-2, AC-6, IA-5 | PR.AA | Secrets and CI/CD identity guidance | Access control and identity governance |
| SSH and system identity | IA-5, AC-6, AU-2 | PR.AA, DE.CM | Secrets minimization and strong trust patterns | Secure operations |
| Audit logging | AU-2, AU-3, AU-6, AU-9, AU-11, AU-12 | DE.CM, RS.AN | Logging and auditing guidance | Audit trail requirements |
| Monitoring and incident response | SI-4, IR-4, IR-6 | DE.AE, RS.MI, RC.RP | Abuse detection and break-glass guidance | Monitoring and incident response |
| Escrow and recovery | CP-9, MP-6, SC-12 | RC.RP | Break-glass and archival guidance | Business continuity and retention |
| CI/CD handling | SA-15, CM-3, AU-2 | PR.PS, DE.CM | CI/CD and runtime retrieval guidance | Secure development and change control |

### 18.2 Hong Kong-specific control points

If the organization is in scope of Hong Kong critical-infrastructure regulation, the following items should be explicitly referenced in the local version of this standard:

| Hong Kong CoP requirement area | Recommended explicit reference in this framework |
|---|---|
| Security management plan endorsement | State that the plan requires Board, delegated committee, or senior-management endorsement and review at least every two years. |
| Material changes | State that platform migrations, virtualization, major database upgrades, infrastructure changes, and new external dependencies trigger material-change assessment and update obligations. |
| Risk assessments and audits | State that formal risk assessments and security audits must be planned, documented, and evidenced for covered systems. |
| Access reviews | State that user privileges and data access rights are reviewed at least annually, with approval records retained. |
| Privileged access | State that privileged accounts are separate from regular accounts or delivered through just-in-time elevation, and only authorized devices may be used where required. |
| Cryptographic keys | State explicitly that key lifecycle management must include generation, storage, archive, retrieval, distribution, retirement, and destruction, and that keys are stored separately from encrypted data. |
| Logs | State explicitly that logs must include access to critical files including password and key files, use of privileged rights, access-right changes, and audit-policy modifications, with a minimum six-month retention period. |
| Documentation and cross-reference | State that the local control set maintains cross-references between this framework and the Hong Kong CoP, including any alternative controls with equivalent outcomes. |

### 18.3 Evidence expectations

| Control area | Example evidence |
|---|---|
| Inventory completeness | Export of active secrets with owners, locations, and review dates |
| Rotation compliance | Rotation job results, exception records, test output |
| Access governance | Approval records, role definitions, quarterly reviews |
| Logging | SIEM queries, retention settings, immutable archive configuration |
| Monitoring | Detection rule catalogue and incident tickets |
| Recovery | Escrow test evidence and break-glass exercise report |

---

## 19. Roles and Responsibilities

Strong separation of duties is required so that no individual can request, approve, issue, use, and recover critical secrets without oversight.

### 19.1 Role model

| Role | Responsibilities |
|---|---|
| Board / risk governance | Risk appetite, governance oversight, policy expectation |
| CISO | Policy ownership, exception approval, major incident escalation |
| Security architecture | Standards, approved patterns, cryptoperiods, design review |
| Secrets platform team | Operation of Vault, KMS integration, availability, namespaces, audit devices |
| IAM team | Service account governance, trust relationships, access reviews |
| PKI / crypto operations | CA management, certificate issuance, signing key governance |
| SOC | Monitoring, triage, investigation, incident coordination |
| Application owner | Business need, dependency mapping, consumer testing, decommissioning |
| DevOps / CI/CD team | Secure pipeline implementation and runtime identity patterns |
| Internal audit / compliance | Independent assurance and evidence review |
| Escrow custodian | Recovery share custody and ceremony participation |

### 19.2 RACI summary

| Activity | CISO | Security Architecture | Platform Team | IAM | App Owner | SOC | Audit |
|---|---|---|---|---|---|---|---|
| Define standards | A | R | C | C | C | I | I |
| Approve exceptions | A | R | C | C | C | I | I |
| Provision policies | I | C | R | C | C | I | I |
| Approve service account access | C | C | I | A/R | C | I | I |
| Rotate secrets | I | C | R | C | C | I | I |
| Monitor alerts | I | C | C | C | C | A/R | I |
| Break-glass ceremony | A | C | R | I | I | C | I |
| Quarterly reviews | C | C | C | R | A | I | I |
| Annual assurance | I | C | C | C | C | I | A/R |

### 19.3 Governance rules

- Production secrets require owner approval and expiry or review date.
- Platform operators cannot self-approve privileged access to production secrets.
- All exceptions must be time-bound and reviewed.
- Recovery ceremonies require multi-party participation and evidence capture.

---

## 20. References

### 20.1 Normative references

1. NIST SP 800-57 Part 1 Rev. 5, *Recommendation for Key Management: Part 1 - General*.
2. NIST SP 800-57 Part 2, *Recommendation for Key Management: Best Practices for Key Management Organizations*.
3. NIST SP 800-130, *A Framework for Designing Cryptographic Key Management Systems*.
4. NIST SP 800-12 Rev. 1, *An Introduction to Information Security*.
5. NIST SP 800-53 Rev. 5, *Security and Privacy Controls for Information Systems and Organizations*.
6. NIST Cybersecurity Framework 2.0.
7. OWASP *Key Management Cheat Sheet*.
8. OWASP *Secrets Management Cheat Sheet*.
9. OWASP *Logging Cheat Sheet*.
10. OWASP *CI/CD Security Cheat Sheet*.
11. PCI DSS v4.0 Requirement 3 and related cryptographic key management guidance.
12. ISO/IEC 27001:2022 aligned guidance relevant to cryptography, access control, logging, and operations.
13. FIPS 140-3 for cryptographic module validation.
14. NIST SP 800-63 Digital Identity Guidelines for identity proofing, authentication, federation, and authenticator lifecycle management.
15. Hong Kong OCCICS Code of Practice (Version 1.0, 1 January 2026) for the Protection of Critical Infrastructures (Computer Systems) Ordinance.

### 20.2 Informative implementation references

- NIST SP 800-12 Rev. 1 for foundational rationale on why cryptographic key management is essential to information security.
- Ansible documentation for CyberArk password lookup and external secret integrations, plus CyberArk documentation for Ansible-based secret retrieval.
- Huawei Cloud CSMS API references and the HuaweiCloud Ansible collection for orchestrated hybrid retrieval patterns.
- Huawei Cloud CSMS and DEW documentation for secret creation, versioning, KMS-backed encryption, and CCE Secrets Manager for DEW integration.
- CyberArk Conjur and CyberArk Secrets Provider for Kubernetes documentation for Kubernetes delivery patterns.
- Kubernetes official guidance for Secret good practices, etcd encryption, RBAC minimization, and external secret providers.
- NIST SP 800-63 Digital Identity Guidelines and NIST SP 800-63B Authentication and Lifecycle Management for distinguishing identities, authenticators, federation, and lifecycle controls from general secret and key management.
- Hong Kong OCCICS Code of Practice for local regulatory alignment, management-plan cross-referencing, audit evidence, and alternative-control documentation.
- HashiCorp Vault documentation for dynamic secrets, PKI, SSH CA, transit encryption, and audit logging.
- AWS KMS and AWS Secrets Manager documentation for cloud-native implementations.
- Azure Key Vault and Google Cloud KMS / Secret Manager implementation guidance.
- ACSC guidance on managing cryptographic keys and secrets.

### 20.3 Document maintenance

This standard must be reviewed at least annually and whenever one of the following occurs:

- Material change to cryptographic algorithms or approved key lengths.
- Adoption of new secret platforms or retirement of existing platforms.
- Major security incident involving credential, key, or trust compromise.
- Regulatory changes affecting logging, retention, or cryptographic handling.
- Introduction of new machine identity patterns, service meshes, or workload identity platforms.

### 20.4 External reference table

| ID | Reference | Link | Primary use in this standard |
|---|---|---|---|
| R1 | NIST SP 800-57 Part 1 Rev. 5 | [NIST SP 800-57 Part 1 Rev. 5](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final) | Key lifecycle, cryptoperiods, archival, destruction |
| R1A | NIST SP 800-12 Rev. 1 | [NIST SP 800-12 Rev. 1](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-12r1.pdf) | Foundational information-security rationale, cryptography, and key-management importance |
| R2 | NIST SP 800-53 Rev. 5 | [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final) | Access, audit, crypto, monitoring, incident response controls |
| R3 | NIST Cybersecurity Framework 2.0 | [NIST CSF 2.0](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf) | Govern, Identify, Protect, Detect, Respond, Recover structure |
| R4 | NIST SP 800-130 | [NIST SP 800-130 PDF](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-130.pdf) | CKMS architecture model |
| R5 | OWASP Key Management Cheat Sheet | [OWASP Key Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Key_Management_Cheat_Sheet.html) | Practical key handling controls |
| R6 | OWASP Secrets Management Cheat Sheet | [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) | Secret lifecycle, CI/CD, metadata, in-memory handling |
| R7 | OWASP Logging Cheat Sheet | [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html) | Audit logging content and protection |
| R8 | OWASP CI/CD Security Cheat Sheet | [OWASP CI/CD Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/CI_CD_Security_Cheat_Sheet.html) | Pipeline hardening and approvals |
| R9 | PCI DSS Requirement 3 guidance | [PCI DSS Requirement 3 Explained](https://pcidssguide.com/pci-dss-requirement-3/) | Cryptographic key management expectations |
| R10 | Huawei CSMS overview / DEW | [Huawei CSMS Overview](https://support.huaweicloud.com/eu/usermanual-dew/dew_01_2000.html) | Huawei platform implementation reference |
| R11 | Huawei CSMS create secret | [Huawei Create Secret API/Guide](https://support.huaweicloud.com/intl/en-us/usermanual-dew/dew_01_9993.html) | Secret creation and attributes |
| R12 | Huawei CSMS secret versions | [Huawei Managing Secret Versions](https://support.huaweicloud.com/intl/en-us/usermanual-dew/dew_01_8882.html) | Version-aware retrieval and rotation |
| R13 | Huawei CCE Secrets Manager for DEW | [Huawei CCE Secrets Manager for DEW](https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0370.html) | Kubernetes integration on Huawei |
| R14 | CyberArk Secrets Provider for Kubernetes | [CyberArk Secrets Provider for Kubernetes](https://docs.cyberark.com/conjur-open-source/latest/en/content/integrations/k8s-ocp/cjr-secrets-provider-lp.htm) | Kubernetes delivery patterns outside Huawei |
| R15 | CyberArk Conjur | [CyberArk Conjur](https://www.conjur.org) | Policy-based secret management reference |
| R16 | Ansible CyberArk lookup plugin | [Ansible CyberArk password lookup](https://docs.ansible.com/projects/ansible/latest/collections/community/general/cyberarkpassword_lookup.html) | Ansible-based CyberArk retrieval |
| R17 | CyberArk Ansible integration | [CyberArk Ansible integration](https://docs.cyberark.com/secrets-manager-sh/13.7/en/content/integrations/ansible.html) | Ansible integration pattern |
| R18 | HuaweiCloud Ansible collection | [HuaweiCloud Ansible collection](https://github.com/huaweicloud/huaweicloud-ansible-modules) | Huawei hybrid orchestration reference |
| R19 | Kubernetes Secrets good practices | [Kubernetes Secrets good practices](https://kubernetes.io/docs/concepts/security/secrets-good-practices/) | etcd encryption, RBAC, and external provider guidance |
| R20 | CyberArk VM integration example | [Conjur secrets in VM applications](https://developer.cyberark.com/blog/technical-deep-dive-using-conjur-secrets-in-vm-deployed-ansible-tower-applications/) | VM retrieval model reference |
| R21 | NIST SP 800-63 Digital Identity Guidelines | [NIST SP 800-63 Digital Identity Guidelines](https://pages.nist.gov/800-63-3/) | Identity proofing, authentication, federation, and authenticator lifecycle distinctions |
| R21A | NIST cryptographic key glossary | [NIST cryptographic key glossary](https://csrc.nist.gov/glossary/term/cryptographic_key) | Formal key definition used by this framework |
| R22 | Hong Kong OCCICS Code of Practice (v1.0, 1 Jan 2026) | [OCCICS CoP PDF](https://www.occics.gov.hk/filemanager/en/content_19/CoP_en_v1.0.pdf) | Hong Kong regulatory overlay for governance, documentation, access control, cryptography, logging, audits, and incident handling |

### 20.5 Cross-reference validation

All detected internal markdown anchor links in the table of contents and section references were validated against document headings using GitHub-style anchor generation.

The document also contains 24 clickable external HTTPS links in the reference sections and appendix reference table.

# Appendix A. Huawei, CyberArk, and Kubernetes Implementation Profiles

This appendix extends the framework with a split-platform implementation model for Huawei-hosted infrastructure, non-Huawei infrastructure, and Kubernetes runtime delivery patterns.

## A.1 Appendix purpose

This appendix defines how the enterprise should implement secrets management when:

- workloads run on Huawei-hosted cloud or Huawei-aligned private cloud platforms,
- workloads run outside Huawei infrastructure and require enterprise vaulting,
- applications run on Kubernetes and need externalized secret delivery.

The appendix does not replace the core lifecycle, audit, inventory, rotation, escrow, and incident-response requirements in Chapters 1 to 20. It applies those requirements to specific deployment patterns.

## A.2 Platform split model

| Environment / operating zone | Primary secret platform | Typical usage | Delivery pattern |
|---|---|---|---|
| Huawei-hosted cloud or Huawei-aligned private cloud | Huawei Cloud Secret Management Service (CSMS) under DEW | Application secrets, DB credentials, service secrets, selected API keys | CCE Secrets Manager for DEW via SecretProviderClass and ServiceAccount scoping |
| Huawei CCE Kubernetes clusters | Huawei CSMS + CCE Secrets Manager for DEW | Pod-mounted secrets, file delivery, selected synchronized Kubernetes Secrets | CSI-based mount first, Kubernetes Secret sync only when required |
| Non-Huawei cloud, hybrid, or on-prem zones | CyberArk-integrated retrieval, including Ansible lookup-based retrieval from CyberArk vault services where Conjur is not available | Service secrets, machine credentials, CI/CD secrets, VM bootstrap secrets, non-Huawei runtime secrets | Ansible lookup and injection, sidecar, init container, shared file volume, or controlled Kubernetes Secret sync |
| Enterprise crypto custody | HSM / KMS / PKI platform from the core framework | KEKs, CA keys, signing keys, high-assurance private keys | HSM-backed operations only |

## A.3 Huawei implementation profile

Huawei documentation describes Cloud Secret Management Service as part of DEW for centrally managing secrets, storing encrypted secret versions, supporting secret creation and versioning, and using either a default `csms/default` key or a user-defined symmetric KMS key for encryption. [web:82][web:83][web:93]

Huawei also documents the CCE Secrets Manager for DEW add-on as integrating CCE with DEW so secrets managed outside the cluster can be mounted to service pods, with scheduled rotation support when `objectVersion` is set to `latest` and real-time detection of `SecretProviderClass` changes. [page:2]

### A.3.1 Huawei design principles

- Use Huawei CSMS as the system of record for secrets primarily consumed by Huawei-hosted workloads. [web:93][page:2]
- Use customer-managed KMS keys for Tier 1 and Tier 2 Huawei-hosted secrets instead of relying only on the default CSMS key. [web:82]
- Tag every secret with owner, environment, application, sensitivity tier, and rotation policy so the inventory remains complete and aligned to Chapter 8 requirements. [web:82]
- Prefer runtime secret mounting into pods over hard-coded values, plaintext manifests, or manual distribution. [page:2]

### A.3.2 Huawei reference architecture

```mermaid
flowchart TB
    App[Application Pod] --> SA[CCE ServiceAccount]
    SA --> SPC[SecretProviderClass]
    SPC --> CCE[CCE Secrets Manager for DEW]
    CCE --> CSMS[Huawei CSMS]
    CSMS --> KMS[Huawei KMS key]
    CCE --> VOL[Mounted secret volume]
    CCE --> K8S[Optional synced Kubernetes Secret]
    VOL --> App
    K8S --> App
    CCE --> LOGS[Addon and pod status logs]
```

### A.3.3 Huawei control requirements

| Control area | Huawei-specific requirement |
|---|---|
| Authorization | Restrict each workload through `ServiceAccount` annotations so only explicitly named secrets can be mounted |
| Versioning | Use explicit secret versions for deterministic deployments; use `latest` only where controlled refresh is intended |
| Rotation | Configure `rotation_poll_interval` to match application tolerance, restart behavior, and risk level |
| Delivery model | Use file or volume delivery as the default; sync to Kubernetes Secrets only when compatibility demands it |
| Monitoring | Collect dew-provider, CSI driver, and `spcPodStatus` telemetry for audit and troubleshooting |
| KMS usage | Use customer-managed KMS keys for higher sensitivity tiers |
| Failure mode | Pod startup should fail or block if undeclared secrets are requested |

### A.3.4 Huawei Kubernetes example: mount CSMS secrets as files

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: finance-app-sa
  annotations:
    cce.io/dew-resource: '["payments-db-secret","hsm-client-secret"]'
---
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: finance-app-spc
spec:
  provider: cce
  parameters:
    objects: |
      - objectName: "payments-db-secret"
        objectType: "csms"
        objectVersion: "latest"
      - objectName: "hsm-client-secret"
        objectType: "csms"
        objectVersion: "v1"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: finance-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: finance-app
  template:
    metadata:
      labels:
        app: finance-app
    spec:
      serviceAccountName: finance-app-sa
      volumes:
        - name: secrets-store-inline
          csi:
            driver: secrets-store.csi.k8s.io
            readOnly: true
            volumeAttributes:
              secretProviderClass: "finance-app-spc"
      containers:
        - name: finance-app
          image: finance-app:latest
          volumeMounts:
            - name: secrets-store-inline
              mountPath: /mnt/secrets
              readOnly: true
```

### A.3.5 Huawei Kubernetes example: synchronize selected values to a Kubernetes Secret

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: finance-app-sync
spec:
  provider: cce
  parameters:
    objects: |
      - objectName: "payments-db-secret"
        objectType: "csms"
        objectVersion: "latest"
        jmesPath:
          - path: username
            objectAlias: dbusername
          - path: password
            objectAlias: dbpassword
  secretObjects:
    - secretName: finance-db-credentials
      type: Opaque
      data:
        - objectName: dbusername
          key: username
        - objectName: dbpassword
          key: password
```

### A.3.6 Huawei operating notes

- Treat synchronized Kubernetes Secrets as secondary copies rather than the enterprise source of truth.
- For higher-sensitivity workloads, prefer file-based or volume-based delivery over environment-variable exposure.
- Record the CSMS secret name, version, KMS key, consuming namespace, and service account in the enterprise inventory.

## A.4 CyberArk implementation profile

CyberArk documents the Secrets Provider for Kubernetes as able to populate either Kubernetes Secrets or shared volume files with secrets stored and managed in Conjur. CyberArk also documents deployment options including sidecar or init-container mode inside an application pod and a namespace-level service model. [page:3][web:94]

### A.4.1 CyberArk design principles

- Use CyberArk Conjur as the primary platform for secrets outside Huawei-hosted infrastructure. [page:3][web:94]
- Scope Conjur policies by application, namespace, environment, and host or workload identity. [web:94]
- Prefer explicit secret mappings rather than broad wildcard retrieval, because CyberArk notes that fetch-all patterns can expose more secrets than necessary if a workload is compromised. [page:3]
- Prefer sidecar or init-container delivery with shared files when applications can consume files directly. [page:3]
- Use Kubernetes Secret synchronization only when application compatibility requires native Secret objects. [page:3]

### A.4.2 CyberArk reference architecture

```mermaid
flowchart TB
    App[Application Pod] --> CSP[CyberArk Secrets Provider]
    CSP --> Conjur[CyberArk Conjur]
    Conjur --> Policy[RBAC and policy checks]
    Policy --> CSP
    CSP --> Files[Shared secret files]
    CSP --> Sync[Optional synced Kubernetes Secret]
    Files --> App
    Sync --> App
```

### A.4.3 CyberArk delivery options

| Delivery option | Use case | Notes |
|---|---|---|
| Sidecar or init container | Preferred for app-specific delivery | Secrets exposed as shared local files |
| Namespace service model | Namespace-wide service pattern | Supports multiple apps in one namespace |
| Kubernetes Secret sync | Compatibility for apps expecting native Secrets | Adds another in-cluster copy |
| Push-to-file | Minimal application change pattern | Good for file-based secret consumption |

### A.4.4 CyberArk Kubernetes example: sidecar or init-container pattern

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: conjur-finance-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: conjur-finance-app
  template:
    metadata:
      labels:
        app: conjur-finance-app
    spec:
      serviceAccountName: conjur-finance-app
      volumes:
        - name: conjur-secrets
          emptyDir:
            medium: Memory
      initContainers:
        - name: cyberark-secrets-provider
          image: cyberark/secrets-provider-for-k8s:latest
          volumeMounts:
            - name: conjur-secrets
              mountPath: /conjur/secrets
      containers:
        - name: app
          image: finance-app:latest
          volumeMounts:
            - name: conjur-secrets
              mountPath: /mnt/secrets
              readOnly: true
```

### A.4.5 CyberArk control requirements

| Control area | Requirement |
|---|---|
| Secret scope | Limit each Conjur host or workload to only required variables |
| Mapping model | Prefer individually mapped secrets rather than fetch-all |
| Namespace isolation | Separate production, non-production, and shared-services namespaces |
| Audit | Ingest Conjur access and policy events into the enterprise SIEM |
| Rotation | Align Conjur rotation workflows with downstream reload capability |
| Duplication control | Minimize long-lived duplication into native Kubernetes Secret objects |

## A.5 Kubernetes-specific controls

Kubernetes documents that Secret objects hold sensitive data such as passwords, OAuth tokens, and SSH keys, and that secret data is stored unencrypted by default in etcd unless encryption at rest is configured. Kubernetes also recommends considering external secret providers through the Secrets Store CSI Driver when confidential data should be kept outside the cluster. [page:4]

### A.5.1 Mandatory Kubernetes controls

| Control | Requirement |
|---|---|
| etcd encryption | Enable encryption at rest for Kubernetes Secret data |
| RBAC | Restrict `get`, `watch`, and especially `list` access on Secret objects to the minimum required principals |
| Namespace isolation | Use namespaces to isolate secret access boundaries |
| Pod design | Mount secrets only into containers that need them |
| Manifest hygiene | Do not commit base64-encoded Secret manifests with real secret values to source control |
| External secret providers | Prefer CSI-based or provider-based external secret delivery for higher-value secrets |
| App hygiene | Applications must not log, forward, or expose values after reading them |
| Pod creation rights | Treat users who can create pods that mount secrets as effectively able to access those secrets |

### A.5.2 Kubernetes decision table

| Pattern | Recommended use | Risk note |
|---|---|---|
| Native Kubernetes Secret only | Lower-sensitivity or compatibility edge cases | Requires etcd encryption and strong RBAC |
| External secret via CSI volume mount | Preferred for high-value runtime secrets | Keeps source of truth outside cluster |
| Synced Kubernetes Secret from external store | Transitional or compatibility use | Duplicates secret material into cluster state |
| Environment variable from Secret | Only when app cannot consume files or external retrieval | Higher exposure through process and debugging surfaces |

### A.5.3 Enterprise Kubernetes pattern

```mermaid
flowchart LR
    A[Enterprise source of truth] --> B{Deployment zone}
    B -->|Huawei hosted zone| C[Huawei CSMS]
    B -->|Non-Huawei zone| D[CyberArk Conjur]
    C --> E[CCE Secrets Manager for DEW]
    D --> F[CyberArk Secrets Provider]
    E --> G[Pod volume mount or synced Kubernetes Secret]
    F --> H[Shared file mount or synced Kubernetes Secret]
    G --> I[Application runtime]
    H --> I
```

## A.6 Standards statements to add

1. Huawei-hosted workloads must use Huawei CSMS as the default secrets platform unless an approved exception exists. [web:93][page:2]
2. Non-Huawei workloads should use CyberArk-integrated retrieval as the default enterprise pattern for application and machine secrets outside Huawei infrastructure, with Ansible-based retrieval used where Conjur is not available. [page:3][web:94][web:111][web:112]
3. Kubernetes platforms must use external secret delivery for Tier 1 and Tier 2 runtime secrets wherever feasible. [page:4][page:2][page:3]
4. Any synchronized Kubernetes Secret derived from Huawei CSMS or CyberArk Conjur must be treated as a secondary copy and governed by tighter TTL, RBAC, and audit controls. [page:2][page:3][page:4]
5. The enterprise inventory must record the platform of record, delivery mechanism, consuming namespace or workload, service account, and refresh model for each runtime-delivered secret. [web:82][page:2][page:3]



## A.7 Virtual machine patterns

These patterns extend the appendix with virtual-machine examples for Huawei VPC environments and non-Huawei zones using CyberArk.

### A.7.1 Huawei VPC pattern: ECS VM to CSMS, database, and CCE in the same VPC

Huawei documents that CCE Secrets Manager for DEW can access DEW through a VPC endpoint, and Huawei VPC guidance notes that ECS communication depends on VPC routing and security-group rules. [web:84][web:97] In a Huawei-hosted design, an ECS virtual machine, a managed database, and a CCE cluster can operate in the same VPC with security groups and routing constrained to only the required east-west flows. [web:97][web:84]

```mermaid
flowchart TB
    subgraph VPC[Huawei VPC]
        VM[ECS virtual machine]
        DB[Managed database]
        CCE[CCE Kubernetes cluster]
        VPCE[VPC Endpoint for DEW]
        CSMS[Huawei CSMS]
    end
    VM -->|TLS request for secret| VPCE
    VPCE --> CSMS
    VM -->|DB auth using retrieved credential| DB
    VM -->|mTLS or API call| CCE
    CCE -->|Secret retrieval through add-on| VPCE
    VPCE --> CSMS
```

#### Example use case

- An ECS-based payment adapter retrieves a database password or API credential from Huawei CSMS over a private VPC endpoint, then opens a TLS-protected session to a database in the same VPC. [web:84][web:83]
- The same ECS VM can call services hosted in a CCE cluster in that VPC, while CCE workloads retrieve their own runtime secrets through the CCE Secrets Manager for DEW integration. [web:84]
- Security groups should allow only the ECS subnet to reach the database port, the VPC endpoint, and approved CCE service addresses. [web:97]

#### Control requirements

| Control | Requirement |
|---|---|
| Network path | Use private VPC routing and a VPC endpoint for DEW so the VM does not depend on public internet access for secret retrieval. [web:84] |
| Identity | Bind the ECS VM to a managed machine identity or tightly scoped credential used only for CSMS retrieval. [web:83][web:84] |
| Segmentation | Restrict security-group rules so the VM can reach only DEW endpoint addresses, required database ports, and approved CCE services. [web:97] |
| Rotation | Application should re-read or refresh secrets after version changes rather than caching indefinitely. [web:81][web:84] |
| Inventory | Record the ECS instance, subnet, VPC, destination database, and related CCE services as source/destination metadata in the master inventory. [web:97][web:84] |

### A.7.2 Huawei VPC pattern: ECS VM retrieves keys or secrets from CSMS

Huawei documents that CSMS supports secret creation and secret version management, which makes it suitable for VM-based applications that must retrieve current versions of secrets at runtime. [web:82][web:81]

```mermaid
sequenceDiagram
    participant VM as ECS VM
    participant VPCE as VPC Endpoint
    participant CSMS as Huawei CSMS
    participant APP as VM Application
    participant DB as Database/API
    VM->>VPCE: TLS request for secret metadata/value
    VPCE->>CSMS: Private API call
    CSMS-->>VPCE: Current secret version
    VPCE-->>VM: Secret response
    VM->>APP: Deliver in memory or protected file
    APP->>DB: Authenticate with retrieved secret
```

#### Example implementation options

| VM secret use case | Recommended pattern |
|---|---|
| VM needs DB password | Retrieve secret at startup from CSMS, hold in protected memory, and refresh on schedule or on connection failure. [web:82][web:81] |
| VM needs API key for internal service | Read the current CSMS secret version over the VPC endpoint and inject into the application process without writing plaintext to disk. [web:84][web:82] |
| VM needs certificate or private key material | Store only lower-sensitivity exported material in CSMS; keep high-assurance private keys in HSM-backed systems per the main framework. [web:82] |
| VM batch job needs short-lived credential | Use the VM to fetch a current secret immediately before job execution, then zeroize and terminate after the job finishes. [web:81] |

### A.7.3 Non-Huawei pattern: VM retrieves secrets from CyberArk

CyberArk describes Conjur as centrally managing secrets across tools, apps, containers, and clouds with policy-based RBAC and audit trails, and CyberArk's VM integration example shows remote nodes retrieving their own secrets using their own identities rather than embedding them directly in automation tools. [web:94][web:103]

```mermaid
flowchart LR
    VM[Virtual machine] --> AUTH[Host identity authentication]
    AUTH --> CONJUR[CyberArk Conjur]
    CONJUR --> POLICY[Policy and RBAC check]
    POLICY --> VM
    VM --> APP[Application or job]
    APP --> DEST[Database, API, SSH target, or broker]
    CONJUR --> SIEM[Audit trail]
```

#### Example use cases

- A non-Huawei VM running an application retrieves a database password or API token from Conjur at startup using the VM's own identity and then connects to the downstream service. [web:94][web:103]
- An automation VM retrieves SSH private-key material or an external secret through Conjur only for the duration of a runbook, aligning with CyberArk's example of secrets being made available only to a subset of nodes. [web:103]
- A hybrid enterprise can keep Huawei-zone workloads on CSMS and use Conjur as the cross-platform vault for non-Huawei VMs, CI/CD hosts, and external operational zones. [web:94]

### A.7.4 Hybrid VM and Kubernetes pattern

A common enterprise pattern is to place ECS virtual machines, managed databases, and CCE clusters in one Huawei VPC while also operating non-Huawei VMs that use CyberArk. Huawei documents VPC security-group controls for ECS connectivity and CCE integration with DEW, while CyberArk documents centralized policy-based secret distribution across applications and clouds. [web:97][web:84][web:94]

```mermaid
flowchart TB
    subgraph HuaweiZone[Huawei zone]
        ECS[ECS VM]
        HCSMS[Huawei CSMS]
        HCDB[Database]
        HCCE[CCE cluster]
        HVPCE[VPC Endpoint]
        ECS --> HVPCE --> HCSMS
        ECS --> HCDB
        ECS --> HCCE
        HCCE --> HVPCE
    end
    subgraph ExternalZone[Non-Huawei zone]
        VM2[Hybrid VM]
        CONJUR[CyberArk Conjur]
        EXTDB[External DB or API]
        VM2 --> CONJUR --> VM2
        VM2 --> EXTDB
    end
```

### A.7.5 VM operational controls

| Control | Huawei VM using CSMS | VM using CyberArk |
|---|---|---|
| Secret retrieval path | Private VPC endpoint to DEW/CSMS. [web:84] | Mutual-authenticated or policy-controlled Conjur connection. [web:94] |
| Identity scope | ECS identity or app credential limited to required secrets. [web:83] | Host identity or machine identity with explicit RBAC. [web:94][web:103] |
| Local storage | Avoid plaintext disk storage; use memory or protected temporary files only when necessary. [web:81][web:103] | Avoid plaintext disk storage; prefer ephemeral retrieval. [web:103] |
| Monitoring | Log CSMS access, VPC endpoint use, DB auth, and downstream calls. [web:84][web:97] | Log Conjur retrievals, policy decisions, and downstream privileged operations. [web:94] |
| Rotation response | Refresh on secret-version change or app reconnect logic. [web:81] | Refresh on Conjur rotation or job start. [web:103][web:94] |
| Segmentation | Security groups and route tables restricted to necessary flows. [web:97] | Host firewall and network ACLs restricted to Conjur and approved destinations. [web:94] |



## A.7.6 Alternative when Conjur is not available: Ansible with CyberArk

Yes, Ansible can be used to retrieve secrets from CyberArk when Conjur is not available. Ansible documents the `community.general.cyberarkpassword` lookup plugin for retrieving credentials from CyberArk, and CyberArk documents Ansible integrations that enable managed nodes to retrieve secrets from CyberArk Secrets Manager using variables defined on the Ansible control node. [web:111][web:112][web:114][web:118]

This makes Ansible a viable **integration layer** for virtual machines that need just-in-time secret injection, especially during provisioning, configuration, or controlled runtime refresh. The tradeoff is that Ansible becomes part of the secret-delivery path, so the control node, logs, and temporary artifacts must be tightly protected and `no_log: true` must be enforced for secret-bearing tasks. [web:111][web:112]

```mermaid
flowchart LR
    AC[Ansible control node] --> CYB[CyberArk vault or CCP / Secrets Manager]
    CYB --> AC
    AC --> VM[Target virtual machine]
    VM --> APP[Application or service]
    CYB --> AUDIT[CyberArk audit trail]
    AC --> ALOG[Ansible job logs]
```

### Example: Ansible retrieves a password from CyberArk and injects it into a VM

```yaml
- name: Retrieve CyberArk secret and inject into VM
  hosts: finance_vms
  gather_facts: false
  vars:
    db_secret: >-
      {{ lookup('community.general.cyberarkpassword',
                { 'appid': 'app_ansible',
                  'query': 'safe=Finance;folder=root;object=PaymentsDB',
                  'output': 'Password,PassProps.UserName,PassProps.Address' }) }}
  tasks:
    - name: Create runtime secret directory in memory-backed path
      ansible.builtin.file:
        path: /run/finance-secrets
        state: directory
        owner: root
        group: root
        mode: '0700'

    - name: Write secret to protected runtime file
      ansible.builtin.copy:
        dest: /run/finance-secrets/db.password
        content: "{{ db_secret.Password }}"
        owner: root
        group: app
        mode: '0640'
      no_log: true

    - name: Restart application after secret refresh
      ansible.builtin.service:
        name: finance-app
        state: restarted
```

### Control requirements for Ansible-to-CyberArk injection

| Control | Requirement |
|---|---|
| Control node trust | Treat the Ansible control node as a privileged secret-handling component and harden it to the same standard as the secret platform |
| Task logging | Use `no_log: true` for every task that retrieves, stores, templates, or echoes secret values |
| Delivery target | Prefer memory-backed paths such as `/run` or tmpfs-backed locations over persistent disk |
| Least privilege | Limit the CyberArk application ID, safe, and query scope to only the secrets required by that playbook |
| Lifetime | Inject secrets only for the job or service lifecycle required and rotate or remove them after use |
| Audit | Correlate CyberArk retrieval logs with Ansible job IDs, inventory hostnames, and downstream service restarts |

## A.7.7 Hybrid pattern: Ansible retrieves from Huawei CSMS and injects into VMs

Huawei documents CSMS secret creation and versioning APIs, and the HuaweiCloud Ansible collection provides modules for interacting with Huawei Cloud services. In hybrid environments, Ansible can act as the control-plane orchestrator that authenticates to Huawei APIs, retrieves the required secret version from CSMS, and injects it into a VM running either inside a Huawei VPC or outside Huawei infrastructure. [web:83][web:121][web:82]

This pattern is useful when a virtual machine cannot directly call CSMS at runtime, when outbound internet or endpoint routing is restricted, or when a build or configuration pipeline must place a bootstrap credential onto a VM before the application starts. Because Ansible becomes the transfer point, it must follow the same protection rules as the CyberArk pattern. [web:83][web:121]

```mermaid
flowchart LR
    AC[Ansible control node] --> IAM[Huawei IAM / auth]
    IAM --> CSMS[Huawei CSMS]
    CSMS --> AC
    AC --> VMH[VM in Huawei VPC]
    AC --> VMX[VM outside Huawei in hybrid zone]
    VMH --> APPH[Application]
    VMX --> APPX[Application]
```

### Example: Ansible retrieves a Huawei secret and writes it to a VM

```yaml
- name: Retrieve Huawei CSMS secret and inject into VM
  hosts: finance_vms
  gather_facts: false
  vars:
    huawei_region: cn-north-4
    csms_endpoint: "https://csms.{{ huawei_region }}.myhuaweicloud.com/v1/{project_id}/secrets/{secret_id}"
    iam_token: "{{ lookup('env', 'HUAWEI_IAM_TOKEN') }}"
  tasks:
    - name: Read secret metadata or current version from Huawei CSMS API
      ansible.builtin.uri:
        url: "{{ csms_endpoint }}"
        method: GET
        headers:
          X-Auth-Token: "{{ iam_token }}"
        return_content: true
      register: huawei_secret
      delegate_to: localhost
      no_log: true

    - name: Create protected runtime directory
      ansible.builtin.file:
        path: /run/finance-secrets
        state: directory
        owner: root
        group: root
        mode: '0700'

    - name: Inject secret into runtime file on VM
      ansible.builtin.copy:
        dest: /run/finance-secrets/api.key
        content: "{{ huawei_secret.json.secret_text | default('REPLACE_WITH_FIELD_MAPPING') }}"
        owner: root
        group: app
        mode: '0640'
      no_log: true
```

### Example: Hybrid flow where Huawei CSMS provides the secret and Ansible injects it to a non-Huawei VM

```mermaid
flowchart TB
    HCSMS[Huawei CSMS] --> ACTRL[Ansible control node]
    ACTRL --> HVM[Huawei VPC VM]
    ACTRL --> XVM[External or hybrid VM]
    HVM --> HAPP[Huawei-side application]
    XVM --> XAPP[External application]
```

### Control requirements for Ansible-to-Huawei injection

| Control | Requirement |
|---|---|
| API auth | Use short-lived Huawei IAM tokens or approved API credentials on the control node, not on the target VM |
| Secret field mapping | Map only the fields required by the VM workload and avoid transferring full multi-value payloads unnecessarily |
| Network segmentation | Permit control-node access only to Huawei API endpoints and the target VMs required for the playbook |
| Storage | Use protected runtime files or memory-backed paths on the VM; do not leave bootstrap secrets in cloud-init history or shell history |
| Version awareness | Record the injected secret version so a later rotation can reconcile which hosts received which version |
| Audit | Log the Ansible job ID, target host, secret identifier, and success or failure status without logging secret content |

## A.8 Appendix references

| Ref ID | Reference | Clickable link | Primary appendix use |
|---|---|---|---|
| R10 | Huawei CSMS overview / DEW | [Huawei CSMS Overview](https://support.huaweicloud.com/eu/usermanual-dew/dew_01_2000.html) | Huawei platform baseline |
| R11 | Huawei CSMS create secret | [Huawei Create Secret](https://support.huaweicloud.com/intl/en-us/usermanual-dew/dew_01_9993.html) | Secret creation and secret attributes |
| R12 | Huawei CSMS secret versions | [Huawei Managing Secret Versions](https://support.huaweicloud.com/intl/en-us/usermanual-dew/dew_01_8882.html) | Version-aware retrieval and rotation |
| R13 | Huawei CCE Secrets Manager for DEW | [Huawei CCE Secrets Manager for DEW](https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0370.html) | Kubernetes integration and VPC-aligned runtime delivery |
| R14 | CyberArk Secrets Provider for Kubernetes | [CyberArk Secrets Provider for Kubernetes](https://docs.cyberark.com/conjur-open-source/latest/en/content/integrations/k8s-ocp/cjr-secrets-provider-lp.htm) | Kubernetes integration outside Huawei |
| R15 | CyberArk Conjur | [CyberArk Conjur](https://www.conjur.org) | CyberArk policy-based secret management reference |
| R16 | Ansible CyberArk lookup plugin | [Ansible CyberArk password lookup](https://docs.ansible.com/projects/ansible/latest/collections/community/general/cyberarkpassword_lookup.html) | Ansible lookup from CyberArk |
| R17 | CyberArk Ansible integration | [CyberArk Ansible integration](https://docs.cyberark.com/secrets-manager-sh/13.7/en/content/integrations/ansible.html) | Ansible-based CyberArk retrieval patterns |
| R18 | HuaweiCloud Ansible collection | [HuaweiCloud Ansible collection](https://github.com/huaweicloud/huaweicloud-ansible-modules) | Hybrid orchestration with Huawei APIs |
| R19 | Kubernetes Secrets good practices | [Kubernetes Secrets good practices](https://kubernetes.io/docs/concepts/security/secrets-good-practices/) | etcd encryption, RBAC, namespace isolation |
| R20 | CyberArk VM integration example | [Conjur secrets in VM applications](https://developer.cyberark.com/blog/technical-deep-dive-using-conjur-secrets-in-vm-deployed-ansible-tower-applications/) | VM retrieval model reference |
| R21 | Hong Kong OCCICS Code of Practice | [OCCICS CoP PDF](https://www.occics.gov.hk/filemanager/en/content_19/CoP_en_v1.0.pdf) | Hong Kong regulatory overlay for local deployment |
