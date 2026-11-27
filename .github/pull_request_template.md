## Linked issue
WG-___ (do not use closing keywords until the owner has accepted the work)

## Summary
<!-- What changed and why. Keep it to one concern. -->

## Scope check
- [ ] Within the issue's scope; no unapproved scope change
- [ ] Planning/documentation only **or** the owner has approved this implementation handover
- [ ] No production deployment, paid service, hosted migration or production secret involved

## Definition of done (see docs/DEVELOPMENT_AND_TESTING_STANDARDS.md)
- [ ] Acceptance criteria met, evidence linked
- [ ] Tests added/updated and passing (unit, integration, contract, security as applicable)
- [ ] Tenant isolation enforced and tested (workspace identifier)
- [ ] Authorisation enforced server-side; audit events emitted for sensitive actions
- [ ] No secrets, keys or customer data in code, fixtures, logs or docs
- [ ] Documentation and issue register updated; links resolve
- [ ] New dependencies justified and licence-compatible

## Risk and security notes
<!-- Threat-model references, new permissions, new external calls. -->

## Reviewer
- [ ] Independent architectural review requested (significant changes)
- [ ] Owner approval required before merge to main
