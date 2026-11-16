# Business model and authoring examples

## Service request versus requirement

A **service request** starts from something already built or already offered: support, an incident or a service. Examples: “The existing Export button downloads an empty file,” “I did not receive the OTP,” or “I need access to the published module.” Manage it with `nexora_request_create`, `nexora_request_comment`, attachments and, when applicable, `nexora_request_link`.

A **requirement** expresses something new or an intentional behavior change that must be defined before it is built. Examples: “Allow PDF and CSV exports,” “Add OTP with expiration and resend,” or “Keep an audit trail for every ownership change.” Register it as a versioned document, normally with `definition.content.kind = requirement`.

A service request may reveal a requirement, but it never becomes one automatically. Preserve the conversation and create the new document once the new need is defined.

| Case | Type | Reason |
|---|---|---|
| “The current report duplicates rows” | Service request | Failure of an existing capability. |
| “Create reports exportable to PDF” | Requirement | New capability. |
| “The recovery email never arrives” | Service request | Operational diagnosis. |
| “Add recovery with a one-time link” | Requirement | New behavior. |

## Layers the agent must distinguish

- **Requirement**: need, outcome, scope and criteria.
- **Feature**: stable, reusable capability; multiple requirements may extend it.
- **Business rule**: mandatory condition, calculation, authorization, limit or invariant.
- **Technical decision**: architecture/implementation choice, alternatives and consequences.
- **Good practice**: recommendation with scope and rationale; it is not mandatory until adopted.
- **Security**: asset, threat, boundary, control, evidence and residual risk.
- **Technical debt**: concrete future cost, cause, impact, priority and repayment criterion.

Every assertion declares provenance: `declared_business`, `observed_code` or `agent_proposed`. Code observed in a snapshot verifies implementation, not approved business intent.

## Requirement example

```markdown
# REQ-OTP-001 — Login verification with OTP

## Problem
High-risk accounts need a second factor.

## Objective
Allow login confirmation through an OTP delivered by an approved channel.

## Scope
- Includes issuance, verification, expiration, resend and attempt blocking.
- Excludes account recovery.

## Actors
- Partially authenticated user
- Identity service
- Support team

## Acceptance criteria
- A valid OTP allows the login to continue.
- An expired OTP is rejected without revealing sensitive information.
- Resend and attempt limits follow the approved policy.

## Open questions
- Which channel is approved and what expiration is authorized?

## Provenance
declared_business; the policy must be resolved before publication.
```

Send `acceptance_criteria` as an array of strings, one criterion per element, without bullets, numbering or checkboxes. Read the exact requirement and call `nexora_features_list` before proposing its map. Use `nexora_feature_map_propose` and wait for a human decision.

## Feature example

```markdown
# Authentication

## Purpose
Manage identity and access factors.

## Capabilities
- Login
- OTP verification

## Requirements
- REQ-OTP-001, exact version

## Boundary
Do not store secrets outside the approved credential mechanism.
```

Reuse a feature when the change shares its domain and lifecycle. Create a new feature only when there is a real boundary and justification; do not duplicate “Authentication” for every requirement.

## Business-rule example

```markdown
# RB-OTP-003 — Verification attempts

## Rule
After the approved maximum is exceeded, the challenge is invalidated and a new one must be issued.

## Scope
Each OTP challenge and account according to the current policy.

## Source
The maximum and time window must come from an approved business authority.

## Evidence
REQ-OTP-001, exact version, plus the snapshot/path/lines returned by Nexora.
```

If a value is missing, record the question; do not invent “3 attempts” or “5 minutes.”

## Technical-decision example

```markdown
# ADR-014 — OTP verification in Identity

## Context
Authentication already exists and needs a second factor.

## Decision
Identity will issue short-lived challenges and persist only the digest needed for verification.

## Rejected alternatives
- Plaintext OTP storage: unnecessary exposure.
- Client-only verification: the client is not a trusted boundary.

## Consequences
Expiration, anti-replay, metrics and concurrency tests are required.

## Status
Proposed; requires authorized technical review.
```

