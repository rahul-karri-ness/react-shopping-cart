# Spec: ATN-1806 — Account Creation / User Registration

## Overview

**Ticket:** ATN-1806  
**Title:** Account Creation - User Registration  
**Type:** Story  
**Priority:** Major  
**Status:** In Progress  

As a user, I want to create an account so that I can save my shopping preferences.

---

## Functional Requirements

### FR-1: Registration Form Access
- The application must provide a dedicated registration page/view accessible from the login/auth page.
- The registration form must contain the following fields:
  - **Full Name** (text input, required)
  - **Email Address** (email input, required)
  - **Password** (password input, required)
  - **Confirm Password** (password input, required)

### FR-2: Password Security Requirements
- Password must be a minimum of **8 characters**.
- Password must contain at least **one numeric digit** (0–9).
- Password must contain at least **one special character** (e.g., `!@#$%^&*`).
- Confirm Password field must match the Password field exactly.
- Validation errors must be shown inline, adjacent to the relevant field.

### FR-3: Successful Registration Flow
- On successful form submission, the user receives a **confirmation email** to the provided address.
- After submission, the user is shown a success message indicating that a confirmation email has been sent.
- The user is not automatically logged in after registration (email confirmation required first).

### FR-4: Error Handling & Inline Validation
- If the email address is already registered, an error message must be displayed: *"This email address is already in use."*
- All required fields must show inline validation errors when left empty on submit.
- Password mismatch must show: *"Passwords do not match."*
- Invalid email format must show: *"Please enter a valid email address."*
- Password not meeting security requirements must show a descriptive error message.

### FR-5: Navigation
- The login page must include a link/button to navigate to the registration form.
- The registration page must include a link to navigate back to the login page.

---

## User Stories

### Story 1: Access the Registration Form

**Given** I am on the login/auth page  
**When** I click the "Sign Up Now!" link  
**Then** I am navigated to the registration form  
**And** I see fields for Full Name, Email, Password, and Confirm Password  

---

### Story 2: Successful Account Registration

**Given** I am on the registration form  
**When** I fill in a valid Full Name, a unique Email, a Password meeting all security requirements, and a matching Confirm Password  
**And** I click the "Register" button  
**Then** my account is created  
**And** I see a success message: *"Registration successful! Please check your email to confirm your account."*  
**And** a confirmation email is sent to the provided email address  

---

### Story 3: Password Validation Failure

**Given** I am on the registration form  
**When** I enter a password that does not meet the security requirements (e.g., fewer than 8 characters, no number, or no special character)  
**And** I click the "Register" button  
**Then** I see an inline error message describing the unmet requirement  
**And** the form is not submitted  

---

### Story 4: Duplicate Email Error

**Given** I am on the registration form  
**When** I enter an email address that is already associated with an existing account  
**And** I click the "Register" button  
**Then** I see an error message: *"This email address is already in use."*  
**And** the form is not submitted  

---

### Story 5: Required Field Validation

**Given** I am on the registration form  
**When** I submit the form without filling in one or more required fields  
**Then** I see inline error messages for each empty required field  
**And** the form is not submitted  

---

### Story 6: Password Mismatch

**Given** I am on the registration form  
**When** I enter a Password and a Confirm Password that do not match  
**And** I click the "Register" button  
**Then** I see an inline error: *"Passwords do not match."*  
**And** the form is not submitted  

---

## Acceptance Criteria

| # | Criterion | Priority |
|---|-----------|----------|
| AC-1 | Registration form is accessible from the auth/login page via a "Sign Up" link | Must Have |
| AC-2 | Form contains fields: Full Name, Email, Password, Confirm Password | Must Have |
| AC-3 | Password must be ≥ 8 characters, contain a number, and contain a special character | Must Have |
| AC-4 | Confirm Password must match Password | Must Have |
| AC-5 | Successful registration triggers a confirmation email to the user | Must Have |
| AC-6 | Success message is shown after registration | Must Have |
| AC-7 | Duplicate email shows error: "This email address is already in use." | Must Have |
| AC-8 | All required fields show inline validation errors when empty on submit | Must Have |
| AC-9 | Invalid email format shows appropriate error | Must Have |
| AC-10 | Password mismatch shows: "Passwords do not match." | Must Have |
| AC-11 | Registration page has a link back to the login page | Should Have |

---

## Assumptions

1. **No backend API exists yet** — the current app uses `localStorage` for auth state. The registration flow will simulate the confirmation email (e.g., log to console or show a mock success state) and store the user in `localStorage`. A real email service integration is out of scope for this ticket.
2. **Email uniqueness check** will be performed against users stored in `localStorage` (no server-side check).
3. **Auto-login after registration is NOT performed** — the user must use the login form after confirming their email (simulated by showing a success message).
4. **The existing `/auth` route** will be extended to support a toggle between Login and Register views (no new route required unless the team prefers a separate `/register` route).
5. **Confirmation email** is simulated — no real email service (e.g., SendGrid, SES) is integrated in this ticket.
6. **The existing `AuthProvider` context** will be extended to handle registration state and actions.

---

## Out of Scope

- Social login (Google, Facebook, etc.)
- Email verification link handling / token-based confirmation
- Real email delivery service integration
- Password strength meter UI
- CAPTCHA / bot protection
- Account management / profile editing
- Forgot password flow (separate ticket)
- Backend API integration (future ticket)
- Multi-factor authentication (MFA)
