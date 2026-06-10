# ATN-1806 - Implementation Plan

## Overview

Implement user registration for the react-shopping-cart application. No new npm dependencies are required. All work follows existing patterns: Formik + Yup validation, Context API + useReducer, Field + Input component, AuthLayout, RouteWrapper, and SCSS partials.

---

## Implementation Phases

### Phase 1: Add passwordRegExp Constant

**File:** `src/constants/common.js`

Add a `passwordRegExp` export alongside the existing `phoneRegExp`:

```js
export const passwordRegExp = /^(?=.*[0-9])(?=.*[!@#$%^&*])[a-zA-Z0-9!@#$%^&*]{8,}$/;
```

This regex enforces:
- At least one digit (`(?=.*[0-9])`)
- At least one special character from the set `!@#$%^&*` (`(?=.*[!@#$%^&*])`)
- Minimum 8 characters total

---

### Phase 2: Extend Auth Context

**File:** `src/contexts/auth.jsx`

**2a. Extend initialState:**
```js
const initialState = {
  isLoggedIn: false,
  user: null,
  isLoggingIn: false,
  isRegistering: false,
  registrationSuccess: false,
  registrationError: null
};
```

**2b. Add reducer cases:**
```js
case "REGISTER_REQUEST":
  return {
    ...state,
    isRegistering: true,
    registrationSuccess: false,
    registrationError: null
  };
case "REGISTER_SUCCESS":
  return {
    ...state,
    isRegistering: false,
    registrationSuccess: true,
    registrationError: null
  };
case "REGISTER_FAILURE":
  return {
    ...state,
    isRegistering: false,
    registrationSuccess: false,
    registrationError: action.payload.error
  };
```

**2c. Add register action creator:**
```js
export const register = (dispatch, userData) => {
  dispatch({ type: "REGISTER_REQUEST" });
  try {
    // Read existing registered users (case-insensitive duplicate check)
    const existing = JSON.parse(localStorage.getItem("registeredUsers") || "[]");
    const duplicate = existing.some(
      (u) => u.email.toLowerCase() === userData.email.toLowerCase()
    );
    if (duplicate) {
      dispatch({
        type: "REGISTER_FAILURE",
        payload: { error: "Email address is already in use." }
      });
      return { success: false, error: "Email address is already in use." };
    }
    // Persist new user (store email in lowercase for consistency)
    const newUser = { ...userData, email: userData.email.toLowerCase() };
    localStorage.setItem("registeredUsers", JSON.stringify([...existing, newUser]));
    dispatch({ type: "REGISTER_SUCCESS" });
    return { success: true };
  } catch (err) {
    dispatch({
      type: "REGISTER_FAILURE",
      payload: { error: "Registration failed. Please try again." }
    });
    return { success: false, error: "Registration failed. Please try again." };
  }
};
```

---

### Phase 3: Create RegisterPage Component

**File:** `src/pages/register.jsx`

```jsx
import React, { useContext, useState } from "react";
import { Formik, Form, Field } from "formik";
import { useHistory } from "react-router-dom";
import * as Yup from "yup";
import { AuthDispatchContext, register } from "contexts/auth";
import { passwordRegExp } from "constants/common";
import Input from "components/core/form-controls/Input";

// Yup schema — mirrors LoginSchema pattern from src/pages/auth.jsx
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
      "Password must be at least 8 characters and include a number and a special character (!@#$%^&*)."
    )
    .required("Password is required."),
  confirmPassword: Yup.string()
    .oneOf([Yup.ref("password"), null], "Passwords do not match.")
    .required("Please confirm your password.")
});

const RegisterPage = () => {
  const authDispatch = useContext(AuthDispatchContext);
  const history = useHistory();
  const [submitError, setSubmitError] = useState(null);
  const [successMessage, setSuccessMessage] = useState(null);

  const handleRegisterSuccess = () => {
    setSuccessMessage(
      "Registration successful! A confirmation email has been sent. Redirecting to login..."
    );
    // Redirect to /auth after 2 seconds (assumption A-5)
    setTimeout(() => {
      history.push("/auth");
    }, 2000);
  };

  return (
    <Formik
      initialValues={{
        fullName: "",
        email: "",
        password: "",
        confirmPassword: ""
      }}
      validationSchema={RegisterSchema}
      onSubmit={async (values, { resetForm, setFieldError }) => {
        setSubmitError(null);
        const result = register(authDispatch, {
          fullName: values.fullName,
          email: values.email,
          password: values.password
        });
        if (result.success) {
          resetForm();
          handleRegisterSuccess();
        } else {
          // Surface duplicate email error on the email field
          if (result.error && result.error.toLowerCase().includes("email")) {
            setFieldError("email", result.error);
          } else {
            setSubmitError(result.error || "Registration failed. Please try again.");
          }
        }
      }}
    >
      {() => (
        <Form>
          <h2 className="register-title">Create Account</h2>

          {successMessage && (
            <div className="register-success" role="alert">
              {successMessage}
            </div>
          )}

          {submitError && (
            <div className="register-error" role="alert">
              {submitError}
            </div>
          )}

          <Field
            name="fullName"
            type="text"
            label="Full Name"
            placeholder="Full Name"
            component={Input}
          />
          <Field
            name="email"
            type="email"
            label="Email Address"
            placeholder="Email Address"
            component={Input}
          />
          <Field
            name="password"
            type="password"
            label="Password"
            placeholder="Password"
            component={Input}
          />
          <Field
            name="confirmPassword"
            type="password"
            label="Confirm Password"
            placeholder="Confirm Password"
            component={Input}
          />

          <button type="submit" className="auth-button block">
            Create Account
          </button>

          <p>
            Already have an account?{" "}
            <a
              href="/#"
              onClick={(e) => {
                e.preventDefault();
                history.push("/auth");
              }}
            >
              Sign In
            </a>
          </p>
        </Form>
      )}
    </Formik>
  );
};

export default RegisterPage;
```

