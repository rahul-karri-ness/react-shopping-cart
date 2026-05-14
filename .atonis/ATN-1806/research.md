# Research: ATN-1806 — Account Creation / User Registration

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| UI Framework | React | ^17.0.2 |
| Routing | react-router-dom | ^5.2.0 |
| Form Management | Formik | ^2.2.6 |
| Validation | Yup | ^0.32.9 |
| HTTP Client | axios | ^0.21.1 |
| Styling | SCSS (node-sass) + styled-components | ^5.0.0 / ^5.2.1 |
| CSS Utilities | classnames | ^2.2.6 |
| State Management | React Context API + useReducer | built-in |
| Persistence | localStorage via custom hook | custom |
| Testing | @testing-library/react + jest-dom | ^11.1.0 / ^5.11.4 |

---

## Relevant Existing Code Modules

### 1. Authentication Context
**File:** `src/contexts/auth.jsx` (lines 1–86)

The existing auth context uses the **Context + useReducer** pattern with two separate contexts for state and dispatch (following the recommended React pattern to avoid unnecessary re-renders).

Key exports:
- `AuthStateContext` — provides `{ isLoggedIn, user, isLoggingIn }`
- `AuthDispatchContext` — provides the dispatch function
- `signIn(dispatch, userData)` — stores user in `localStorage` and dispatches `LOGIN_SUCCESS` (line 47–55)
- `signOut(dispatch)` — clears `localStorage` and dispatches `LOGOUT_SUCCESS` (line 57–62)

**Gap identified:** No `REGISTER_REQUEST`, `REGISTER_SUCCESS`, or `REGISTER_FAILURE` action types exist. These must be added to the reducer (lines 14–45) to support the registration flow.

**User object shape** (inferred from line 69):
```js
{ username: string }
```
Must be extended to include `name` and `email` for registration.

---

### 2. Login Page (Auth Page)
**File:** `src/pages/auth.jsx` (lines 1–90)

- Uses **Formik** with `initialValues`, `validationSchema` (Yup), and `onSubmit`.
- Uses `Field` + custom `Input` component for form fields.
- `goToRegister` handler (line 24–26) is a **stub** — currently does nothing. This is the exact hook point for navigating to `/register`.
- `LoginSchema` (lines 9–12) uses `Yup.string().required()` — the same pattern should be used for the registration schema with additional `.matches()` and `.min()` validators.

---

### 3. Reusable Input Component
**File:** `src/components/core/form-controls/Input.jsx` (lines 1–38)

- Accepts Formik `field` and `form` props (Formik Field component pattern).
- Renders `invalid-feedback` div when `touched[field.name] && errors[field.name]` (line 32–34).
- Applies `is-invalid` CSS class to the `<input>` on error (line 26).
- Supports `label`, `type`, `placeholder`, `className` props.
- **Fully reusable** — no changes needed; registration form can use this directly.

---

### 4. Auth Layout
**File:** `src/layouts/AuthLayout.jsx` (lines 1–23)

- Wraps content in `.auth-container > .wrapper` with centered card styling.
- Contains `.auth-brand` logo section.
- The registration page **must** use this layout to maintain visual consistency.

---

### 5. Route Wrapper
**File:** `src/layouts/RouteWrapper.jsx` (lines 1–36)

- Handles public vs. private route logic via `isPrivate` prop.
- Registration route should be **public** (`isPrivate = false`, which is the default).
- New route must be added to `src/App.js` (lines 24–41).

---

### 6. App Router
**File:** `src/App.js` (lines 1–51)

Current routes:
```
/         → HomePage    (CommonLayout)
/checkout → CheckoutPage (CommonLayout)
/auth     → AuthPage    (AuthLayout)
```
A new route `/register` → `RegisterPage` (AuthLayout) must be added to the `<Switch>` block (after line 40).

---

### 7. SCSS — Auth Styles
**File:** `src/assets/scss/pages/_auth.scss` (lines 1–26)

Existing classes available for reuse:
- `.auth-container` — full-viewport centered flex container
- `.auth-brand` — logo section with bottom margin
- `.auth-button` — full-width, 48px height button with border-radius

The registration page can reuse all these classes without modification.

---

### 8. SCSS — Form Controls
**File:** `src/assets/scss/components/_form-control.scss` (lines 1–32)

- `.form-group` — 16px bottom margin, contains `.invalid-feedback` (red, 14px)
- `.form-control` — full-width input with green focus border
- `.is-invalid` — red border on error state
- `.field-group` — flex row for side-by-side fields (available if needed)

---

### 9. SCSS Variables
**File:** `src/assets/scss/base/_variables.scss` (lines 1–13)

Key variables:
- `$red: #e23d3d` — used for error states
- `$primary-green: #077915` — used for focus/active states
- `$white: #fff` — card background
- `$gray-light-bg: #f5f5f5` — page background

---

### 10. useLocalStorage Hook
**File:** `src/hooks/useLocalStorage.js` (lines 1–28)

- Wraps `window.localStorage` with React state.
- Used in `AuthProvider` (auth.jsx line 65) to persist the user session.
- The registration flow should use the same hook/pattern to persist the newly created user.

---

### 11. Constants
**File:** `src/constants/common.js` (line 1)

- Contains `phoneRegExp` regex.
- A new `passwordRegExp` constant for the security requirements regex should be added here to keep validation patterns centralised.

---

## Patterns to Follow

| Pattern | Where Used | Apply To |
|---------|-----------|----------|
| Formik + Yup validation | `src/pages/auth.jsx` | `RegisterPage` |
| Context + useReducer | `src/contexts/auth.jsx` | Extend auth reducer |
| `Field` + custom `Input` | `src/pages/auth.jsx` | Registration form fields |
| `AuthLayout` wrapper | `src/App.js` | `/register` route |
| `RouteWrapper` with `isPrivate=false` | `src/App.js` | `/register` route |
| `localStorage` persistence | `src/contexts/auth.jsx` | Store registered user |
| SCSS page styles in `_auth.scss` | `src/assets/scss/pages/` | Registration page styles |

---

## External Dependencies

No new npm packages are required. All needed libraries are already installed:
- **Formik** — form state management
- **Yup** — schema validation (password regex, email format, required fields)
- **react-router-dom** — navigation (`useHistory`)
- **classnames** — conditional CSS classes
- **axios** — available for future API integration (email confirmation stub)

---

## Risks and Unknowns

| # | Risk | Severity | Mitigation |
|---|------|----------|------------|
| R-1 | No backend API — email confirmation cannot be truly sent. | Medium | Simulate with a Toast/Alert notification on the frontend. |
| R-2 | No user database — duplicate email check is limited to `localStorage`. | Medium | Check against stored user; document as assumption. |
| R-3 | `App.test.js` (line 4–8) tests for "learn react" text which doesn't exist in the app — existing test is already broken. | Low | Do not regress further; write new tests for registration components only. |
| R-4 | `console.log` debug statement in `src/pages/auth.jsx` line 19 — violates code quality standards. | Low | Note for cleanup; out of scope for this ticket but flagged. |
| R-5 | Password stored in `localStorage` as plain text (existing pattern from `signIn`). | High | For this frontend-only app, document as known limitation. In production, passwords must never be stored client-side. |
| R-6 | `useEffect` dependency array in `auth.jsx` line 75 only includes `state.isLoggedIn` — may cause stale closure warnings. | Low | Follow same pattern for consistency; flag for future refactor. |
