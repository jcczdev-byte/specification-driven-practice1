# Research: User Authentication

## 1. Authentication model

- Decision: Support two first-class identity methods: email/password and Google OAuth, both mapped to a single normalized user account.
- Rationale: The feature explicitly requires both flows, and the account model needs one identity record to enforce role defaults, duplicate prevention, and security policies consistently.
- Alternatives considered: Maintain separate account records per auth method; rejected because it would allow duplicates, complicate access control, and violate the requirement to prevent duplicate accounts for the same verified email address.

## 2. Password policy and validation

- Decision: Enforce a password policy of at least 12 characters with uppercase, lowercase, number, and special character for both registration and password reset.
- Rationale: This is explicitly required in the specification and must be enforced both when creating a new password-based account and when resetting a forgotten password.
- Alternatives considered: Simpler rules such as minimum length only; rejected because they would not satisfy the explicit security requirement and would fail the acceptance criteria for password strength.

## 3. Email verification and account access

- Decision: Password-based accounts must remain unverified until the user confirms their email address; only verified accounts can access authenticated experiences.
- Rationale: The feature requires evidence-based verification before access and includes clear resend flows for expired or missing verification messages.
- Alternatives considered: Allow immediate login after registration; rejected because it weakens account ownership assurance and contradicts the must-have verification requirement.

## 4. Recovery flow security

- Decision: Password recovery starts with a privacy-preserving request that responds equivalently for registered and unregistered addresses, and on success sends a single-use, time-limited verification code.
- Rationale: Requirements specify non-disclosing responses, time-limited code expiration, single-use enforcement, and rate limits for repeated attempts.
- Alternatives considered: Immediate password reset links or direct knowledge-based answers; rejected because they expose account existence and reduce security compared with code-based verification.

## 5. Google sign-in behavior

- Decision: Google-authenticated users are linked to their existing account by verified email if present; otherwise, a new trainee account is created only after any missing required profile data is gathered.
- Rationale: This matches both the requirements for linking existing accounts and the requirement that every created account defaults to the trainee role.
- Alternatives considered: Auto-create a new account for every Google sign-in; rejected because duplicate accounts would be created and the requirement to link existing identities would fail.

## 6. Role and permissions model

- Decision: All accounts created through this feature default to the trainee role, and the system will not allow self-elevation in registration or auth flows.
- Rationale: The feature explicitly mandates default role assignment and forbids user-selected role changes.
- Alternatives considered: Allow custom roles during signup; rejected because it violates the requirement and introduces privilege escalation risk.

## 7. Accessibility and user feedback

- Decision: Every auth transition must provide loading, success, error, cancellation, and retry states, with keyboard navigation and readable labels for all user-facing controls.
- Rationale: The feature and constitution both require accessible flows with safe feedback and no confusing silent failures.
- Alternatives considered: Minimal validation-only UI; rejected because it would not satisfy accessibility and state-management requirements for security-sensitive flows.

## 8. Data handling and observability

- Decision: Authentication and recovery flows will record security-relevant events for sign-ins, Google authentication, failures, code verification, password changes, and account linking.
- Rationale: The requirements demand event logging for compliance, security monitoring, and incident investigation.
- Alternatives considered: Logging only success events; rejected because it would not support detection of failed attempts, reuse, or abnormal access patterns.

## 9. Open technical decisions to confirm during implementation

- Decision: The implementation stack, persistence layer, and framework specifics remain to be selected by the engineering team working in the repository.
- Rationale: The feature spec defines the behavior and constraints but not the runtime framework, DB choice, or frontend architecture.
- Alternatives considered: Hard-coding a framework assumption; rejected because the repository does not yet contain a concrete tech stack and this would violate the evidence-based decision principle.

## Summary of resolved clarifications

- Minimum age for account creation: 18.
- Password rules: 12+ characters and all required character classes.
- Email verification required before a password-based account can access the product.
- Google authentication is treated as a verified identity source when an email is present.
- Recovery requests must not reveal whether an email is registered.
