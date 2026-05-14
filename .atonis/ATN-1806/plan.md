# Plan: ATN-1806 — Account Creation / User Registration

## Overview

This plan describes the implementation approach for the User Registration feature in the React Shopping Cart application. It covers all phases, API contracts, data model changes, files to create/modify, and explicit compliance with project guardrails.

---

## Implementation Phases

### Phase 1 — Auth Context Extension
Extend `src/contexts/auth.jsx` to support registration state and actions.

**Changes:**
- Add `isRegistering` and `registrationSuccess` to `initialState`.
- Add `REGISTER_REQUEST`, `REGISTER_SUCCESS`, `REGISTER_FAILURE` cases to the reducer.
- Add `register(dispatch, userData, registeredUsers)` action creator that:
  1. Checks if email already exists in `registeredUsers` array (from `localStorage`).
  2. If duplicate: dispatches `REGISTER_FAILURE` with error payload.
  3. If unique: dispatches `REGISTER_SUCCESS`, stores new user in `registeredUsers` array in `localStorage`.
- Simulates confirmation email by logging to console: `console.info("Confirmation email sent to:", email)`.

---

### Phase 2 — Constants Update
Add password validation regex to `src/constants/common.js`.

**Changes:**
- Export `passwordRegExp` for use in Yup schema.

---

### Phase 3 — Registration Form Component
Create `src/components/RegisterForm.jsx` — a self-contained Formik registration form.

**Fields:**
| Field | Type | Validation |
|-------|------|-----------|
| name | text | Required, min 2 chars |
| email | email | Required, valid email format |
| password | password | Required, ≥8 chars, ≥1 digit, ≥1 special char |
| confirmPassword | password | Required, must match `password` |

**Yup Schema (`RegisterSchema`):**
```js
Yup.object().shape({
  name: Yup.string().min(2, "Name must be at least 2 characters.").required("Full name is required."),
  email: Yup.string().email("Please enter a valid email address.").required("Email is required."),
  password: Yup.string()
    .min(8, "Password must be at least 8 characters.")
    .matches(/[0-9]/, "Password must contain at least one number.")
    .matches(/[!@#$%^&*]/, "Password must contain at least one special character (!@#$%^&*).")
    .required("Password is required."),
  confirmPassword: Yup.string()
    .oneOf([Yup.ref("password"), null], "Passwords do not match.")
    .required("Please confirm your password.")
})
```

**On Submit:**
- Call `register(authDispatch, { name, email, password }, registeredUsers)`.
- On success: show inline success message (no redirect; user must log in separately).
- On failure (duplicate email): show field-level error via `setFieldError("email", "This email address is already in use.")`.

---

### Phase 4 — Auth Page Update
Update `src/pages/auth.jsx` to support view toggling between Login and Register.

**Changes:**
- Add `const [view, setView] = React.useState("login")` local state.
- Implement `goToRegister`: `setView("register")`.
- Add `goToLogin`: `setView("login")`.
- Conditionally render `<RegisterForm onSwitchToLogin={goToLogin} />` or the existing Login Formik form based on `view`.
- Remove `console.log("location => ", location)` debug statement (line 19).

---

### Phase 5 — SCSS Updates
Update `src/assets/scss/pages/_auth.scss` to add styles for:
- `.auth-success` — success message styling (green background, icon).
- `.auth-toggle-link` — consistent link styling for view toggle.

---

### Phase 6 — Tests
Write unit tests for all new and modified components/modules (TDD order — tests first).

**Test files to create:**
- `src/contexts/auth.test.js` — tests for reducer and action creators including registration.
- `src/components/RegisterForm.test.jsx` — tests for form rendering, validation, submission.
- `src/pages/auth.test.jsx` — tests for view toggling between login and register.

---

## API Contracts

> **Note:** No real backend API exists. The following describes the simulated contract for future backend integration.

### POST /api/auth/register (Future)
**Request Body:**
```json
{
  "name": "string",
  "email": "string",
  "password": "string"
}
```

**Success Response (201):**
```json
{
  "message": "Registration successful. Please check your email to confirm your account.",
  "userId": "string"
}
```

**Error Response (409 — Duplicate Email):**
```json
{
  "error": "EMAIL_ALREADY_IN_USE",
  "message": "This email address is already in use."
}
```

**Error Response (400 — Validation):**
```json
{
  "error": "VALIDATION_ERROR",
  "fields": {
    "email": "Please enter a valid email address.",
    "password": "Password must be at least 8 characters."
  }
}
```

> **Current Implementation:** All of the above is simulated client-side using `localStorage`.

---

## Data Model Changes

