# ADR-012: BYOK Credential Encryption and Key Management

- **Status:** Proposed — not owner-approved
- **Date:** 2026-11-26
- **Deciders:** Owner (final approval); ChatGPT (architecture review)
- **Related issues:** WG-002, WG-003, WG-020, WG-023

## Context

Customers may provide model-provider API keys. These credentials can incur financial charges and grant access to customer accounts. They must remain confidential, tenant-isolated, revocable and usable only by authorised server-side provider calls.

## Options considered

| Option | Advantages | Disadvantages | Cost/free-tier implications | Lock-in and exit considerations |
|---|---|---|---|---|
| Application-layer envelope encryption with managed KMS | Separates encrypted data from key-encryption keys; supports key rotation and central access audit | Requires KMS integration, IAM design and careful recovery procedures | KMS usage may not have a free production allowance; costs must be checked before adoption | Moderate provider coupling; use a crypto/key-provider abstraction and documented export/recovery approach |
| Application-layer encryption with a separately protected master key | Can be implemented without a dedicated managed KMS at very low scale | Key storage, rotation, access control and recovery are operator responsibilities; a compromised runtime may expose the key | Low direct service cost; high operational responsibility | Low vendor lock-in but high key-management risk if poorly operated |
| Database/provider encryption at rest only | Simple and often included by hosting provider | Does not protect against application/database privilege compromise or prevent plaintext access during use | Often included in hosting plan | Strong dependence on provider security model; insufficient alone for BYOK |

## Proposed decision

Use **application-layer envelope encryption** for stored customer provider credentials. A tenant-scoped data-encryption key encrypts each credential; a separately protected key-encryption key protects the data key. Prefer a managed KMS or dedicated secret-management service for production, subject to a cost and portability review.

For development, use synthetic credentials only. Do not store real customer API keys until the encryption design, key recovery, rotation, revocation and audit controls have been implemented and reviewed.

## Required controls

- Plaintext keys are write-only at entry and are never returned to the browser after storage.
- Decryption occurs only server-side, immediately before a permitted provider request.
- Credentials are scoped to the owning workspace and provider.
- Support replacement, rotation, revocation and deletion.
- Separate encryption keys from the data they encrypt.
- Record who/what accessed a credential, when and for what purpose, without logging the value.
- Redact keys from exceptions, traces, analytics, support exports and backups.
- Define emergency key recovery and compromise response.
- Rotate key-encryption keys without requiring customers to re-enter every provider key where technically feasible.

## Security, privacy and data residency

- Use authenticated encryption with a vetted cryptographic library; do not invent cryptographic primitives.
- Protect key-encryption keys with least-privilege access and audit logging.
- Enforce tenant checks before decryption, not merely before displaying a credential record.
- TLS is required for credential submission and provider calls.
- Define retention and deletion for encrypted values, backups and key metadata.
- Review provider terms and region/data-residency requirements before production.

## Consequences

### Positive
- Reduces impact of a database-only compromise.
- Provides a defined lifecycle for customer secrets.
- Keeps credential access controlled by the application backend.

### Negative / trade-offs
- Key management and recovery add operational complexity.
- KMS use may create recurring cost and provider coupling.
- Losing the key-encryption key can make stored credentials unrecoverable.

### Operational implications
- No production BYOK until WG-020 passes security review.
- Document key hierarchy, rotation, backup, recovery and incident procedures.
- Keep secret references and encryption metadata portable.

## Validation plan

1. Threat-model database compromise, runtime compromise, key-store outage and accidental log leakage.
2. Select a vetted cryptographic library and managed key service only after owner approval.
3. Test rotation, revocation, backup/restore and deletion using synthetic values.
4. Run automated secret-redaction tests against logs and error paths.
5. Obtain an independent security review before enabling real BYOK.

## References

- https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Key_Management_Cheat_Sheet.html

## Approval record

- **ChatGPT review:** Proposed — initial review 2026-11-26
- **Owner decision:** Pending
