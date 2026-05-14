# Tasks: ATN-1806 — Account Creation / User Registration

> Tasks are ordered in TDD sequence: each **test task precedes** the implementation task it covers.  
> Tasks marked **[PARALLEL]** can be executed concurrently with other parallel tasks.  
> Per-story checkpoints are marked with 🏁.

---

## Phase 1 — Auth Context Extension

### TASK-01 · [TEST] Auth Context — Registration Reducer Tests
**File to create:** `src/contexts/auth.test.jsx`  
**Covers:** TASK-02

Write unit tests for the extended auth reducer covering:
- `REGISTER_REQUEST` sets `isRegistering: true`, `isLoggedIn: false`
- `REGISTER_SUCCESS` sets `isLoggedIn: true`, `isRegistering: false`, and populates `user`
- `REGISTER_FAILURE` sets `isRegistering: false`, `isLoggedIn: false`, `user: null`
- `register()` action creator — happy path: stores user in localStorage, dispatches `REGISTER_SUCCESS`
- `register()` action creator — duplicate email: throws error, dispatches `REGISTER_FAILURE`
- `register()` action creator — password is NOT stored in the persisted user object
- Unknown action type still throws error (existing behaviour preserved)

**Acceptance:** All 7 test cases pass; existing LOGIN/LOGOUT tests are unaffected.

---

### TASK-02 · [IMPL] Auth Context — Add Registration Actions
**File to modify:** `src/contexts/auth.jsx`

- Add `isRegistering: false` to `initialState`
- Add `REGISTER_REQUEST`, `REGISTER_SUCCESS`, `REGISTER_FAILURE` cases to the reducer
- Export `register(dispatch, userData)` action creator:
  - Dispatches `REGISTER_REQUEST`
  - Reads existing user from `localStorage`; if `existingUser.email === userData.email`, dispatches `REGISTER_FAILURE` and throws `new Error("EMAIL_ALREADY_IN_USE")`
  - Builds safe user object: `{ name, email, username: email }` (no password)
  - Stores safe user in `localStorage`
  - Dispatches `REGISTER_SUCCESS` with safe user as payload

**Acceptance:** TASK-01 tests pass; no regressions on existing auth tests.

---

🏁 **Checkpoint — Story 4 (Duplicate Email Prevention):** Auth context correctly rejects duplicate emails.

---

## Phase 2 — Constants Update

### TASK-03 · [TEST] [PARALLEL] Constants — Password Regex Tests
**File to create:** `src/constants/common.test.js`  
**Covers:** TASK-04

Write unit tests for `passwordRegExp`:
- ✅ Happy path: `"Password1!"` matches
- ✅ Happy path: `"Secure@99"` matches
- ❌ Negative: `"short1!"` (< 8 chars) does not match
- ❌ Negative: `"NoNumbers!"` (no digit) does not match
- ❌ Negative: `"NoSpecial1"` (no special char) does not match
- ❌ Edge case: empty string does not match
- ❌ Edge case: exactly 8 chars with digit and special: `"Pass1!ab"` matches
- ❌ Edge case: only special chars and digits, no letters: `"12345678!"` matches (regex allows)

**Acceptance:** All 8 test cases pass.

---

### TASK-04 · [PARALLEL] [IMPL] Constants — Add passwordRegExp
**File to modify:** `src/constants/common.js`

Add:
```js
// Password must be min 8 chars, contain at least 1 digit and 1 special character
export const passwordRegExp = /^(?=.*[0-9])(?=.*[!@#$%^&*])[a-zA-Z0-9!@#$%^&*]{8,}$/;
```

**Acceptance:** TASK-03 tests pass.

---

🏁 **Checkpoint — Story 3 (Password Validation):** Password regex is defined, tested, and centralised.

---

## Phase 3 — Registration Page

### TASK-05 · [TEST] RegisterPage — Render Tests
**File to create:** `src/pages/register.test.jsx`  
**Covers:** TASK-06 (partial)

Write render tests:
- Registration form renders with Name, Email, and Password fields
- "Create Account" submit button is present
- "Already have an account? Login" link is present
- Page title / heading "Create Account" is visible

**Acceptance:** All 4 render tests pass.

---

### TASK-06 · [IMPL] RegisterPage — Component Scaffold
**File to create:** `src/pages/register.jsx`

Create the `RegisterPage` functional component:
- Import `Formik`, `Form`, `Field` from `formik`
- Import `* as Yup` from `yup`
- Import `useContext` from `react`; `useHistory` from `react-router-dom`
- Import `AuthDispatchContext` from `contexts/auth`
- Import `Input` from `components/core/form-controls/Input`
- Import `passwordRegExp` from `constants/common`
- Define `RegisterSchema` with `name`, `email`, `password` fields (see plan.md)
- Render Formik form with three `Field` components using the `Input` component
- Add "Create Account" submit button with `className="auth-button block"`
- Add "Already have an account?" link pointing to `/auth`
- `onSubmit` stub: `console.log(values)` (to be replaced in TASK-08)

**Acceptance:** TASK-05 render tests pass; page displays correctly at `/register`.

---

### TASK-07 · [TEST] RegisterPage — Validation Tests
**File to modify:** `src/pages/register.test.jsx`  
**Covers:** TASK-06 (validation behaviour)

Add validation tests:
- Submitting empty form shows "Full name is required", "Email address is required", "Password is required"
- Entering name with 1 character shows "Name must be at least 2 characters"
- Entering invalid email format shows "Please enter a valid email address"
- Entering password < 8 chars shows "Password must be at least 8 characters"
- Entering password without digit/special char shows "Password must contain at least one number and one special character"
- Valid inputs produce no error messages

**Acceptance:** All 6 validation tests pass.

---

