# ATN-1806 — Account Creation - User Registration
## Spec

---

### Overview

As a user, I want to create an account so that I can save my shopping preferences and access personalised features within the application.

---

### Functional Requirements

| ID   | Requirement |
|------|-------------|
| FR-1 | The application must provide a registration form accessible via a "Sign Up Now!" link on the login page at route `/register`. |
| FR-2 | The registration form must include fields for full name, email address, and password. |
| FR-3 | The password field must enforce security requirements: minimum 8 characters, at least one numeric digit, and at least one special character. |
| FR-4 | Upon successful registration the user must see a confirmation message (simulating a confirmation email) and be redirected to `/auth` after 2 seconds. |
| FR-5 | The form must display clear, field-level error messages for all invalid inputs, including: blank required fields, invalid email format, duplicate email address (case-insensitive), and password that does not meet security requirements. |

---

### User Stories

#### Story 1 — Form Access
**Given** I am on the login page at `/auth`
**When** I click the "Sign Up Now!" link
**Then** I am navigated to `/register` and the registration form is displayed with fields for full name, email address, and password.

#### Story 2 — Successful Registration
**Given** I am on the registration form at `/register`
**When** I enter a valid full name (2 or more characters), a valid email address not already in use, and a password that meets security requirements, and I submit the form
**Then** a success confirmation message is displayed, and I am automatically redirected to `/auth` after 2 seconds.

#### Story 3 — Password Validation Failure
**Given** I am on the registration form at `/register`
**When** I submit the form with a password that is fewer than 8 characters, or lacks a numeric digit, or lacks a special character
**Then** an inline error message is displayed beneath the password field describing the unmet requirement, and the form is not submitted.

#### Story 4 — Duplicate or Invalid Email
**Given** I am on the registration form at `/register`
**When** I submit the form with an email address that is already registered (case-insensitive match) or is not a valid email format
**Then** an inline error message is displayed beneath the email field, and the form is not submitted.

#### Story 5 — Empty Required Fields
**Given** I am on the registration form at `/register`
**When** I submit the form without filling in one or more required fields
**Then** inline error messages are displayed beneath each empty required field, and the form is not submitted.

---

### Acceptance Criteria

| # | Criterion | Priority | Status |
|---|-----------|----------|--------|
| AC-1 | The registration form is accessible at `/register` via a link from `/auth`. | Must | Open |
| AC-2 | The form contains fields for full name, email address, and password, all of which are required. | Must | Open |
| AC-3 | Password validation enforces: minimum 8 characters, at least 1 digit, and at least 1 special character. | Must | Open |
| AC-4 | Submitting the form with a duplicate email (case-insensitive) shows an error message and prevents submission. | Must | Open |
| AC-5 | A success message is shown after valid registration, and the user is redirected to `/auth` after 2 seconds. | Must | Open |
| AC-6 | All required-field and format errors are shown inline beneath the relevant field. | Must | Open |

---

### Assumptions

| ID  | Assumption |
|-----|------------|
| A-1 | No real backend API exists. Registration data is persisted to `localStorage` under the key `registeredUsers`. |
| A-2 | Confirmation email is simulated via a UI success message only — no real email is sent. |
| A-3 | Duplicate email check is case-insensitive. |
| A-4 | Registration page route is `/register`. |
| A-5 | User is redirected to `/auth` after 2 seconds on successful registration. |
| A-6 | Full name field requires a minimum of 2 characters. |

---

### Out of Scope

- Real backend API integration or server-side user persistence
- Actual email delivery or email verification links
- OAuth or third-party sign-up providers (Google, Facebook, etc.)
- Password confirmation / re-enter password field
- CAPTCHA or bot protection
- Account management (profile editing, password reset)
- Email uniqueness enforced server-side
