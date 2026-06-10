# ATN-1806 — Account Creation - User Registration
## Research

---

### Tech Stack

| Technology | Version | Role |
|------------|---------|------|
| React | 17.0.2 | UI framework — functional components + hooks |
| Formik | 2.2.6 | Form state management and submission handling |
| Yup | 0.32.9 | Schema-based form validation |
| React Router DOM | 5.2.0 | Client-side routing (BrowserRouter, Switch, Route) |
| Context API + useReducer | React 17 built-in | Global auth state management |
| SCSS (node-sass 5.0.0) | — | Component and page styling |
| classnames | 2.2.6 | Conditional CSS class composition |
| @testing-library/react | 11.1.0 | Component unit and integration testing |
| @testing-library/user-event | 12.1.10 | Simulated user interactions in tests |

No new npm dependencies are required for this feature.

---

### Relevant Existing Files

#### Files to Modify

| File | Location | Change Required |
|------|----------|-----------------|
| `auth.jsx` (page) | `src/pages/auth.jsx` | Wire `goToRegister` (line 24) — currently a no-op stub — to `history.push("/register")`. |
| `auth.jsx` (context) | `src/contexts/auth.jsx` | Add `REGISTER_REQUEST`, `REGISTER_SUCCESS`, `REGISTER_FAILURE` action cases to the reducer (lines 14–44). Add exported `register` action creator. |
| `App.js` | `src/App.js` | Add `/register` `RouteWrapper` using `AuthLayout` and the new `RegisterPage` component (after line 43). |
| `common.js` | `src/constants/common.js` | Add `passwordRegExp` constant alongside existing `phoneRegExp` (line 1). |
| `_index.scss` (pages) | `src/assets/scss/pages/_index.scss` | Add `@import "register"` alongside existing imports (line 3). |

#### Files to Create

| File | Location | Purpose |
|------|----------|---------|
| `RegisterPage` component | `src/pages/register.jsx` | New registration form page using Formik + Yup + Field + Input pattern. |
| `_register.scss` | `src/assets/scss/pages/_register.scss` | Page-specific styles for the registration form. |
| `RegisterPage.test.jsx` | `src/pages/register.test.jsx` | Unit and integration tests for the RegisterPage component. |

---

### Key Code Patterns to Follow

#### Pattern 1 — Formik + Yup + Field + Input (from `src/pages/checkout.jsx`)

The existing checkout page (lines 19–31) defines a Yup schema and wires it to Formik via `validationSchema`. Fields use the `Field` component with `component={Input}`, which renders the reusable `Input` core component.

```jsx
// src/pages/checkout.jsx — lines 19–31
const AddressSchema = Yup.object().shape({
  fullName: Yup.string().required("Full Name is required"),
  phoneNumber: Yup.string()
    .required("Phone Number is required")
    .matches(phoneRegExp, "Phone Number is not a valid 10 digit number")
    ...
});
```

The `RegisterPage` must follow this exact pattern.

#### Pattern 2 — Input Core Component (from `src/components/core/form-controls/Input.jsx`)

The `Input` component (lines 1–36) accepts `field` and `form` props injected by Formik's `Field`. It renders `invalid-feedback` for validation errors when `touched[field.name] && errors[field.name]`. The `RegisterPage` must use this component for all fields.

#### Pattern 3 — Auth Context Reducer (from `src/contexts/auth.jsx` lines 14–44)

The reducer handles `LOGIN_REQUEST`, `LOGIN_SUCCESS`, `LOGIN_FAILURE`, and `LOGOUT_SUCCESS`. New `REGISTER_REQUEST`, `REGISTER_SUCCESS`, and `REGISTER_FAILURE` cases must be added following the same shape. The `register` action creator must mirror the `signIn` pattern (lines 47–55): write to `localStorage` and dispatch the success action.

```js
// src/contexts/auth.jsx — lines 47–55 (signIn pattern to mirror)
export const signIn = (dispatch, userData) => {
  localStorage.setItem("user", JSON.stringify(userData));
  return dispatch({ type: "LOGIN_SUCCESS", payload: { user: userData } });
};
```

The `register` action creator will write to `localStorage` key `registeredUsers` (an array) and dispatch `REGISTER_SUCCESS`.

#### Pattern 4 — RouteWrapper with AuthLayout (from `src/App.js` lines 38–43)

```jsx
// src/App.js — lines 38–43
<RouteWrapper
  path="/auth"
  component={AuthPage}
  layout={AuthLayout}
/>
```

The `/register` route must follow this exact pattern with `layout={AuthLayout}`.

#### Pattern 5 — Constants (from `src/constants/common.js` line 1)

```js
export const phoneRegExp = /^((\\+[1-9]{1,4}[ \\-]*)...$/;
```

The `passwordRegExp` constant must be exported from the same file.

#### Pattern 6 — useHistory for navigation (from `src/pages/auth.jsx` lines 3, 16–17)

Navigation uses `useHistory` from `react-router-dom`. The `goToRegister` stub at line 24 must call `history.push("/register")`.

---

### Key Finding — goToRegister Stub

In `src/pages/auth.jsx` at line 24, `goToRegister` is currently a no-op:

```jsx
const goToRegister = (e) => {
  e.preventDefault();
  // missing: history.push("/register")
};
```

This function must be wired to `history.push("/register")` to satisfy FR-1 and Story 1.

---

### Auth State Shape

Current `initialState` in `src/contexts/auth.jsx` (lines 4–8):

```js
const initialState = {
  isLoggedIn: false,
  user: null,
  isLoggingIn: false
};
```

New fields to add for registration flow:

```js
isRegistering: false,   // true during REGISTER_REQUEST
registrationError: null // error message string or null
```

---

### localStorage Data Model

| Key | Shape | Purpose |
|-----|-------|---------|
| `user` | `{ username, password }` | Existing — current logged-in user |
| `registeredUsers` | `Array<{ fullName, email, password }>` | New — persisted registered user list |

The `register` action creator reads `registeredUsers` from `localStorage`, checks for duplicate email (case-insensitive), appends the new user, and writes back.

---

### Risks and Unknowns

| Risk | Severity | Mitigation |
|------|----------|------------|
| Passwords stored in `localStorage` in plain text | Medium | Acceptable per A-1 (no real backend). Document clearly in code comments. Do NOT log password values. |
| `localStorage` unavailable (private browsing, storage full) | Low | `useLocalStorage` hook already wraps access in try/catch (lines 4–10 of `src/hooks/useLocalStorage.js`). Use same pattern. |
| Redirect timing (2-second delay) conflicts with component unmount | Low | Clear `setTimeout` in a `useEffect` cleanup function to prevent memory leaks. |
| `console.log` in `src/pages/auth.jsx` line 19 | Low | Existing debug artifact — do not replicate in new code. |
