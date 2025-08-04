# NIST SP 800-57 Part 1 Rev. 5 - Key Requirements for Secrets and Key Management Systems

## Key Security Services Required
1. **Confidentiality** - Information not disclosed to unauthorized parties
2. **Data Integrity** - Data has not been modified in unauthorized manner
3. **Authentication** - Three types:
   - Identity authentication
   - Integrity authentication  
   - Source authentication
4. **Authorization** - Official sanction/permission to perform functions
5. **Non-repudiation** - Commitment and accountability for digital signatures

## Key Management Functions (Section 8)
1. **Pre-operational Phase**
   - Entity Registration Function
   - System Initialization Function
   - Initialization Function
   - Keying-Material Installation Function
   - Key Establishment Function

2. **Operational Phase**
   - Key Usage Function
   - Key Derivation Function
   - Key Update Function
   - Key Recovery Function
   - Key Backup Function
   - Key Archive Function

3. **Post-operational Phase**
   - Key Destruction Function
   - Key Revocation Function

## Key States and Lifecycle (Section 7)
1. **Pre-activation State** - Key generated but not authorized for use
2. **Active State** - Key authorized for use
3. **Suspended State** - Key use temporarily suspended
4. **Deactivated State** - Key no longer authorized for use
5. **Compromised State** - Key suspected or known to be compromised
6. **Destroyed State** - Key destroyed and no longer available

## Protection Requirements (Section 6)
### For Keys in Transit:
- Availability
- Integrity
- Confidentiality
- Association with usage/application
- Association with other entities
- Association with other related key information

### For Keys in Storage:
- Availability
- Integrity
- Confidentiality
- Association with usage/application
- Association with other entities
- Association with other related key information

## Key Types and Information (Section 5.1)
1. **Cryptographic Keys**
   - Symmetric keys (secret keys)
   - Asymmetric keys (public/private key pairs)
   - Key-wrapping keys
   - Key-encryption keys

2. **Other Related Information**
   - Initialization vectors
   - Passwords/passphrases
   - Domain parameters
   - Certificates
   - Metadata

## Cryptoperiods and Key Rotation (Section 5.3)
- Keys must have defined cryptoperiods (lifetime for use)
- Factors affecting cryptoperiods:
  - Strength of cryptographic algorithm
  - Volume of data protected
  - Sensitivity of data
  - Environment factors
  - Cost of key replacement

## Key Management Issues Addressed:
- Key usage restrictions
- Cryptoperiod length determination
- Domain-parameter validation
- Public-key validation
- Key-inventory management
- Accountability and audit requirements
- Survivability requirements
- Algorithm and key size selection guidance

## Security Strength Requirements
- Minimum security strength levels: 80, 112, 128, 192, 256 bits
- Note: 80-bit security strength no longer considered sufficiently secure
- Algorithm suites must have comparable security strengths

## Compromise Handling (Section 5.5)
- Procedures for key compromise detection
- Protective measures against compromise
- Recovery procedures
- Notification requirements

