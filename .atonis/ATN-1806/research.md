# Research: ATN-1806 — Account Creation / User Registration

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| UI Framework | React | ^17.0.2 |
| Routing | react-router-dom | ^5.2.0 |
| Form Management | Formik | ^2.2.6 |
| Validation | Yup | ^0.32.9 |
| Styling | SCSS (node-sass) | ^5.0.0 |
| HTTP Client | axios | ^0.21.1 |
| CSS Utilities | classnames | ^2.2.6 |
| State Persistence | localStorage (custom hook) | — |
| Testing | @testing-library/react + jest | ^11.1.0 |

---

## Relevant Code Modules

### 1. Auth Context — `src/contexts/auth.jsx`
**Lines 1–86**

The central auth state management module. Uses `useReducer` + React Context pattern (split into `AuthStateContext` and `AuthDispatchContext`).

**Current actions supported:**
- `LOGIN_REQUEST` (line 16)
- `LOGIN_SUCCESS` (line 23)
- `LOGIN_FAILURE` (line 30)
- `LOGOUT_SUCCESS` (line 37)

**Exported action creators:**
- `signIn(dispatch, userData)` — line 47: stores user in `localStorage` and dispatches `LOGIN_SUCCESS`
- `signOut(dispatch)` — line 57: clears `localStorage` and dispatches `LOGOUT_SUCCESS`

**State shape (line 5–9):**
```js
{
  isLoggedIn: false,
  user: null,
  isLoggingIn: false
}
```

**Persistence:** Uses `useLocalStorage` hook (line 65) to persist `user` object under key `"user"`.

**What needs to change:**
- Add `REGISTER_REQUEST`, `REGISTER_SUCCESS`, `REGISTER_FAILURE` action types to the reducer.
- Add `register(dispatch, userData)` action creator.
- Extend state with `isRegistering: false` and `registrationSuccess: false`.

---

### 2. Auth Page — `src/pages/auth.jsx`
**Lines 1–90**

Currently renders only the **Login form** using Formik + Yup. Has stub handlers for `goToRegister` (line 24) and `goToForgotPassword` (line 20) that do nothing (`e.preventDefault()` only).

**Current Yup schema (lines 9–12):**
```js
const LoginSchema = Yup.object().shape({
  password: Yup.string().required("Password is required!"),
  username: Yup.string().required("Mobile Number or Email Address is required!")
});
```

**What needs to change:**
- Add a `view` state toggle (`"login"` | `"register"`) to switch between Login and Register forms.
- Implement `goToRegister` to set view to `"register"`.
- Add a `RegisterSchema` Yup validation schema with name, email, password, confirmPassword rules.
- Add a `RegisterForm` component (or inline JSX) with the four required fields.
- Wire `onSubmit` for registration to call the new `register` action creator.

---

### 3. Input Component — `src/components/core/form-controls/Input.jsx`
**Lines 1–38**

Reusable Formik-compatible input component. Accepts `type`, `label`, `placeholder`, `className`, `field` (Formik field props), and `form` (Formik form state).

**Inline error display (lines 32–34):**
```jsx
{touched[field.name] && errors[field.name] && (
  <div className="invalid-feedback">{errors[field.name]}</div>
)}
```

**Pattern to follow:** All registration form fields must use this `Input` component via Formik `<Field component={Input} />`.

---

### 4. Auth Layout — `src/layouts/AuthLayout.jsx`
**Lines 1–23**

Wraps auth pages in a centered card layout with the brand logo. The registration form will reuse this layout — no changes needed.

---

### 5. Route Wrapper — `src/layouts/RouteWrapper.jsx`
**Lines 1–35**

Handles private route redirection. The `/auth` route is public (`isPrivate` defaults to `false`). No changes needed for registration.

---

### 6. App Router — `src/App.js`
**Lines 16–49**

Defines the route structure. The `/auth` route (line 37–40) maps to `AuthPage` with `AuthLayout`. 

**What may need to change:**
- Optionally add a `/register` route if the team prefers a separate URL (assumed: toggle within `/auth` is sufficient per assumptions).

---

### 7. useLocalStorage Hook — `src/hooks/useLocalStorage.js`
**Lines 1–28**

Generic hook for reading/writing to `localStorage` with JSON serialization. Used by `AuthProvider` (line 65 of `auth.jsx`) and `CartProvider`.

**Pattern for storing registered users:**
- Store registered users as an array under key `"registeredUsers"` in `localStorage`.
- On registration, check if email already exists in this array.

---

### 8. SCSS — Auth Styles — `src/assets/scss/pages/_auth.scss`
**Lines 1–26**

Defines `.auth-container`, `.auth-brand`, `.auth-button` classes. The registration form will reuse these styles.

**What may need to change:**
- Add styles for the registration form toggle link and success message.

---

### 9. SCSS — Form Controls — `src/assets/scss/components/_form-control.scss`
**Lines 1–32**

Defines `.form-group`, `.form-control`, `.invalid-feedback`, `.field-group` (side-by-side fields). The `.field-group` class (line 26) can be used for side-by-side layout if needed.

---

### 10. Constants — `src/constants/common.js`
**Line 1**

Contains `phoneRegExp` regex. New password validation regex should be added here:
```js
export const passwordRegExp = /^(?=.*[0-9])(?=.*[!@#$%^&*])[a-zA-Z0-9!@#$%^&*]{8,}$/;
```

---

## Patterns to Follow

### Context Pattern
All contexts follow the **split context pattern** (separate State and Dispatch contexts) with `useReducer`. New registration state must follow this same pattern in `src/contexts/auth.jsx`.

### Formik + Yup Pattern
All forms use `Formik` with a `Yup` schema passed as `validationSchema`. Field components use `<Field component={Input} />`. This pattern must be followed for the registration form.

### Action Creator Pattern
Auth actions are exported as standalone functions (e.g., `signIn`, `signOut`) that accept `dispatch` as the first argument. The new `register` action creator must follow this pattern.

### localStorage Persistence
User data is persisted via `useLocalStorage` hook. Registered users list should be stored under a dedicated key (e.g., `"registeredUsers"`).

---

## External Dependencies

| Dependency | Purpose | Already Installed |
|-----------|---------|------------------|
| Formik | Form state management | ✅ Yes |
| Yup | Schema validation | ✅ Yes |
| classnames | Conditional CSS classes | ✅ Yes |
| axios | HTTP requests (future API) | ✅ Yes |
| react-router-dom | Navigation | ✅ Yes |

No new dependencies are required.

---

## Risks & Unknowns

| Risk | Severity | Mitigation |
|------|----------|-----------|
| No real backend — email confirmation is simulated | Medium | Clearly document assumption; show mock success state |
| Email uniqueness check is client-side only | Medium | Store registered users in `localStorage`; note this is a temporary solution |
| `console.log` debug statement in `auth.jsx` line 19 | Low | Remove before shipping |
| `auth.jsx` has stub `goToRegister` and `goToForgotPassword` handlers | Low | Implement `goToRegister` as part of this ticket |
| No existing unit tests for auth flows | High | Write tests as part of this ticket (TDD approach) |
| `App.test.js` test (line 4) looks for "learn react" text — will fail | High | Fix or remove this stale test |
| Password regex complexity — must cover edge cases | Medium | Use well-tested Yup `.matches()` with clear error messages |