### Registered Users (localStorage key: `"registeredUsers"`)
```js
// Array of user objects
[
  {
    id: "uuid-or-timestamp",   // unique identifier
    name: "string",
    email: "string",           // used as unique key
    password: "string",        // NOTE: plain text — acceptable for mock only; must be hashed in production
    createdAt: "ISO-8601"
  }
]
```

### Auth State Extension (`src/contexts/auth.jsx`)
```js
const initialState = {
  isLoggedIn: false,
  user: null,
  isLoggingIn: false,
  isRegistering: false,       // NEW
  registrationSuccess: false, // NEW
  registrationError: null     // NEW
};
```

---

## Files to Create

| File | Purpose |
|------|---------|
| `src/components/RegisterForm.jsx` | New registration form component |
| `src/contexts/auth.test.js` | Unit tests for auth context (reducer + action creators) |
| `src/components/RegisterForm.test.jsx` | Unit tests for RegisterForm component |
| `src/pages/auth.test.jsx` | Unit tests for AuthPage view toggling |

---

## Files to Modify

| File | Change Summary |
|------|---------------|
| `src/contexts/auth.jsx` | Add registration state, reducer cases, and `register` action creator |
| `src/pages/auth.jsx` | Add view toggle, wire RegisterForm, remove debug log |
| `src/constants/common.js` | Add `passwordRegExp` export |
| `src/assets/scss/pages/_auth.scss` | Add `.auth-success` and `.auth-toggle-link` styles |

---

## Guardrail Compliance Matrix

| Guardrail | Source | Compliance Approach |
|-----------|--------|-------------------|
| Follow consistent coding standards (ESLint) | Best Practices Doc | All new code follows existing ESLint config (`react-app`); no new rules introduced |
| Use functional components and React hooks | Best Practices Doc | `RegisterForm` is a functional component; `useState` used for view toggle |
| Ensure all components are modular and reusable | Best Practices Doc | `RegisterForm` is extracted as a standalone component, reusable independently |
| Sanitize user inputs to prevent XSS | Best Practices Doc | Yup validation sanitizes/validates all inputs; React's JSX escapes output by default |
| Store sensitive data securely | Best Practices Doc | Documented that plain-text password in `localStorage` is mock-only; production must hash |
| Write unit tests for all components and hooks | Best Practices Doc | Tests created for `auth.jsx` context, `RegisterForm`, and `AuthPage` |
| Implement integration tests for critical user flows | Best Practices Doc | Auth page test covers the full registration flow end-to-end |
| Display user-friendly error messages | Best Practices Doc | All Yup errors and duplicate-email errors are user-friendly inline messages |
| Log errors to monitoring service | Best Practices Doc | `console.error` used for caught errors; noted that Sentry integration is future work |
| Use ARIA attributes for accessibility | Best Practices Doc | `Input` component uses `htmlFor`/`id` pairing; form labels included |
| Ensure keyboard navigability | Best Practices Doc | All form fields and buttons are native HTML elements — keyboard navigable by default |
| No dead code / debug artifacts | Definition of Done | `console.log` in `auth.jsx` line 19 removed |
| No secrets or API keys hardcoded | Definition of Done | No secrets; `localStorage` keys are non-sensitive |
| Errors handled gracefully | Definition of Done | `try/catch` in form `onSubmit`; field-level errors via `setFieldError` |
| All tests pass | Definition of Done | Stale `App.test.js` test updated to reflect actual app content |
| No regression introduced | Definition of Done | Login flow unchanged; only additive changes to auth context and page |
| New code follows existing architectural patterns | Definition of Done | Split context pattern, Formik+Yup, action creator pattern all followed |

---

## Key Architectural Decisions

### Decision 1: Toggle View vs. Separate Route
**Options considered:**
- A) Toggle between Login/Register within `/auth` using local `useState`
- B) Add a separate `/register` route

**Decision:** Option A — Toggle within `/auth`.  
**Rationale:** The existing `goToRegister` stub in `auth.jsx` (line 24) already implies this pattern. No new route, no changes to `App.js`, minimal surface area change.

---

### Decision 2: RegisterForm as Separate Component vs. Inline JSX
**Options considered:**
- A) Inline JSX in `auth.jsx`
- B) Separate `RegisterForm.jsx` component

**Decision:** Option B — Separate component.  
**Rationale:** Follows the modularity guardrail; makes the component independently testable; keeps `auth.jsx` readable.

---

### Decision 3: Registered Users Storage
**Options considered:**
- A) Store in same `"user"` localStorage key
- B) Store in separate `"registeredUsers"` array key

**Decision:** Option B — Separate `"registeredUsers"` key.  
**Rationale:** Keeps the logged-in user state separate from the registry of all registered users. Avoids data collision.

---

### Decision 4: Auto-login After Registration
**Decision:** No auto-login.  
**Rationale:** AC-5 requires a confirmation email step. Auto-login would bypass this. User must log in after registration.
