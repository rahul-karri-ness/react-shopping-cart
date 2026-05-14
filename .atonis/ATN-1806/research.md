# Research: Account Creation — User Registration
**Ticket:** ATN-1806  
**Repository:** react-shopping-cart  

---

## 1. Tech Stack Analysis

| Layer | Technology | Version |
|-------|-----------|---------|
| UI Framework | React | ^17.0.2 |
| Routing | react-router-dom | ^5.2.0 |
| Form Management | Formik | ^2.2.6 |
| Schema Validation | Yup | ^0.32.9 |
| HTTP Client | axios | ^0.21.1 |
| Styling | SCSS (node-sass) | ^5.0.0 |
| CSS Utilities | classnames | ^2.2.6 |
| State Management | React Context API + useReducer | built-in |
| Persistence | localStorage via custom hook | custom |
| Testing | @testing-library/react + jest-dom | ^11.1.0 / ^5.11.4 |

---

## 2. Relevant Existing Code Modules

### 2.1 Authentication Context
**File:** `src/contexts/auth.jsx`  
**Lines:** 1–86

- Exports `AuthStateContext` and `AuthDispatchContext` (line 11–12)
- `reducer` handles: `LOGIN_REQUEST`, `LOGIN_SUCCESS`, `LOGIN_FAILURE`, `LOGOUT_SUCCESS` (lines 14–45)
- `signIn` action creator (lines 47–55): persists user to `localStorage` and dispatches `LOGIN_SUCCESS`
- `signOut` action creator (lines 57–62): clears `localStorage` and dispatches `LOGOUT_SUCCESS`
- `AuthProvider` (lines 64–84): wraps app with dual context providers; uses `useLocalStorage` for persistence
- **Gap:** No `REGISTER_REQUEST`, `REGISTER_SUCCESS`, or `REGISTER_FAILURE` action types exist — these must be added.

### 2.2 Auth Page (Login)
**File:** `src/pages/auth.jsx`  
**Lines:** 1–90

- Uses `Formik` + `Yup` for form management and validation (lines 9–12, 38–52)
- `LoginSchema` validates `username` (required) and `password` (required) only (lines 9–12)
- `goToRegister` handler exists but is a no-op stub (lines 24–26) — **must be wired to `/register` route**
- `signInSuccess` dispatches `signIn` and redirects via `history.push` (lines 28–35)
- Pattern to follow: Formik `<Form>` + `<Field component={Input}>` + Yup schema

### 2.3 Input Form Control
**File:** `src/components/core/form-controls/Input.jsx`  
**Lines:** 1–38

- Accepts Formik `field` and `form` props (line 9–11)
- Renders `is-invalid` class and `.invalid-feedback` div when `touched[field.name] && errors[field.name]` (lines 25–34)
- Supports `type`, `label`, `placeholder`, `className` props
- **Reuse as-is** for name, email, and password fields in the registration form

### 2.4 Auth Layout
**File:** `src/layouts/AuthLayout.jsx`  
**Lines:** 1–23

- Provides centred card layout with brand logo (lines 4–21)
- **Reuse as-is** — the registration page should use the same `AuthLayout`

### 2.5 Route Wrapper
**File:** `src/layouts/RouteWrapper.jsx`  
**Lines:** 1–35

- Wraps routes with a layout and handles private route redirection (lines 5–34)
- New `/register` route must be added to `src/App.js` using `RouteWrapper` with `AuthLayout`

### 2.6 App Router
**File:** `src/App.js`  
**Lines:** 1–51

- Three routes defined: `/` (home), `/checkout`, `/auth` (lines 25–40)
- New route `/register` must be added here (line ~40, before closing `</Switch>`)

### 2.7 Local Storage Hook
**File:** `src/hooks/useLocalStorage.js`  
**Lines:** 1–27

- Generic hook for reading/writing to `localStorage` with JSON serialisation (lines 3–26)
- Used by `AuthProvider` to persist user session
- **No changes needed** — registration will use the same hook via `AuthProvider`

### 2.8 SCSS — Auth Page Styles
**File:** `src/assets/scss/pages/_auth.scss`  
**Lines:** 1–26

- `.auth-container`, `.auth-brand`, `.auth-button` classes defined
- **Reuse as-is** — registration form will share the same auth page styles

### 2.9 SCSS — Form Controls
**File:** `src/assets/scss/components/_form-control.scss`  
**Lines:** 1–32

