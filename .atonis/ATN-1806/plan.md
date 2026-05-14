# Plan: Account Creation — User Registration
**Ticket:** ATN-1806  
**Repository:** react-shopping-cart  

---

## 1. Implementation Phases

### Phase 1 — Auth Context Extension
Extend `src/contexts/auth.jsx` to support registration state and actions.

### Phase 2 — Registration Page
Create `src/pages/register.jsx` with Formik form, Yup validation, and API integration.

### Phase 3 — Routing
Add `/register` route in `src/App.js` and wire the "Sign Up Now!" link in `src/pages/auth.jsx`.

### Phase 4 — Tests
Write unit tests for the registration page and auth context registration actions.

---

## 2. API Contract

### POST /api/auth/register

**Request Body:**
```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "Secure@123"
}
```

**Success Response — 201 Created:**
```json
{
  "id": "user-uuid",
  "name": "Jane Doe",
  "email": "jane@example.com",
  "token": "jwt-token-string"
}
```

**Error Response — 409 Conflict (email already in use):**
```json
{
  "message": "Email address is already registered."
}
```

**Error Response — 422 Unprocessable Entity (validation failure):**
```json
{
  "message": "Validation failed.",
  "errors": {
    "email": "Invalid email format",
    "password": "Password does not meet requirements"
  }
}
```

> **Note:** Actual endpoint and response shape must be confirmed with the backend team. The frontend will display `error.response.data.message` or a generic fallback.

---

## 3. Data Model Changes

### Auth Context State Extension

**Current state shape** (`src/contexts/auth.jsx`, lines 5–9):
```js
{
  isLoggedIn: false,
  user: null,
  isLoggingIn: false
}
```

**Extended state shape:**
```js
{
  isLoggedIn: false,
  user: null,
  isLoggingIn: false,
  isRegistering: false,      // NEW: tracks registration in-flight state
  registrationError: null    // NEW: stores API error message for display
}
```

### New Reducer Action Types

| Action Type | Payload | Effect |
|-------------|---------|--------|
| `REGISTER_REQUEST` | — | Sets `isRegistering: true`, clears `registrationError` |
| `REGISTER_SUCCESS` | `{ user }` | Sets `isRegistering: false`, `registrationError: null` |
| `REGISTER_FAILURE` | `{ error }` | Sets `isRegistering: false`, `registrationError: error` |

---

## 4. Files to Create / Modify

| File | Action | Description |
|------|--------|-------------|
| `src/contexts/auth.jsx` | **Modify** | Add `isRegistering`, `registrationError` to state; add 3 new reducer cases; add `register` async action creator |
| `src/pages/register.jsx` | **Create** | Registration page with Formik + Yup; fields: name, email, password; API call via `register` action |
| `src/App.js` | **Modify** | Import `RegisterPage`; add `<RouteWrapper path="/register" component={RegisterPage} layout={AuthLayout} />` |
| `src/pages/auth.jsx` | **Modify** | Wire `goToRegister` to `history.push("/register")`; remove `console.log` on line 19 |
| `src/pages/register.test.jsx` | **Create** | Unit tests: renders form, validates fields, handles success, handles duplicate email, handles API error |
| `src/contexts/auth.test.jsx` | **Create** | Unit tests: REGISTER_REQUEST, REGISTER_SUCCESS, REGISTER_FAILURE reducer cases; `register` action creator |

---

## 5. Detailed Implementation

### 5.1 `src/contexts/auth.jsx` — Changes

**Add to `initialState`:**
```js
isRegistering: false,
registrationError: null
```

**Add to `reducer`:**
```js
case "REGISTER_REQUEST":
  return { ...state, isRegistering: true, registrationError: null };
case "REGISTER_SUCCESS":
  return { ...state, isRegistering: false, registrationError: null };
case "REGISTER_FAILURE":
  return { ...state, isRegistering: false, registrationError: action.payload.error };
```

**Add `register` action creator:**
```js
export const register = async (dispatch, userData) => {
  dispatch({ type: "REGISTER_REQUEST" });
  try {
    const response = await axios.post("/api/auth/register", userData);
    dispatch({ type: "REGISTER_SUCCESS", payload: { user: response.data } });
    return response.data;
  } catch (error) {
    const message =
      error?.response?.data?.message || "Registration failed. Please try again.";
    dispatch({ type: "REGISTER_FAILURE", payload: { error: message } });
    throw error;
  }
};
```

### 5.2 `src/pages/register.jsx` — New File

