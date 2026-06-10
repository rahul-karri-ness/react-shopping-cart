# ATN-1806 - Account Creation - User Registration

## Overview

As a user, I want to create an account so that I can save my shopping preferences.

---

## Functional Requirements

### FR-1: Registration Form Access
A user must be able to navigate to the registration page from the login page by clicking the "Sign Up Now!" link. The registration page must be accessible at the route `/register`.

### FR-2: Registration Form Fields
The registration form must contain the following fields:
- Full Name (text, required, minimum 2 characters)
- Email Address (email, required, valid email format)
- Password (password, required, minimum 8 characters, must include at least one number and one special character)
- Confirm Password (password, required, must match the Password field)

### FR-3: Password Security Requirements
The password field must enforce the following rules:
- Minimum 8 characters
- At least one numeric digit
- At least one special character (e.g., `!@#$%^&*`)
- Confirm Password must match Password

### FR-4: Successful Registration
Upon successful form submission with valid data:
- The user's registration data is persisted to localStorage under the key `registeredUsers`
- A UI success message is displayed simulating a confirmation email notification
- The user is automatically redirected to `/auth` after 2 seconds

### FR-5: Error Handling and Validation
The form must display inline error messages for:
- Empty required fields
- Invalid email format
- Email address already registered (case-insensitive duplicate check)
- Password not meeting security requirements
- Confirm Password not matching Password

---

## User Stories

### Story 1: Accessing the Registration Form
**Given** a user is on the login page at `/auth`
**When** the user clicks the "Sign Up Now!" link
**Then** the user is navigated to the registration form at `/register`

### Story 2: Successful Registration
**Given** a user is on the registration page at `/register`
**When** the user fills in a valid full name, a unique email address, a password meeting security requirements, and a matching confirm password, then submits the form
**Then** the registration data is saved to localStorage under `registeredUsers`, a success message is displayed, and the user is redirected to `/auth` after 2 seconds

### Story 3: Password Validation Failure
**Given** a user is on the registration page at `/register`
**When** the user submits the form with a password that does not meet security requirements (e.g., too short, missing a number, or missing a special character)
**Then** an inline error message is displayed beneath the password field describing the specific requirement that was not met, and the form is not submitted

### Story 4: Duplicate or Invalid Email
**Given** a user is on the registration page at `/register`
**When** the user submits the form with an email address that is already registered (case-insensitive) or with an invalid email format
**Then** an appropriate inline error message is displayed beneath the email field, and the form is not submitted

### Story 5: Empty Required Fields
**Given** a user is on the registration page at `/register`
**When** the user submits the form without filling in one or more required fields
**Then** inline error messages are displayed beneath each empty required field indicating that the field is required, and the form is not submitted

---

## Acceptance Criteria

| # | Criterion | Priority |
|---|-----------|----------|
| AC-1 | User can navigate to `/register` by clicking "Sign Up Now!" on the login page | Must |
| AC-2 | Registration form contains fields for Full Name, Email, Password, and Confirm Password | Must |
| AC-3 | Password must be at least 8 characters, contain a number and a special character | Must |
| AC-4 | Confirm Password must match Password; an error is shown if they do not match | Must |
| AC-5 | Successful registration saves data to localStorage and shows a UI success message | Must |
| AC-6 | User is redirected to `/auth` 2 seconds after successful registration | Must |
| AC-7 | Duplicate email check is case-insensitive; an error is shown if email is already in use | Must |
| AC-8 | Inline validation error messages are shown for all invalid or empty fields | Must |

---

## Assumptions

| ID | Assumption |
|----|-----------|
| A-1 | No real backend API exists. Registration data is persisted to localStorage under the key `registeredUsers`. |
| A-2 | Confirmation email is simulated via a UI success message only; no actual email is sent. |
| A-3 | Duplicate email check is case-insensitive. |
| A-4 | Registration page route is `/register`. |
| A-5 | User is redirected to `/auth` after 2 seconds on successful registration. |
| A-6 | Full name field requires a minimum of 2 characters. |

---

## Out of Scope

- Real backend API integration or server-side user persistence
- Actual email delivery or email verification flow
- OAuth or third-party authentication (Google, Facebook, etc.)
- Account management (profile editing, password reset)
- Email uniqueness enforced server-side
- CAPTCHA or bot protection
- Multi-step registration wizard