- `.form-group`, `.form-control`, `.invalid-feedback`, `.is-invalid` classes defined (lines 1–24)
- `.field-group` for side-by-side fields (lines 26–32)
- **Reuse as-is** — no new styles needed for standard fields

### 2.10 SCSS Variables
**File:** `src/assets/scss/base/_variables.scss`  
**Lines:** 1–13

- `$red: #e23d3d` used for error states (line 9)
- `$primary-green: #077915` used for focus states (line 1)

### 2.11 Constants
**File:** `src/constants/common.js`  
**Line:** 1

- `phoneRegExp` regex exported — pattern to follow for adding a `passwordRegExp` or Yup `.matches()` rule

---

## 3. Patterns to Follow

### 3.1 Form Pattern (Formik + Yup)
```jsx
// Pattern from src/pages/auth.jsx (lines 9-12, 38-52)
const RegistrationSchema = Yup.object().shape({
  name: Yup.string().required("Full name is required"),
  email: Yup.string().email("Invalid email").required("Email is required"),
  password: Yup.string()
    .min(8, "Password must be at least 8 characters")
    .matches(/[0-9]/, "Password must contain at least one number")
    .matches(/[!@#$%^&*]/, "Password must contain at least one special character")
    .required("Password is required")
});
```

### 3.2 Context Action Pattern
```jsx
// Pattern from src/contexts/auth.jsx (lines 47-55)
export const register = async (dispatch, userData) => {
  dispatch({ type: "REGISTER_REQUEST" });
  try {
    const response = await axios.post("/api/auth/register", userData);
    dispatch({ type: "REGISTER_SUCCESS", payload: { user: response.data } });
    return response.data;
  } catch (error) {
    dispatch({ type: "REGISTER_FAILURE", payload: { error: error.response?.data?.message } });
    throw error;
  }
};
```

### 3.3 Route Registration Pattern
```jsx
// Pattern from src/App.js (lines 25-40)
<RouteWrapper
  path="/register"
  component={RegisterPage}
  layout={AuthLayout}
/>
```

### 3.4 Field Component Pattern
```jsx
// Pattern from src/pages/auth.jsx (lines 56-67)
<Field
  name="name"
  type="text"
  label="Full Name"
  placeholder="Enter your full name"
  component={Input}
/>
```

---

## 4. External Dependencies

| Dependency | Purpose | Already Installed |
|-----------|---------|------------------|
| `axios` | HTTP requests to registration API | ✅ Yes (^0.21.1) |
| `formik` | Form state management | ✅ Yes (^2.2.6) |
| `yup` | Schema validation | ✅ Yes (^0.32.9) |
| `classnames` | Conditional CSS classes | ✅ Yes (^2.2.6) |
| `react-router-dom` | Client-side routing | ✅ Yes (^5.2.0) |

**No new npm packages are required.**

---

## 5. Files to Create

| File | Purpose |
|------|---------|
| `src/pages/register.jsx` | New registration page component with Formik form |
| `src/contexts/auth.jsx` | Extend with REGISTER_REQUEST/SUCCESS/FAILURE actions and `register` action creator |
| `src/App.js` | Add `/register` route |
| `src/pages/auth.jsx` | Wire `goToRegister` to navigate to `/register` |
| `src/pages/register.test.jsx` | Unit tests for the registration page |
| `src/contexts/auth.test.jsx` | Unit tests for auth context registration actions |

---

## 6. Risks and Unknowns

| # | Risk / Unknown | Severity | Mitigation |
|---|---------------|----------|------------|
| R-1 | Backend API endpoint for registration (`POST /api/auth/register`) is not confirmed. | High | Use a mock/stub in tests; document assumption in spec. |
| R-2 | "Email already in use" error format from API is unknown. | Medium | Handle generic API error messages; display `error.response.data.message` or a fallback string. |
| R-3 | Confirmation email delivery is a backend concern — no frontend control. | Low | Frontend only triggers the API call; no email sending logic on client. |
| R-4 | `console.log` and `console.error` debug statements exist in `src/pages/auth.jsx` (line 19, 50) — should not be replicated in new code. | Low | Use proper error handling without console statements in production code. |
| R-5 | `App.test.js` (line 4–7) tests for "learn react" text which will fail — pre-existing issue, not introduced by this ticket. | Low | Note as pre-existing; do not regress further. |
| R-6 | No API service layer exists — `axios` calls are made directly in context. | Medium | Follow the same pattern as existing auth context for consistency; consider a service layer as a future improvement. |