```jsx
import React, { useContext } from "react";
import { Formik, Form, Field } from "formik";
import { useHistory } from "react-router-dom";
import * as Yup from "yup";
import { AuthDispatchContext, AuthStateContext, register } from "contexts/auth";
import Input from "components/core/form-controls/Input";

const RegistrationSchema = Yup.object().shape({
  name: Yup.string()
    .min(2, "Name must be at least 2 characters")
    .required("Full name is required"),
  email: Yup.string()
    .email("Please enter a valid email address")
    .required("Email address is required"),
  password: Yup.string()
    .min(8, "Password must be at least 8 characters")
    .matches(/[0-9]/, "Password must contain at least one number")
    .matches(/[!@#$%^&*()_+\-=[\]{};':"\\|,.<>/?]/, "Password must contain at least one special character")
    .required("Password is required")
});

const RegisterPage = () => {
  const authDispatch = useContext(AuthDispatchContext);
  const { isRegistering, registrationError } = useContext(AuthStateContext);
  const history = useHistory();

  const handleRegistrationSuccess = () => {
    history.push("/auth");
  };

  return (
    <Formik
      initialValues={{ name: "", email: "", password: "" }}
      validationSchema={RegistrationSchema}
      onSubmit={async (values, { resetForm, setFieldError }) => {
        try {
          await register(authDispatch, values);
          resetForm();
          handleRegistrationSuccess();
        } catch (err) {
          if (err?.response?.status === 409) {
            setFieldError("email", "This email address is already registered.");
          }
        }
      }}
    >
      {() => (
        <Form>
          <h2>Create Account</h2>
          <Field name="name" type="text" label="Full Name" placeholder="Enter your full name" component={Input} />
          <Field name="email" type="email" label="Email Address" placeholder="Enter your email address" component={Input} />
          <Field name="password" type="password" label="Password" placeholder="Create a password" component={Input} />
          {registrationError && (
            <div className="invalid-feedback" role="alert">{registrationError}</div>
          )}
          <button className="auth-button block" type="submit" disabled={isRegistering}>
            {isRegistering ? "Creating Account..." : "Create Account"}
          </button>
          <p>
            Already have an account?{" "}
            <a href="/auth" onClick={(e) => { e.preventDefault(); history.push("/auth"); }}>
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

### 5.3 `src/App.js` — Changes

Add import and route:
```jsx
import RegisterPage from "pages/register";
// ...
<RouteWrapper path="/register" component={RegisterPage} layout={AuthLayout} />
```

### 5.4 `src/pages/auth.jsx` — Changes

Wire `goToRegister` and remove debug `console.log`:
```jsx
const goToRegister = (e) => {
  e.preventDefault();
  history.push("/register");
};
// Remove: console.log("location => ", location);  // line 19
```

---

## 6. Guardrail Compliance Matrix

| Guardrail | Requirement | Compliance |
|-----------|-------------|------------|
| **Code Quality** | Follow existing Formik + Yup pattern | ✅ `register.jsx` mirrors `auth.jsx` pattern exactly |
| **Code Quality** | No dead code / debug artifacts | ✅ No `console.log`; removes existing one in `auth.jsx` |
| **Code Quality** | SOLID principles | ✅ Single responsibility: page handles UI, context handles state/API |
| **Security** | No hardcoded secrets or API keys | ✅ No secrets; API URL is a relative path |
| **Security** | Password never stored in plain text client-side | ✅ Password only sent to API; never persisted to localStorage |
| **Security** | Input validation on client side | ✅ Yup schema enforces min length, number, special char |
| **Performance** | No unnecessary re-renders | ✅ Context split (state vs dispatch) prevents unnecessary re-renders |
| **Accessibility** | Form labels for all inputs | ✅ `Input` component renders `<label>` when `label` prop provided |
| **Accessibility** | Error messages use `role="alert"` | ✅ API-level error div includes `role="alert"` |
| **Accessibility** | Button disabled state during submission | ✅ `disabled={isRegistering}` on submit button |
| **Testing** | Happy path covered | ✅ Successful registration test |
| **Testing** | Negative paths covered | ✅ Duplicate email, weak password, empty fields |
| **Testing** | Edge cases covered | ✅ API failure, network error, 409 conflict |
| **Error Handling** | Meaningful user-facing error messages | ✅ Field-level Yup errors + API error displayed in UI |
| **Error Handling** | Graceful API failure handling | ✅ try/catch with fallback message |
| **Patterns** | Follows existing Context API + useReducer pattern | ✅ New actions added to existing reducer |
| **Patterns** | Follows existing routing pattern | ✅ `RouteWrapper` + `AuthLayout` used |
| **No Regression** | Existing login flow unchanged | ✅ Only additive changes to `auth.jsx` context and page |

---

## 7. Assumptions Stated in Plan

1. Backend API endpoint is `POST /api/auth/register` — to be confirmed.
2. A 409 HTTP status code indicates "email already in use".
3. Registration redirects to `/auth` (login page) after success, not auto-login.
4. No `confirmPassword` field is required (not in acceptance criteria).
5. The `register` action creator uses `axios` directly, consistent with existing `signIn` pattern.
