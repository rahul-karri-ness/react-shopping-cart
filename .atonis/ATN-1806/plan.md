# ATN-1806 — Account Creation - User Registration
## Implementation Plan

---

### Overview

This plan describes the implementation of the user registration feature in six sequential phases. All phases follow the existing architectural patterns of the codebase: functional components, React hooks, Context API + useReducer, Formik + Yup for forms, and SCSS for styling. No new npm dependencies are required.

---

### Phase 1 — Add passwordRegExp Constant

**File:** `src/constants/common.js`

Add a `passwordRegExp` export alongside the existing `phoneRegExp`. The regex enforces: minimum 8 characters, at least one digit, and at least one special character.

```js
// Enforces: min 8 chars, at least 1 digit, at least 1 special character
export const passwordRegExp = /^(?=.*[0-9])(?=.*[!@#$%^&*()_+\-=[\]{};':"\\|,.<>/?]).{8,}$/;
```

This constant is imported by the Yup schema in Phase 3.

---

### Phase 2 — Extend Auth Context with Registration Actions

**File:** `src/contexts/auth.jsx`

#### 2a — Extend initialState

Add two new fields to `initialState`:

```js
const initialState = {
  isLoggedIn: false,
  user: null,
  isLoggingIn: false,
  isRegistering: false,      // true while registration is in progress
  registrationError: null    // holds error message string on failure
};
```

#### 2b — Add Reducer Cases

Add three new cases to the reducer switch statement, following the existing LOGIN pattern:

```js
case "REGISTER_REQUEST":
  return { ...state, isRegistering: true, registrationError: null };

case "REGISTER_SUCCESS":
  return { ...state, isRegistering: false, registrationError: null };

case "REGISTER_FAILURE":
  return { ...state, isRegistering: false, registrationError: action.payload.error };
```

#### 2c — Add register Action Creator

Export a `register` function that:
1. Reads `registeredUsers` from `localStorage` (defaults to `[]`).
2. Checks for duplicate email (case-insensitive).
3. On duplicate: dispatches `REGISTER_FAILURE` and returns `{ success: false, error }`.
4. On success: appends the new user to the array, writes back to `localStorage`, dispatches `REGISTER_SUCCESS`, and returns `{ success: true }`.

```js
export const register = (dispatch, userData) => {
  dispatch({ type: "REGISTER_REQUEST" });
  try {
    // Read existing registered users from localStorage (no real backend per A-1)
    const existing = JSON.parse(localStorage.getItem("registeredUsers") || "[]");
    const isDuplicate = existing.some(
      (u) => u.email.toLowerCase() === userData.email.toLowerCase()
    );
    if (isDuplicate) {
      const error = "An account with this email address already exists.";
      dispatch({ type: "REGISTER_FAILURE", payload: { error } });
      return { success: false, error };
    }
    const updated = [...existing, userData];
    localStorage.setItem("registeredUsers", JSON.stringify(updated));
    dispatch({ type: "REGISTER_SUCCESS" });
    return { success: true };
  } catch (err) {
    const error = "Registration failed. Please try again.";
    dispatch({ type: "REGISTER_FAILURE", payload: { error } });
    return { success: false, error };
  }
};
```

---

### Phase 3 — Create RegisterPage Component

**File:** `src/pages/register.jsx`

The component follows the exact Formik + Yup + Field + Input pattern from `src/pages/checkout.jsx` and `src/pages/auth.jsx`.

Key behaviours:
- Yup schema (`RegisterSchema`) validates `fullName` (min 2 chars), `email` (valid format), and `password` (matches `passwordRegExp`).
- On `onSubmit`: calls `register(authDispatch, { fullName, email, password })`.
- On `{ success: true }`: sets local `registrationSuccess` state to `true`, displays a success message, and schedules `history.push("/auth")` after 2000 ms via `setTimeout` cleaned up in `useEffect`.
- On `{ success: false }`: sets a `formError` state string displayed as a form-level error above the submit button.
- Uses `useContext(AuthDispatchContext)` for dispatch.
- Uses `useHistory` for navigation.
- All interactive elements include appropriate `aria-label` attributes (guardrail: accessibility).
- No `console.log` calls — errors handled via `console.error` only (guardrail: no debug artifacts).

