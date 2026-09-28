# Implementation Plan: User Authentication

**Branch**: `001-user-authentication` | **Date**: 2026-09-27 | **Spec**: [specs/001-user-authentication/spec.md](specs/001-user-authentication/spec.md)

**Input**: Feature specification from `/specs/001-user-authentication/spec.md`

## Summary

This feature adds secure authentication for a web product with three primary entry points: password-based sign-in, Google OAuth sign-in, and password recovery by email verification code. The design centers on a single normalized user account model, enforcement of a strong password policy, mandatory email verification for password-based accounts, and privacy-preserving recovery flows that prevent account enumeration while maintaining rate limits and safe retry behavior.

## Technical Context

**Language/Version**: Java 21 and Spring Boot latest release version for the backend; Angular latest release version for the frontend.

**Primary Dependencies**: Spring Security for authentication and authorization, BCryptPasswordEncoder for password hashing, and Resend API service for sending email verification and recovery messages. Google OAuth integration will be implemented via Spring Security OAuth2 client support.

**Storage**: PostgreSQL relational database for users, authentication identities, sessions, and password recovery records.

**Testing**: JUnit 5, Mockito, and AssertJ for unit tests; Testcontainers for integration tests. Frontend testing will use Angular-compatible unit and component test tooling as needed.

**Target Platform**: Web application with responsive layouts; native mobile-specific handling is out of scope unless the repository adds it later.

**Project Type**: Web application

**Performance Goals**: Auth flows should respond within 1 second for routine UI actions under normal conditions; email delivery timing is excluded from the product response requirement.

**Constraints**: Passwords and recovery codes must never be exposed in user-facing messages; recovery responses must be privacy-preserving; email verification is mandatory before password-based authenticated access; duplicate accounts must be prevented by normalized email identity.

**Scale/Scope**: The initial release is a standard user-authentication feature covering sign-in, sign-up, Google auth, email verification, and password recovery for a web audience; it does not include advanced admin-driven IAM or mobile-native behavior.

## Constitution Check

_GATE: Must pass before Phase 0 research. Re-check after Phase 1 design._

- Code Quality and Maintainability: PASS. The design keeps auth responsibilities grouped around account, recovery, and session services, and it defines the data invariants near the code.
- Testing and Verification: PASS. The feature requires unit, integration, and end-to-end checks for validation, recovery, and identity linking.
- Consistent and Accessible User Experience: PASS. The plan explicitly requires loading, success, error, cancellation, retry, responsiveness, and keyboard-accessible states across auth flows.
- Performance as a Product Requirement: PASS with implementation follow-up. The plan defines response expectations and notes that the team must verify them during implementation.
- Evidence-Based Technical Decisions: PASS. Decisions are recorded in the research and plan and known unknowns are explicitly marked as NEEDS CLARIFICATION rather than guessed.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)

<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
# [REMOVE IF UNUSED] Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# [REMOVE IF UNUSED] Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# [REMOVE IF UNUSED] Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]

ios/ or android/
└── [platform-specific structure: feature modules, UI flows, platform tests]
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation                  | Why Needed         | Simpler Alternative Rejected Because |
| -------------------------- | ------------------ | ------------------------------------ |
| [e.g., 4th project]        | [current need]     | [why 3 projects insufficient]        |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient]  |
