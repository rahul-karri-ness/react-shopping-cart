# ATN-1806 - Account Creation - User Registration
## Implementation Plan

**Ticket:** ATN-1806

---

## Overview

This plan describes the implementation of the user registration feature for the react-shopping-cart application. The feature adds a `/register` route with a Formik-based registration form, extends the auth context with registration actions, persists registered users to localStorage, and wires the existing no-op `goToRegister` stub in the login page to navigate to the new route.

No new npm dependencies are required.

---

## Implementation Phases

### Phase 1 - Add passwordRegExp Constant

**File:** `src/constants/common.js`

Add a `passwordRegExp` constant that enforces: minimum 8 characters, at least one digit, at least one special character.

```js
export const passwordRegExp = /^(?=.*[0-9])(?=.*[!@#$%^&*()_+\-=[\]{};':"\\|,.<>/?]).{8,}$/;
```

This follows the existing pattern of `phoneRegExp` exported from the same file.

---

### Phase 2 - Extend Auth Context with Registration Actions

**File:** `src/contexts/auth.jsx`

**2a - Extend initialState:**
Add `isRegistering: false` to `initialState`.

**2b - Extend reducer:**
Add three new cases to the existing `switch` statement:

```js
case "REGISTER_REQUEST":
  return { ...state, isRegistering: true };
case "REGISTER_SUCCESS":
  return { ...state, isRegistering: false };
case "REGISTER_FAILURE":
  return { ...state, isRegistering: false };
```

**2c - Add register action creator:**
Export a new `register(dispatch, userData)` function:

```js
export const register = (dispatch, userData) => {
  dispatch({ type: "REGISTER_REQUEST" });
  const existing = JSON.parse(localStorage.getItem("registeredUsers") || "[]");
  const duplicate = existing.some(
    (u) => u.email.toLowerCase() === userData.email.toLowerCase()
  );
  if (duplicate) {
    dispatch({ type: "REGISTER_FAILURE" });
    return { success: false, error: "Email is already registered" };
  }
  const updated = [...existing, userData];
  localStorage.setItem("registeredUsers", JSON.stringify(updated));
  dispatch({ type: "REGISTER_SUCCESS" });
  return { success: true };
};
```

---

### Phase 3 - Create RegisterPage Component

**File:** `src/pages/register.jsx`

Create a new page component using the established Formik + Yup + Field + Input pattern from `src/pages/auth.jsx` and `src/pages/checkout.jsx`.

Key elements:
- Import `passwordRegExp` from `constants/common`
- Import `register` and `AuthDispatchContext` from `contexts/auth`
- Define `RegisterSchema` using Yup with rules for `fullName`, `email`, and `password`
- On submit: call `register(authDispatch, values)`
  - If `success: false`: call `setFieldError("email", result.error)`
  - If `success: true`: show success message, then `setTimeout(() => history.push("/auth"), 2000)`
- Render a link back to `/auth` for users who already have an account

---

### Phase 4 - Add /register Route in App.js

**File:** `src/App.js`

Add a new `RouteWrapper` entry inside the `Switch` block, after the `/auth` route:

```jsx
import RegisterPage from "pages/register";

<RouteWrapper
  path="/register"
  component={RegisterPage}
  layout={AuthLayout}
/>
```

---

### Phase 5 - Wire goToRegister Navigation Stub

**File:** `src/pages/auth.jsx`

Replace the no-op body of `goToRegister` with a `history.push` call:

```jsx
const goToRegister = (e) => {
  e.preventDefault();
  history.push("/register");
};
```

---

### Phase 6 - Add Register SCSS

**File:** `src/assets/scss/pages/_register.scss`

Create a new SCSS file following the same pattern as `_auth.scss`. The `.auth-container`, `.auth-brand`, and `.auth-button` classes defined in `_auth.scss` are reused. Add only register-specific overrides if needed.

**File:** `src/assets/scss/pages/_index.scss`

Register the new file by adding an import/forward line.

---

## API Contracts

### register(dispatch, userData) â€” Return Shape

```js
// Success
{ success: true }

// Failure
{ success: false, error: "Email is already registered" }
```

### localStorage Schema

| Key               | Type    | Shape                                                        |
|-------------------|---------|--------------------------------------------------------------|
| `registeredUsers` | Array   | `[{ fullName: string, email: string, password: string }]`   |
| `user`            | Object  | `{ username: string, ... }` (existing â€” unchanged)          |

---

## Files to Create

| File                                   | Type     |
|----------------------------------------|----------|
| `src/pages/register.jsx`               | New      |
| `src/pages/register.test.jsx`          | New      |
| `src/assets/scss/pages/_register.scss` | New      |

## Files to Modify

| File                                        | Change Summary                                              |
|---------------------------------------------|-------------------------------------------------------------|
| `src/constants/common.js`                   | Add `passwordRegExp` export                                 |
| `src/contexts/auth.jsx`                     | Add `isRegistering` to state, REGISTER_* reducer cases, `register` action creator |
| `src/App.js`                                | Add `/register` RouteWrapper                                |
| `src/pages/auth.jsx`                        | Wire `goToRegister` to `history.push("/register")`          |
| `src/assets/scss/pages/_index.scss`         | Import `_register.scss`                                     |

---

## Guardrail Compliance Matrix

| Guardrail                                                      | Status    | How Addressed                                                                                   |
|----------------------------------------------------------------|-----------|-------------------------------------------------------------------------------------------------|
| No hardcoded secrets or API keys                               | Compliant | No secrets used. localStorage only stores user-provided data.                                   |
| No new npm dependencies without justification                  | Compliant | Zero new dependencies. Formik, Yup, React Router already present.                              |
| Follow existing code patterns and architecture                 | Compliant | Formik + Yup + Field + Input pattern mirrors auth.jsx and checkout.jsx exactly.                 |
| Errors handled gracefully with meaningful user-facing messages | Compliant | Yup inline errors per field; duplicate email error set via `setFieldError`.                     |
| No dead code, debug artifacts, or commented-out blocks         | Compliant | All code is production-ready; no console.log left in final implementation.                      |
| Unit tests for happy path, negative paths, and edge cases      | Compliant | register.test.jsx covers: successful registration, duplicate email, invalid password, empty fields, case-insensitive duplicate. |
| SOLID principles and single responsibility                     | Compliant | `register` action creator handles only persistence logic. RegisterPage handles only UI.         |
| Accessibility: labels associated with inputs                   | Compliant | Existing `Input` component renders `<label htmlFor={field.name}>` when `label` prop is provided.|
| No regression to existing routes or auth flow                  | Compliant | Existing `/auth`, `/`, `/checkout` routes unchanged. Only `goToRegister` stub is wired.        |
| Password security requirements enforced                        | Compliant | `passwordRegExp` enforces min 8 chars, digit, special character via Yup.                       |
| Case-insensitive duplicate email check                         | Compliant | `.toLowerCase()` normalisation applied on both sides of comparison.                             |
| Redirect after success with delay                              | Compliant | `setTimeout(() => history.push("/auth"), 2000)` after successful registration.                 |

---

## Data Model Changes

No database or API schema changes. The only data model change is the addition of the `registeredUsers` key in localStorage (browser-local, no server impact).

---

## Out of Scope

- Real backend API or server-side registration endpoint
- Email delivery integration
- Password hashing (localStorage is client-only and not a production auth store)
- Account management features