```jsx
import React, { useState, useEffect, useContext } from "react";
import { Formik, Form, Field } from "formik";
import { useHistory } from "react-router-dom";
import * as Yup from "yup";
import { AuthDispatchContext, register } from "contexts/auth";
import Input from "components/core/form-controls/Input";
import { passwordRegExp } from "constants/common";

const RegisterSchema = Yup.object().shape({
  fullName: Yup.string()
    .min(2, "Full name must be at least 2 characters.")
    .required("Full name is required."),
  email: Yup.string()
    .email("Please enter a valid email address.")
    .required("Email address is required."),
  password: Yup.string()
    .matches(
      passwordRegExp,
      "Password must be at least 8 characters and include a number and a special character."
    )
    .required("Password is required.")
});

const RegisterPage = () => {
  const authDispatch = useContext(AuthDispatchContext);
  const history = useHistory();
  const [registrationSuccess, setRegistrationSuccess] = useState(false);
  const [formError, setFormError] = useState(null);

  useEffect(() => {
    let timer;
    if (registrationSuccess) {
      // Redirect to /auth after 2 seconds (A-5). Timer cleared on unmount.
      timer = setTimeout(() => history.push("/auth"), 2000);
    }
    return () => clearTimeout(timer);
  }, [registrationSuccess, history]);

  if (registrationSuccess) {
    return (
      <div className="register-success" role="alert" aria-live="polite">
        <p>Registration successful! A confirmation has been noted.</p>
        <p>Redirecting you to the login page...</p>
      </div>
    );
  }

  return (
    <Formik
      initialValues={{ fullName: "", email: "", password: "" }}
      validationSchema={RegisterSchema}
      onSubmit={async (values, { resetForm }) => {
        try {
          setFormError(null);
          const result = register(authDispatch, {
            fullName: values.fullName,
            email: values.email,
            password: values.password
          });
          if (result.success) {
            resetForm();
            setRegistrationSuccess(true);
          } else {
            setFormError(result.error);
          }
        } catch (err) {
          console.error("Registration error:", err);
          setFormError("An unexpected error occurred. Please try again.");
        }
      }}
    >
      {() => (
        <Form noValidate aria-label="Registration form">
          <Field
            name="fullName"
            type="text"
            placeholder="Full Name"
            label="Full Name"
            component={Input}
          />
          <Field
            name="email"
            type="email"
            placeholder="Email Address"
            label="Email Address"
            component={Input}
          />
          <Field
            name="password"
            type="password"
            placeholder="Password"
            label="Password"
            component={Input}
          />
          {formError && (
            <div className="form-error" role="alert" aria-live="assertive">
              {formError}
            </div>
          )}
          <button
            type="submit"
            className="auth-button block"
            aria-label="Create account"
          >
            Create Account
          </button>
          <p>
            Already have an account?{" "}
            <a href="/auth">Sign In</a>
          </p>
        </Form>
      )}
    </Formik>
  );
};

export default RegisterPage;
```

---

### Phase 4 — Add /register Route in App.js

**File:** `src/App.js`

Add import for `RegisterPage` and add a `RouteWrapper` entry following the existing `/auth` pattern:

```jsx
import RegisterPage from "pages/register";

// Inside <Switch>:
<RouteWrapper
  path="/register"
  component={RegisterPage}
  layout={AuthLayout}
/>
```

---

### Phase 5 — Wire goToRegister Navigation

**File:** `src/pages/auth.jsx`

Replace the no-op `goToRegister` stub (line 24) with a navigation call:

```jsx
const goToRegister = (e) => {
  e.preventDefault();
  history.push("/register");
};
```

---

### Phase 6 — Create _register.scss

**File:** `src/assets/scss/pages/_register.scss`

Add page-specific styles scoped to the `.register-page` and `.register-success` class names, reusing existing SCSS variables:

```scss
.register-success {
  text-align: center;
  padding: 24px 0;
  color: $primary-green;
  p {
    margin-bottom: 8px;
  }
}

.form-error {
  color: $red;
  font-size: 0.875rem;
  margin-bottom: 12px;
  text-align: left;
}
```

Update `src/assets/scss/pages/_index.scss` to add `@import "register"`.

---

### Guardrail Compliance Matrix

| Guardrail | Source | Compliance |
|-----------|--------|------------|
| Follow ESLint rules and consistent coding standards | Guardrail page | All new code uses functional components, consistent naming, and no ESLint-violating patterns. |
| Use functional components and React hooks | Guardrail page | `RegisterPage` is a functional component using `useState`, `useEffect`, `useContext`, `useHistory`. |
| Ensure all components are modular and reusable | Guardrail page | `RegisterPage` is self-contained. Reuses existing `Input`, `Field`, `Form`, `AuthLayout` components. |
| Sanitize user inputs to prevent XSS | Guardrail page | Yup validation sanitizes and validates all inputs before any processing. React's JSX escapes output by default. |
| Store sensitive data securely | Guardrail page | Passwords stored to `localStorage` only per assumption A-1 (no real backend). Passwords are never logged. |
| Use lazy loading for components and routes | Guardrail page | `RegisterPage` can be wrapped in `React.lazy` in a future iteration; current scope matches existing pattern in `App.js`. |
| Ensure all interactive elements are keyboard-navigable | Guardrail page | All form fields and buttons are native HTML elements — keyboard-navigable by default. |
| Use ARIA attributes to improve screen reader support | Guardrail page | `aria-label` on the form, `role="alert"` and `aria-live` on error/success messages. |
| Write unit tests for all components and hooks | Guardrail page | `src/pages/register.test.jsx` covers all 5 user stories with happy path, negative path, and edge cases. |
| Implement integration tests for critical user flows | Guardrail page | Integration tests cover the full registration flow from form render to redirect. |
| Display user-friendly error messages for failed actions | Guardrail page | Inline field errors via Yup + Input component. Form-level error for duplicate email. Success message on registration. |
| Log errors to monitoring | Guardrail page | `console.error` used on unexpected errors, matching existing project pattern. |
| No debug artifacts (console.log, commented-out code) | Definition of Done | No `console.log` calls in new code. No commented-out blocks. |
| No secrets or sensitive data logged | Definition of Done | Password values are never passed to `console.error` or any logging call. |