### TASK-08 · [TEST] RegisterPage — Submission Tests
**File to modify:** `src/pages/register.test.jsx`  
**Covers:** TASK-09

Add submission tests:
- Happy path: valid form submission calls `register()` with correct `{ name, email, password }` payload
- Happy path: after successful registration, user is redirected to `/`
- Duplicate email: form submission with duplicate email shows "An account with this email address already exists."
- Duplicate email: form is NOT reset after duplicate email error
- Edge case: form submit button is disabled while `isRegistering` is true
- Edge case: password field value is never exposed in the rendered DOM after submission

**Acceptance:** All 6 submission tests pass.

---

### TASK-09 · [IMPL] RegisterPage — Submission Logic
**File to modify:** `src/pages/register.jsx`

Replace the `onSubmit` stub with full logic:
- Call `register(authDispatch, values)` inside a `try/catch`
- On success: `resetForm()`, redirect to `/` via `history.push("/")`
- On `EMAIL_ALREADY_IN_USE` error: set Formik `setStatus({ serverError: "An account with this email address already exists." })`
- Render `status.serverError` as an error banner above the form when present
- Add `aria-live="polite"` to the error banner for accessibility
- Add `aria-describedby` linking each input to its error message

**Acceptance:** TASK-08 submission tests pass; TASK-07 validation tests still pass.

---

🏁 **Checkpoint — Story 2 (Successful Registration) & Story 5 (Invalid Input Errors):** Registration form fully functional with validation and submission.

---

## Phase 4 — Router Update

### TASK-10 · [TEST] [PARALLEL] App Router — Route Tests
**File to modify:** `src/App.test.js`  
**Covers:** TASK-11

Add route tests:
- Navigating to `/register` renders the `RegisterPage` component
- Navigating to `/register` uses `AuthLayout` (`.auth-container` class is present)
- `/register` route is publicly accessible (no redirect when not logged in)

**Acceptance:** All 3 route tests pass.

---

### TASK-11 · [PARALLEL] [IMPL] App Router — Add /register Route
**File to modify:** `src/App.js`

- Import `RegisterPage` from `pages/register`
- Add inside `<Switch>`:
  ```jsx
  <RouteWrapper
    path="/register"
    component={RegisterPage}
    layout={AuthLayout}
  />
  ```

**Acceptance:** TASK-10 route tests pass; existing routes unaffected.

---

🏁 **Checkpoint — Story 1 (Access Registration Form):** `/register` route is live and publicly accessible.

---

## Phase 5 — Login Page Navigation Fix

### TASK-12 · [TEST] [PARALLEL] AuthPage — Navigation Tests
**File to modify or create:** `src/pages/auth.test.jsx`  
**Covers:** TASK-13

Add navigation tests:
- "Sign Up Now!" link is present on the login page
- Clicking "Sign Up Now!" navigates to `/register`
- "Forgot Password?" link is present (existing behaviour preserved)

**Acceptance:** All 3 navigation tests pass.

---

### TASK-13 · [PARALLEL] [IMPL] AuthPage — Wire Up goToRegister
**File to modify:** `src/pages/auth.jsx`

- Add `useHistory` import from `react-router-dom`
- Replace `goToRegister` stub with:
  ```js
  const goToRegister = (e) => {
    e.preventDefault();
    history.push("/register");
  };
  ```

**Acceptance:** TASK-12 navigation tests pass; login form functionality unaffected.

---

🏁 **Checkpoint — Story 1 (Access Registration Form):** Login page correctly navigates to registration page.

---

## Phase 6 — Integration & Regression

### TASK-14 · [TEST] End-to-End Registration Flow Test
**File to modify:** `src/pages/register.test.jsx`

Add full flow integration test:
- User lands on `/auth`, clicks "Sign Up Now!", arrives at `/register`
- Fills in valid name, email, password
- Submits form
- Is redirected to `/`
- `localStorage` contains user object with `name`, `email`, `username` but **no** `password`

**Acceptance:** Integration test passes; no regressions in existing tests.

---

### TASK-15 · Final Quality Gate
**No file changes — verification only**

- [ ] Run `npm test -- --watchAll=false` — all tests pass, zero failures
- [ ] Run `npm run build` — build completes with zero errors and zero warnings
- [ ] Verify no `console.log` statements in new/modified files
- [ ] Verify no dead code, commented-out blocks, or debug artifacts
- [ ] Verify password is not present in `localStorage` after registration
- [ ] Verify all new functions have inline JSDoc comments
- [ ] Self-review checklist (Definition of Done) complete

**Acceptance:** All checks pass; PR is ready for developer review.

---

## Task Summary

| Task | Type | Phase | Parallel? | Depends On |
|------|------|-------|-----------|------------|
| TASK-01 | TEST | 1 | No | — |
| TASK-02 | IMPL | 1 | No | TASK-01 |
| TASK-03 | TEST | 2 | ✅ Yes | — |
| TASK-04 | IMPL | 2 | ✅ Yes | TASK-03 |
| TASK-05 | TEST | 3 | No | TASK-02, TASK-04 |
| TASK-06 | IMPL | 3 | No | TASK-05 |
| TASK-07 | TEST | 3 | No | TASK-06 |
| TASK-08 | TEST | 3 | No | TASK-07 |
| TASK-09 | IMPL | 3 | No | TASK-08 |
| TASK-10 | TEST | 4 | ✅ Yes | TASK-06 |
| TASK-11 | IMPL | 4 | ✅ Yes | TASK-10 |
| TASK-12 | TEST | 5 | ✅ Yes | — |
| TASK-13 | IMPL | 5 | ✅ Yes | TASK-12 |
| TASK-14 | TEST | 6 | No | TASK-09, TASK-11, TASK-13 |
| TASK-15 | VERIFY | 6 | No | TASK-14 |
