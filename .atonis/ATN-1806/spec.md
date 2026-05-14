# Spec: Account Creation — User Registration
**Ticket:** ATN-1806  
**Type:** Story  
**Priority:** Major  
**Status:** In Progress  

---

## 1. Overview

As a user, I want to create an account so that I can save my shopping preferences and access personalised features of the Veggy shopping platform.

---

## 2. Functional Requirements

| # | Requirement |
|---|-------------|
| FR-1 | The application shall provide a registration form accessible from the login/auth page. |
| FR-2 | The registration form shall include fields for: **Full Name**, **Email Address**, and **Password**. |
| FR-3 | The password field shall enforce security requirements: minimum 8 characters, at least one numeric digit, and at least one special character. |
| FR-4 | The system shall display inline, field-level error messages for invalid or missing inputs. |
| FR-5 | The system shall display an error message when the submitted email address is already registered. |
| FR-6 | Upon successful registration, the user shall receive a confirmation email. |
| FR-7 | The registration form shall be accessible from the existing "Sign Up Now!" link on the login page. |
| FR-8 | After successful registration, the user shall be redirected appropriately (e.g., to the login page or home page). |

---

## 3. User Stories

### Story 1 — Access the Registration Form
> **As a** new visitor,  
> **I want to** navigate to a registration form from the login page,  
> **So that** I can create a new account.

**Given** I am on the `/auth` page (login screen),  
**When** I click the "Sign Up Now!" link,  
**Then** I should be taken to a registration form with fields for Full Name, Email, and Password.

---

### Story 2 — Successful Registration
> **As a** new user,  
> **I want to** submit my details and create an account,  
> **So that** I can access the platform's personalised features.

**Given** I am on the registration form,  
**When** I enter a valid Full Name, a valid Email, and a Password that meets security requirements,  
**And** I submit the form,  
**Then** my account should be created,  
**And** I should receive a confirmation email,  
**And** I should be redirected to the login page (or home page).

---

### Story 3 — Password Validation
> **As a** new user,  
> **I want to** be informed when my password does not meet security requirements,  
> **So that** I can correct it before submitting.

**Given** I am on the registration form,  
**When** I enter a password that is fewer than 8 characters, or lacks a number, or lacks a special character,  
**And** I attempt to submit or blur the field,  
**Then** an inline error message should describe the specific requirement that was not met.

---

### Story 4 — Duplicate Email Validation
> **As a** returning user who already has an account,  
> **I want to** be informed that my email is already registered,  
> **So that** I can log in instead of creating a duplicate account.

**Given** I am on the registration form,  
**When** I enter an email address that is already associated with an existing account,  
**And** I submit the form,  
**Then** an error message should be displayed indicating the email is already in use.

---

### Story 5 — Invalid Input Handling
> **As a** user filling in the registration form,  
> **I want to** see clear error messages for any invalid inputs,  
> **So that** I know exactly what to fix.

**Given** I am on the registration form,  
**When** I submit the form with one or more empty or invalid fields,  
**Then** each invalid field should display a descriptive inline error message,  
**And** the form should not be submitted until all fields are valid.

---

## 4. Acceptance Criteria

| # | Criterion | Status |
|---|-----------|--------|
| AC-1 | User can access a registration form with fields for name, email, and password. | Required |
| AC-2 | Password must meet security requirements: minimum 8 characters, includes at least one number and one special character. | Required |
| AC-3 | User receives a confirmation email after successful registration. | Required |
| AC-4 | Error messages are displayed for invalid inputs (e.g., email already in use, weak password, empty fields). | Required |

---

## 5. Assumptions

| # | Assumption |
|---|------------|
| A-1 | The confirmation email is triggered by a backend API call; the frontend is responsible only for initiating the request. |
| A-2 | The "email already in use" error is returned by the backend API as an error response; the frontend displays it. |
| A-3 | The registration route will be `/register` (a new route added to the existing React Router setup). |
| A-4 | No email verification step (e.g., OTP) is required before account activation — a simple confirmation email suffices. |
| A-5 | The existing `AuthProvider` context and `useLocalStorage` hook will be extended to support registration state. |
| A-6 | No third-party authentication (OAuth, SSO) is in scope for this ticket. |
| A-7 | The backend API endpoint for registration is assumed to be `POST /api/auth/register`; actual endpoint to be confirmed with backend team. |

---

## 6. Out of Scope

- Social login / OAuth (Google, Facebook, etc.)
- Email OTP / two-factor authentication
- Account profile management after registration
- Password strength meter UI (beyond inline validation messages)
- Admin-side user management
- CAPTCHA / bot protection
- Terms & Conditions / Privacy Policy acceptance checkbox (unless added as a future AC)
