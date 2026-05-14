# Plan: ATN-1806 — Account Creation / User Registration

## Overview

This plan describes the implementation approach for the User Registration feature. It is structured in phases, with explicit guardrail compliance, API contracts, data model changes, and a full list of files to create or modify.

---

## Phases

### Phase 1 — Auth Context Extension

Extend `src/contexts/auth.jsx` to support registration actions and the updated user data shape.

**Changes:**
- Add `isRegistering` flag to `initialState`.
- Add three new action types to the reducer:
  - `REGISTER_REQUEST` — sets `isRegistering: true`
  - `REGISTER_SUCCESS` — sets `isLoggedIn: true`, stores user, sets `isRegistering: false`
  - `REGISTER_FAILURE` — sets `isRegistering: false`
- Export a new `register(dispatch, userData)` action creator that:
  1. Dispatches `REGISTER_REQUEST`
  2. Checks `localStorage` for a duplicate email
  3. On success: stores user via `localStorage`, dispatches `REGISTER_SUCCESS`
  4. On duplicate: dispatches `REGISTER_FAILURE` and throws an error

---

### Phase 2 — Constants Update

Add the password security regex to the shared constants file.

**Changes:**
- Add `passwordRegExp` to `src/constants/common.js`:
  ```js
  // Requires: min 8 chars, at least 1 digit, at least 1 special character
  export const passwordRegExp = /^(?=.*[0-9])(?=.*[!@#$%^&*])[a-zA-Z0-9!@#$%^&*]{8,}$/;
  ```

---

### Phase 3 — Registration Page

Create the new `RegisterPage` component at `src/pages/register.jsx`.

**Structure:**
- Formik form with Yup `RegisterSchema`
- Fields: `name` (text), `email` (email), `password` (password)
- On submit: call `register(authDispatch, { name, email, password })`
- On success: show success toast/alert, auto sign-in, redirect to `/`
- On failure (duplicate email): display server-level error message above the form
- "Already have an account? Login" link navigating to `/auth`

**Yup Schema:**
```js
const RegisterSchema = Yup.object().shape({
  name: Yup.string()
    .min(2, "Name must be at least 2 characters")
    .required("Full name is required"),
  email: Yup.string()
    .email("Please enter a valid email address")
    .required("Email address is required"),
  password: Yup.string()
    .min(8, "Password must be at least 8 characters")
    .matches(passwordRegExp, "Password must contain at least one number and one special character")
    .required("Password is required")
});
```

---

### Phase 4 — Router Update

Register the new route in `src/App.js`.

**Changes:**
- Import `RegisterPage` from `pages/register`
- Add `<RouteWrapper path="/register" component={RegisterPage} layout={AuthLayout} />` inside `<Switch>`

---

### Phase 5 — Login Page Navigation Fix

Wire up the existing stub in `src/pages/auth.jsx`.

**Changes:**
- Replace the no-op `goToRegister` handler with `history.push("/register")`

---

### Phase 6 — SCSS (if needed)

The existing `_auth.scss` classes cover the registration page. No new styles are required unless a password strength indicator is added (out of scope for this ticket).

---

## API Contracts

> **Note:** This is a frontend-only application with no live backend. The following contracts describe the *intended* API shape for future backend integration.

### POST /api/auth/register

**Request Body:**
```json
{
  "name": "string (required, min 2 chars)",
  "email": "string (required, valid email format)",
  "password": "string (required, min 8 chars, 1 digit, 1 special char)"
}
```

**Success Response — 201 Created:**
```json
{
  "user": {
    "id": "string",
    "name": "string",
    "email": "string"
  },
  "message": "Registration successful. Please check your email for confirmation."
}
```

**Error Response — 409 Conflict (duplicate email):**
```json
{
  "error": "EMAIL_ALREADY_IN_USE",
  "message": "An account with this email address already exists."
}
```

**Error Response — 422 Unprocessable Entity (validation failure):**
```json
{
  "error": "VALIDATION_ERROR",
  "fields": {
    "email": "Please enter a valid email address",
    "password": "Password must contain at least one number and one special character"
  }
}
```

