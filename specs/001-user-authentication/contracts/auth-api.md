# Authentication API Contract

## Overview

This contract describes the public authentication and account-recovery interfaces for the product. The exact transport layer can be HTTP, RPC, or another web framework, but the request and response semantics should remain equivalent.

## 1. Register account

### POST /auth/register

Request body:

```json
{
  "firstName": "Ada",
  "lastName": "Lovelace",
  "email": "ada@example.com",
  "dateOfBirth": "1990-10-15",
  "gender": "female",
  "password": "StrongPass!123"
}
```

Success response (201):

```json
{
  "user": {
    "id": "user_123",
    "email": "ada@example.com",
    "role": "trainee",
    "status": "pending_verification"
  },
  "verificationRequired": true
}
```

Error responses:

- 400 for invalid required fields or weak password.
- 409 when an account already exists for the email address.

## 2. Sign in with password

### POST /auth/login

Request body:

```json
{
  "email": "ada@example.com",
  "password": "StrongPass!123"
}
```

Success response (200):

```json
{
  "session": {
    "id": "sess_123",
    "expiresAt": "2026-09-27T12:30:00Z"
  },
  "user": {
    "id": "user_123",
    "email": "ada@example.com",
    "role": "trainee"
  }
}
```

Error responses:

- 401 for invalid credentials.
- 403 when the account exists but is not yet verified.

## 3. Google sign-in

### POST /auth/google/start

Request body:

```json
{
  "redirectUri": "https://app.example.com/oauth/google/callback"
}
```

Success response (200):

```json
{
  "authorizationUrl": "https://accounts.google.com/o/oauth2/v2/auth?..."
}
```

### POST /auth/google/callback

Request body:

```json
{
  "code": "oauth_code",
  "state": "csrf_token"
}
```

Success response (200):

```json
{
  "session": {
    "id": "sess_456",
    "expiresAt": "2026-09-27T12:45:00Z"
  },
  "user": {
    "id": "user_123",
    "email": "ada@example.com",
    "role": "trainee"
  },
  "created": false
}
```

Error responses:

- 400 when OAuth state is invalid or email is missing.
- 401 when Google authentication fails.

## 4. Password recovery request

### POST /auth/recovery/request

Request body:

```json
{
  "email": "ada@example.com"
}
```

Success response (202):

```json
{
  "status": "accepted",
  "message": "If that account exists, a verification code has been sent."
}
```

Notes:

- The response must not reveal whether the email is registered.
- Rate limiting should apply to repeated requests.

## 5. Password recovery verification and reset

### POST /auth/recovery/verify

Request body:

```json
{
  "email": "ada@example.com",
  "code": "123456",
  "newPassword": "NewStrongPass!456"
}
```

Success response (200):

```json
{
  "status": "password-reset-successful"
}
```

Error responses:

- 400 for invalid or weak password.
- 401 for invalid, expired, or already-used code.
- 429 when rate limiting is active.

## 6. Email verification

### POST /auth/verify-email

Request body:

```json
{
  "email": "ada@example.com",
  "code": "verify_code"
}
```

Success response (200):

```json
{
  "status": "verified",
  "user": {
    "id": "user_123",
    "status": "active"
  }
}
```

Error responses:

- 400 for missing or malformed code.
- 401 for expired or invalid code.

## Contract invariants

- Passwords and codes are never returned in clear text to client-facing responses.
- A successful recovery flow invalidates the code immediately after use.
- Registration and recovery flows must preserve safe, privacy-preserving error messaging.
- User role assignment remains server-controlled and defaults to trainee.
