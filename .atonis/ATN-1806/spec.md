# ATN-1806 - Account Creation - User Registration
## Spec

**Ticket:** ATN-1806
**Type:** Story
**Priority:** Major
**Status:** In Progress

---

## User Story

As a user, I want to create an account so that I can save my shopping preferences.

---

## Functional Requirements

| ID   | Requirement                                                                                                                                             | Priority |
|------|---------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| FR-1 | The user can navigate to a registration form from the login page.                                                                                       | Must     |
| FR-2 | The registration form contains fields for full name, email address, and password.                                                                       | Must     |
| FR-3 | The password must satisfy security requirements: minimum 8 characters, at least one numeric digit, and at least one special character.                  | Must     |
| FR-4 | When a user submits valid registration data, a success message is displayed simulating a confirmation email, and the user is redirected to the login page after 2 seconds. | Must     |
| FR-5 | When a user submits invalid or duplicate data, specific inline error messages are displayed for each invalid field without clearing the entire form.     | Must     |

---

## User Stories (Given / When / Then)

### US-1: Access the Registration Form
**Given** the user is on the login page at `/auth`
**When** the user clicks the "Register" link
**Then** the user is navigated to the registration page at `/register`
**And** the registration form is displayed with fields for full name, email address, and password

---

### US-2: Successful Registration
**Given** the user is on the registration page at `/register`
**When** the user enters a valid full name (minimum 2 characters), a valid email address not already registered, and a password meeting security requirements
**And** the user submits the form
**Then** a success message is displayed indicating that registration is complete and a confirmation email has been sent
**And** the user is automatically redirected to the login page at `/auth` after 2 seconds

---

### US-3: Password Validation Failure
**Given** the user is on the registration page at `/register`
**When** the user enters a password that does not meet security requirements (fewer than 8 characters, no digit, or no special character)
**And** the user submits the form or moves focus away from the password field
**Then** an inline error message is displayed beneath the password field describing the specific requirement not met
**And** the form is not submitted

---

### US-4: Duplicate or Invalid Email
**Given** the user is on the registration page at `/register`
**When** the user enters an email address that is already registered (case-insensitive match) or an email address in an invalid format
**And** the user submits the form
**Then** an inline error message is displayed beneath the email field indicating the specific issue
**And** the form is not submitted

---

### US-5: Empty Required Fields
**Given** the user is on the registration page at `/register`
**When** the user submits the form without filling in one or more required fields
**Then** inline error messages are displayed beneath each empty required field
**And** the form is not submitted

---

## Acceptance Criteria

| ID   | Criterion                                                                                                                                  | Priority |
|------|--------------------------------------------------------------------------------------------------------------------------------------------|----------|
| AC-1 | The registration form is accessible at the route `/register` and is reachable via a link on the `/auth` login page.                        | Must     |
| AC-2 | The form contains three fields: full name, email address, and password, all of which are required.                                         | Must     |
| AC-3 | Password validation enforces: minimum 8 characters, at least one digit, and at least one special character.                               | Must     |
| AC-4 | On successful registration, a success message is shown and the user is redirected to `/auth` after 2 seconds.                             | Must     |
| AC-5 | If the submitted email already exists in localStorage (case-insensitive), an error message "Email is already registered" is displayed.    | Must     |
| AC-6 | All validation errors are displayed inline beneath their respective fields without clearing other field values.                            | Must     |

---

## Assumptions

| ID  | Assumption                                                                                                                 |
|-----|----------------------------------------------------------------------------------------------------------------------------|
| A-1 | No real backend API exists. Registration data is persisted to localStorage under the key `registeredUsers`.                |
| A-2 | Confirmation email is simulated via a UI success message only; no actual email is sent.                                    |
| A-3 | Duplicate email detection is case-insensitive.                                                                             |
| A-4 | The registration page route is `/register`.                                                                                |
| A-5 | On successful registration, the user is redirected to `/auth` after a 2-second delay.                                     |
| A-6 | The full name field requires a minimum of 2 characters.                                                                    |

---

## Out of Scope

- Real email delivery or SMTP integration
- Email verification / account activation links
- OAuth or third-party authentication (Google, Facebook, etc.)
- Password strength meter UI component
- CAPTCHA or bot-prevention mechanisms
- Account management (edit profile, change password, delete account)
- Backend API integration or server-side validation
- Password confirmation / re-enter password field