## Good-practice example

```markdown
# BP-SEC-007 — Do not log secrets

## Recommendation
Do not log OTPs, tokens, passwords, API keys or credential-bearing links.

## Applies to
Logs, comments, evidence, attachments and Nexora documents.

## Correct example
Record request_id, event type and a sanitized result.
```

Declare whether it is a recommendation or an adopted rule; do not present a preference as policy.

## Security example

```markdown
# SEC-OTP-002 — Challenge replay

## Asset
Identity and session.

## Threat
Reuse of an intercepted OTP.

## Boundary
Untrusted client → Identity API → challenge storage.

## Control
Single use, expiration, rate limiting, constant-time comparison and secret-free audit.

## Evidence and residual risk
Replay, concurrency and expiration tests; delivery channel remains a documented residual risk.
```

Never publish credentials as knowledge or attach them as evidence.

## Technical-debt example

```markdown
# DT-032 — Missing idempotency for OTP resend

## Problem
A timeout may issue duplicate challenges.

## Cause
The endpoint does not retain an idempotency key per operation.

## Impact
Confusing state, extra messaging cost and additional support work.

## Resolution criterion
The same request_id produces a stable result and timeout/concurrency tests exist.

## Provenance
Snapshot, path and lines returned by Nexora; this is not yet a fix.
```

“Refactor everything” without cause, impact and a criterion is not actionable technical debt.

## Linked examples

Service request:

```markdown
## Symptom
The user does not receive an OTP while logging in to the existing product.

## Expected outcome
Identify whether issuance, provider or delivery failed without exposing the code.

## Evidence
Timestamp with timezone, request_id and a secret-free screenshot.
```

Derived task:

```markdown
## Objective
Investigate the delivery failure reported in SOL-204.

## Not included
Do not implement a new second factor or change expiration.

## Acceptance
- The failure point is identified with reproducible evidence.
- The link to SOL-204 is preserved.
- If a new capability is required, propose REQ-OTP-001 separately.
```

## Authoring workflow

1. Identify actor, permissions, project and whether this is operational support or a new need.
2. Read organization/project knowledge: applicable requirements, features, rules, decisions, practices, security and debt.
3. Query authorized code to separate observed implementation from business intent.
4. Search existing documents and features, preserving exact versions.
5. Write scope, actors, criteria, questions, sources and evidence.
6. Create/revise using the current schema, a stable `request_id` and expected revision.
7. Link request, requirement, feature, task and evidence only through authorized relations.
8. Request human review/publication when applicable; never simulate it with MCP.
9. Read the result again and report ID, revision, state, publication and evidence limits.

## People, time and developer-governance records

Nexora separates delivery evidence from labor and people records:

- A **work log** records actual time spent on a task, with date and minutes. Workflow elapsed time is not worked time.
- A **work block** records planned personal calendar time; it is not proof that work happened.
- An **overtime request** asks for authorization before or according to the applicable labor policy. Approval must be decided by an authorized other person; it does not create hours automatically.
- **Overtime evidence** connects approved time to eligible tasks/commits/events. Preserve the original interval and timezone; adding evidence must not silently alter approval or minutes.
- A **leave/permission request** and a **holiday proposal** are calendar decisions with their own authority and states. Nobody approves their own request.
- A **developer rating** is an authorized evaluation based on governed evidence and the product's rating policy. Do not infer personal qualities from a task, commit, pusher identity or one incident; do not expose private feedback as general knowledge.

Example of a correct distinction:

```markdown
## Work record
Task: TASK-204
Date: 2026-11-15
Minutes: 90
Evidence: actual investigation and test execution recorded for this task.

## Overtime request
Reason: incident investigation exceeded the approved schedule.
Interval: 2026-11-15 18:00–20:00 America/El_Salvador
Requested minutes: 120
Approval: pending; no hours are treated as approved yet.
```

Never use a rating, calendar block or overtime request as a shortcut to complete a task or claim production delivery. These capabilities may be available through authorized HTTP/API pages rather than the MCP tool catalog; follow the actual surface and permission checks.
