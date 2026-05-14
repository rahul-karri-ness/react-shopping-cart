# Spec: ATN-1806 — Account Creation / User Registration

## Overview

**Jira Ticket:** [ATN-1806](https://ness-nde.atlassian.net/browse/ATN-1806)  
**Type:** Story  
**Priority:** Major  
**Assignee:** Rahul Karri

> As a user, I want to create an account so that I can save my shopping preferences.

---

## Functional Requirements

| # | Requirement |
|---|-------------|
| FR-1 | The application shall provide a dedicated registration page accessible from the login page via a "Sign Up" link. |
| FR-2 | The registration form shall include fields for: **Full Name**, **Email Address**, and **Password**. |
| FR-3 | The password field shall enforce the following security rules: minimum 8 characters, at least one numeric digit, and at least one special character (e.g., `!@#$%^&*`). |
| FR-4 | The system shall display inline, field-level error messages when the user submits invalid or incomplete data. |
| FR-5 | The system shall prevent registration if the submitted email address is already associated with an existing account. |
| FR-6 | Upon successful registration, the system shall send a confirmation email to the user's provided email address. |
| FR-7 | Upon successful registration, the user shall be redirected to an appropriate page (e.g., login page or home page with a success notification). |
| FR-8 | All form fields shall be validated both on blur (field-level) and on submit (form-level). |

---

## User Stories

### Story 1 — Access the Registration Form

**Given** I am on the Login (`/auth`) page,  
**When** I click the "Sign Up Now!" link,  
**Then** I am navigated to the Registration page (`/register`),  
**And** I see a form with fields for Full Name, Email Address, and Password.

---

### Story 2 — Successful Account Registration

**Given** I am on the Registration page,  
**When** I fill in a valid Full Name, a valid Email Address, and a Password that meets all security requirements,  
**And** I submit the form,  
**Then** my account is created,  
**And** I receive a confirmation email at the address I provided,  
**And** I am redirected to the login page (or home page) with a success message.

---

### Story 3 — Password Validation Enforcement

**Given** I am on the Registration page,  
**When** I enter a password that does not meet the security requirements (e.g., fewer than 8 characters, no number, or no special character),  
**And** I attempt to submit the form or move focus away from the password field,  
**Then** an inline error message is displayed beneath the password field describing the unmet requirement.

---

### Story 4 — Duplicate Email Prevention

**Given** I am on the Registration page,  
**When** I enter an email address that is already registered in the system,  
**And** I submit the form,  
**Then** an error message is displayed informing me that the email is already in use,  
**And** my account is not created.

---

### Story 5 — Invalid Input Error Messages

**Given** I am on the Registration page,  
**When** I submit the form with one or more empty or invalid fields (e.g., malformed email, empty name),  
**Then** inline error messages are displayed for each invalid field,  
**And** the form is not submitted.

---

## Acceptance Criteria

| # | Criterion | Priority |
|---|-----------|----------|
| AC-1 | Registration form is accessible at `/register` and contains Name, Email, and Password fields. | Must Have |
| AC-2 | Password validation enforces: min 8 chars, at least 1 number, at least 1 special character. | Must Have |
| AC-3 | A confirmation email is triggered upon successful registration. | Must Have |
| AC-4 | Inline error messages appear for invalid inputs (empty fields, bad email format, weak password). | Must Have |
| AC-5 | Submitting a duplicate email shows a clear error message and does not create a new account. | Must Have |
| AC-6 | Successful registration redirects the user and shows a success notification. | Must Have |
| AC-7 | All form fields show validation errors on blur and on submit. | Should Have |
| AC-8 | The registration page uses the existing `AuthLayout` and matches the visual style of the login page. | Should Have |

---

## Assumptions

| # | Assumption |
|---|------------|
| A-1 | Email confirmation is simulated on the frontend (e.g., a toast/alert notification) since no backend email service is currently integrated. |
| A-2 | Duplicate email detection is handled client-side (mock/stub) as there is no live backend API in this repository. |
| A-3 | The registration flow stores the new user in `localStorage` consistent with the existing `signIn` pattern in `contexts/auth.jsx`. |
| A-4 | The "Full Name" field maps to a `name` property on the user object. |
| A-5 | After successful registration, the user is automatically signed in and redirected to the home page (`/`). |
| A-6 | No email uniqueness database exists; uniqueness is checked against the currently stored user in `localStorage`. |

---

## Out of Scope

- Backend API integration for user persistence (database, REST/GraphQL endpoints).
- Real email delivery service integration (SendGrid, SES, etc.).
- OAuth / social login (Google, Facebook, etc.).
- Email verification link flow (click-to-verify).
- Account management (profile editing, password reset).
- CAPTCHA or bot protection.
- Multi-step registration wizard.
