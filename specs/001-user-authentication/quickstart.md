# Quickstart: Authentication Validation

## Prerequisites

- A working web application environment with a configured database and user table.
- Email delivery configured for verification and password recovery messages.
- Google OAuth client credentials configured for the product environment.
- Test users and access to an authentication test suite or browser automation runner.

## Validation scenarios

### 1. Sign in with email and password

1. Create an active account with a valid email and password that meets the policy.
2. Sign in using the exact email address and password.
3. Confirm that the user lands in the authenticated experience.
4. Attempt sign-in with the wrong password and confirm the flow shows a clear, non-sensitive error and does not reveal whether the account exists.

Expected outcome: Sign-in succeeds for valid credentials and fails safely for invalid ones.

### 2. Registration and verification

1. Navigate to the create-account flow.
2. Submit first name, last name, email, date of birth, gender, and a valid password.
3. Confirm that the account is created with the trainee role and that a verification email is sent.
4. Attempt to access protected functionality before verification.
5. Complete the email verification step and repeat the access check.

Expected outcome: Before verification, access is blocked; after verification, access is allowed.

### 3. Google authentication

1. Start Google sign-in for an account with a new email address.
2. Confirm that the user is either signed in or asked only for missing profile information required by the product.
3. Repeat the flow for an email address already associated with an account.
4. Confirm that no duplicate account is created.

Expected outcome: A new account is created only when appropriate, and existing accounts are linked by verified email.

### 4. Password reset

1. Start password recovery from the sign-in screen.
2. Enter a registered or unregistered email address and confirm the response is equivalent across both states.
3. Complete the recovery code flow with a valid code and a compliant new password.
4. Confirm the old password no longer works and the new password works.
5. Reuse the code to confirm the change is rejected.

Expected outcome: Recovery is secure, privacy-preserving, and single-use.

### 5. Security and resilience checks

1. Attempt repeated invalid recovery codes until the limit is reached.
2. Confirm the user receives a safe retry path after the lockout window.
3. Check that leaked password, verification token, or code values are never exposed in page text or logs.
4. Test cancellation and resend behavior for email verification and recovery flows.

Expected outcome: The app blocks abuse attempts while keeping feedback clear and safe for users.

## Recommended automated checks

- Unit tests for validation rules, hashing, token generation, and expiry logic.
- Integration tests for account creation, Google linking, and password reset flows.
- End-to-end tests for the critical user journeys listed above.
- Accessibility checks for keyboard navigation, visible focus states, and clear validation messaging.

## Acceptance signal

The feature is ready to move from plan to implementation when all validation scenarios above pass and the security requirements in the specification are enforced consistently in automated tests.