---

### Phase 4: Add /register Route in App.js

**File:** `src/App.js`

Add import for `RegisterPage` and a new `RouteWrapper` entry:

```jsx
import RegisterPage from "pages/register";

// Inside <Switch>:
<RouteWrapper
  path="/register"
  component={RegisterPage}
  layout={AuthLayout}
/>
```

The route must be placed before the `/auth` route to avoid catch-all conflicts.

---

### Phase 5: Wire goToRegister Navigation

**File:** `src/pages/auth.jsx`

Replace the no-op `goToRegister` stub (lines 24-26) with:

```js
const goToRegister = (e) => {
  e.preventDefault();
  history.push("/register");
};
```

Also remove the debug `console.log` on line 19.

---

### Phase 6: Create _register.scss

**File:** `src/assets/scss/pages/_register.scss`

```scss
.register-title {
  font-size: 20px;
  font-weight: 600;
  margin-bottom: 24px;
  color: $gray-dark;
}

.register-success {
  background: $green-light-bg;
  color: $primary-green;
  border-radius: 4px;
  padding: 12px 16px;
  margin-bottom: 16px;
  font-size: 14px;
  text-align: left;
}

.register-error {
  background: rgba(226, 61, 61, 0.1);
  color: $red;
  border-radius: 4px;
  padding: 12px 16px;
  margin-bottom: 16px;
  font-size: 14px;
  text-align: left;
}
```

**File:** `src/assets/scss/pages/_index.scss`

Add `@import "register";` after the existing imports.

---

## API Contracts

No external API calls. All data operations are client-side via `localStorage`.

### localStorage Schema

**Key:** `registeredUsers`
**Type:** JSON array
**Entry shape:**
```json
{
  "fullName": "string",
  "email": "string (lowercase)",
  "password": "string"
}
```

---

## Data Model Changes

| Location | Change |
|----------|--------|
| `src/contexts/auth.jsx` — `initialState` | Add `isRegistering: false`, `registrationSuccess: false`, `registrationError: null` |
| localStorage key `registeredUsers` | New key; array of `{ fullName, email, password }` objects |

---

## Guardrail Compliance Matrix

| # | Guardrail | Compliance |
|---|-----------|-----------|
| G-1 | No new npm dependencies | Compliant — all libraries already in package.json |
| G-2 | Follow Formik + Yup validation pattern | Compliant — RegisterSchema mirrors LoginSchema; Field + Input pattern used |
| G-3 | Follow Context API + useReducer pattern | Compliant — REGISTER_REQUEST/SUCCESS/FAILURE added to existing reducer |
| G-4 | Follow AuthLayout for auth pages | Compliant — /register route uses AuthLayout via RouteWrapper |
| G-5 | Follow SCSS partial pattern | Compliant — _register.scss created and imported in _index.scss |
| G-6 | No real backend; localStorage persistence | Compliant — registeredUsers key in localStorage |
| G-7 | Case-insensitive duplicate email check | Compliant — email.toLowerCase() comparison in register action creator |
| G-8 | Password min 8 chars, 1 number, 1 special char | Compliant — passwordRegExp enforced via Yup .matches() |
| G-9 | Full name min 2 characters | Compliant — Yup .min(2) on fullName field |
| G-10 | Redirect to /auth after 2 seconds on success | Compliant — setTimeout 2000ms + history.push("/auth") |
| G-11 | Inline error messages for all invalid fields | Compliant — Input component renders .invalid-feedback on touched + error |
| G-12 | No dead code, no debug artifacts | Compliant — console.log removed from auth.jsx; no commented-out code |
| G-13 | Functional components with hooks only | Compliant — RegisterPage is a functional component using useContext, useState, useHistory |
| G-14 | Error messages are user-facing and meaningful | Compliant — all Yup messages and action creator errors are descriptive |