---

## Data Model Changes

### User Object (Extended)

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | string | Yes | Full name, min 2 chars |
| `email` | string | Yes | Valid email format; used as unique identifier |
| `password` | string | Yes | Never stored in plain text in production |
| `username` | string | No | Retained for backward compatibility with existing `isLoggedIn` check in `auth.jsx` line 69 |

**Backward Compatibility:** The existing `isLoggedIn` check uses `_get(persistedUser, "username", "").length > 0`. For registered users, `username` should be set to the `email` value to maintain compatibility without breaking the existing login flow.

---

## Files to Create

| File | Purpose |
|------|---------|
| `src/pages/register.jsx` | New registration page component |
| `src/pages/register.test.jsx` | Unit + integration tests for the registration page |

---

## Files to Modify

| File | Change |
|------|--------|
| `src/contexts/auth.jsx` | Add `REGISTER_*` actions, `register()` action creator, `isRegistering` state |
| `src/constants/common.js` | Add `passwordRegExp` |
| `src/App.js` | Add `/register` route |
| `src/pages/auth.jsx` | Wire up `goToRegister` to `history.push("/register")` |

---

## Guardrail Compliance Matrix

*(Based on the Wiki page: "Best Practices for Code Quality, Security, Performance, Accessibility, Testing, Error Handling, and Deployment")*

| Guardrail | Requirement | Compliance Approach |
|-----------|-------------|---------------------|
| **Code Quality** | Follow ESLint rules | All new code follows existing ESLint config (`react-app`); no `console.log` in production code |
| **Code Quality** | Use functional components and React hooks | `RegisterPage` is a functional component; uses `useContext`, `useHistory` hooks |
| **Code Quality** | Modular and reusable components | Reuses existing `Input`, `AuthLayout`, `RouteWrapper`; no duplication |
| **Security** | Sanitize user inputs | Yup schema validates and sanitizes all inputs before processing |
| **Security** | Store sensitive data securely | Password is **not** stored in `localStorage` in the registered user object (only `name`, `email`, `username`) |
| **Performance** | Minimize API calls | No unnecessary API calls; registration is a single action |
| **Accessibility** | Keyboard-navigable interactive elements | All form fields and buttons are native HTML elements, keyboard-navigable by default |
| **Accessibility** | ARIA attributes | `aria-label` and `aria-describedby` added to form fields and error messages |
| **Accessibility** | Alternative text for images | Existing logo `alt` text in `AuthLayout` is already compliant |
| **Testing** | Unit tests for all components and hooks | `register.test.jsx` covers all happy paths, negative paths, and edge cases |
| **Testing** | Integration tests for critical user flows | Registration form submission flow tested end-to-end with `@testing-library/react` |
| **Error Handling** | User-friendly error messages | Inline Yup validation messages + form-level error for duplicate email |
| **Error Handling** | Log errors to monitoring service | `console.error` used in catch blocks (Sentry integration is out of scope) |

---

## Architectural Decisions

### Decision 1: Extend AuthContext vs. Create a New RegistrationContext
**Chosen:** Extend `AuthContext`  
**Rationale:** Registration is part of the authentication lifecycle. The existing `AuthProvider` already manages user state and `localStorage` persistence. Creating a separate context would introduce unnecessary complexity and duplication.

### Decision 2: Auto Sign-In After Registration vs. Redirect to Login
**Chosen:** Auto sign-in + redirect to `/`  
**Rationale:** Better UX — the user has just provided their credentials; requiring them to log in again is friction. Consistent with the `signIn` action already available in `auth.jsx`.

### Decision 3: Client-Side Duplicate Email Check vs. API Call
**Chosen:** Client-side check against `localStorage`  
**Rationale:** No backend API exists. The check is clearly documented as a frontend-only limitation. The code is structured so the `register()` action creator can be swapped for an API call with minimal changes.

### Decision 4: Password Not Stored in localStorage
**Chosen:** Strip password from the persisted user object  
**Rationale:** Even in a demo app, storing passwords in `localStorage` is a security anti-pattern. The user object stored will contain only `{ name, email, username }`.
