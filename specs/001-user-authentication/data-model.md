# Data Model: User Authentication

## Overview

The authentication domain centers on a single account identity with multiple authentication methods and recovery records. The goal is to ensure one normalized user profile while supporting both password-based and Google-based sign-in without duplicate identities.

## Entities

### UserAccount

| Field                | Type        | Constraints                                                                     | Notes                                                          |
| -------------------- | ----------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| id                   | UUID/string | Required, unique                                                                | Primary identifier for the account.                            |
| firstName            | string      | Required, trimmed, 1-100 chars                                                  | Normalized before validation.                                  |
| lastName             | string      | Required, trimmed, 1-100 chars                                                  | Normalized before validation.                                  |
| email                | string      | Required, lowercase-normalized, unique, verified                                | Canonical lookup key for duplicates and account matching.      |
| dateOfBirth          | date        | Required, valid and age >=18                                                    | Rejected if future or invalid.                                 |
| gender               | enum/string | Required, allowed values include self-described options and “Prefer not to say” | Stored as profile metadata.                                    |
| passwordHash         | string      | Required for password-based accounts; nullable for Google-only accounts         | Must never expose raw password.                                |
| role                 | enum/string | Required, default = trainee                                                     | Cannot be self-selected or elevated by user.                   |
| status               | enum/string | Required                                                                        | Values include pending_verification, active, locked, disabled. |
| createdAt            | datetime    | Required                                                                        | Audit timestamp.                                               |
| updatedAt            | datetime    | Required                                                                        | Audit timestamp.                                               |
| lastLoginAt          | datetime    | Optional                                                                        | Updated after successful sign-in.                              |
| lastPasswordChangeAt | datetime    | Optional                                                                        | Protects recovery and session invalidation.                    |

Validation rules:

- Email addresses are trimmed, lower-cased, and compared case-insensitively.
- Password-based accounts cannot be considered active until email verification succeeds.
- Role assignment is server-side only and defaults to trainee.
- Duplicate account creation is rejected for the same verified email address.

### Role

| Field       | Type     | Constraints      | Notes                                |
| ----------- | -------- | ---------------- | ------------------------------------ |
| code        | string   | Required, unique | e.g. trainee                         |
| name        | string   | Required         | Human-readable role label.           |
| permissions | string[] | Required         | Authorization scope for the account. |

Rules:

- New accounts created by this feature must be assigned the trainee role.
- The user cannot select or change their own role during registration or authentication.

### GoogleIdentity

| Field          | Type        | Constraints                   | Notes                                                            |
| -------------- | ----------- | ----------------------------- | ---------------------------------------------------------------- |
| id             | UUID/string | Required, unique              | Provider identity primary key.                                   |
| userAccountId  | UUID/string | Required, foreign key         | Links to the owning user.                                        |
| provider       | string      | Required, fixed = google      | Identity provider name.                                          |
| providerUserId | string      | Required, unique per provider | Google account subject or equivalent.                            |
| email          | string      | Required, verified            | Used for account matching and reconciliation.                    |
| profileName    | string      | Optional                      | Initial name metadata from OAuth provider.                       |
| isPrimary      | boolean     | Required                      | Whether this Google identity is the primary sign-in association. |
| createdAt      | datetime    | Required                      | Audit timestamp.                                                 |

Rules:

- If the verified Google email matches an existing user, the sign-in is linked to that account instead of creating a duplicate.
- If the Google account lacks a usable email, sign-in fails without creating the account.

### PasswordRecoveryRequest

| Field                | Type        | Constraints          | Notes                                                 |
| -------------------- | ----------- | -------------------- | ----------------------------------------------------- |
| id                   | UUID/string | Required, unique     | Recovery request key.                                 |
| email                | string      | Required, normalized | Target account lookup.                                |
| codeHash             | string      | Required             | Hashed verification code; never stored in plain text. |
| expiresAt            | datetime    | Required             | Time-limited expiration window.                       |
| usedAt               | datetime    | Optional             | Marks single-use completion.                          |
| expiresAfterAttempts | int         | Required             | Rate-limit or attempt threshold.                      |
| status               | enum/string | Required             | values: pending, verified, expired, replaced, blocked |
| requestCount         | int         | Required             | Tracks repeated requests.                             |
| createdAt            | datetime    | Required             | Audit timestamp.                                      |

Rules:

- Recovery codes are single-use and invalidated after use or replacement.
- Requests must be privacy-preserving; they must not disclose whether the address exists.
- Repeated attempts beyond the limit trigger temporary restriction.

### AuthenticationSession

| Field         | Type        | Constraints           | Notes                              |
| ------------- | ----------- | --------------------- | ---------------------------------- |
| id            | UUID/string | Required, unique      | Session token key.                 |
| userAccountId | UUID/string | Required, foreign key | Authenticated account.             |
| issuedAt      | datetime    | Required              | Session creation time.             |
| expiresAt     | datetime    | Required              | TTL for access state.              |
| revokedAt     | datetime    | Optional              | Used for sign-out or invalidation. |
| authMethod    | enum/string | Required              | password or google                 |
| deviceInfo    | string      | Optional              | For audit and security review.     |

Rules:

- Successful sign-in or registration creates an active session.
- Recovered or expired credentials, sign-out, or security events revoke prior sessions when required.

## Relationships

- A `UserAccount` may have zero or more `GoogleIdentity` records.
- A `UserAccount` has zero or more `AuthenticationSession` records over time.
- A `UserAccount` may have several `PasswordRecoveryRequest` records across different recovery flow attempts.
- `Role` is a lookup entity assigned to each account by the server.

## State transitions

### UserAccount lifecycle

- `pending_verification` → `active` after successful email verification.
- `active` → `locked` after repeated failed sign-in or security events.
- `active` or `pending_verification` → `disabled` if account is administratively suspended.

### PasswordRecoveryRequest lifecycle

- `pending` → `verified` after successful code validation.
- `pending` → `expired` when TTL is exceeded.
- `pending` → `replaced` when a new code is issued.
- `pending` → `blocked` after rate-limit enforcement.

### AuthenticationSession lifecycle

- `issued` → `revoked` on sign-out, password reset, or security invalidation.
- `issued` → `expired` when TTL passes without refresh.
