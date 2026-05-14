# Tasks: ATN-1806 — Account Creation / User Registration

> **TDD Order:** Every test task precedes the implementation task it covers.  
> **[PARALLEL]** marks tasks that can be executed concurrently.  
> Tasks are grouped by story checkpoint.

---

## Phase 0 — Setup & Stale Test Fix

### TASK-001 — Fix stale App.test.js
**Type:** Fix  
**File:** `src/App.test.js`  
**Description:** The existing test asserts `screen.getByText(/learn react/i)` which does not match any rendered content. Update the test to assert something meaningful (e.g., the app renders without crashing).  
**Acceptance:** `npm test` passes with no failures before any new code is written.

---

## Phase 1 — Auth Context Extension (TDD)

### TASK-002 — Write tests for auth context registration actions [PARALLEL]
**Type:** Test  
**File:** `src/contexts/auth.test.js` *(create new)*  
**Description:** Write unit tests for the auth reducer and `register` action creator BEFORE implementing them.

**Test cases to cover:**
- ✅ Happy path: `REGISTER_SUCCESS` sets `registrationSuccess: true`, stores user in `registeredUsers`
- ✅ Happy path: `REGISTER_REQUEST` sets `isRegistering: true`
- ✅ Negative path: `REGISTER_FAILURE` sets `registrationError` with message, `registrationSuccess: false`
- ✅ Edge case: Duplicate email triggers `REGISTER_FAILURE` with "This email address is already in use."
- ✅ Edge case: `register` with empty `registeredUsers` array (first user) succeeds
- ✅ Edge case: `register` with `null` registeredUsers falls back gracefully

**Depends on:** TASK-001

---

### TASK-003 — Extend auth context with registration state and actions [PARALLEL]
**Type:** Implementation  
**File:** `src/contexts/auth.jsx` *(modify)*  
**Description:** Implement the registration state, reducer cases, and `register` action creator.

**Changes:**
1. Add to `initialState`:
   ```js
   isRegistering: false,
   registrationSuccess: false,
   registrationError: null
   ```
2. Add reducer cases:
   - `REGISTER_REQUEST` → `{ ...state, isRegistering: true, registrationSuccess: false, registrationError: null }`
   - `REGISTER_SUCCESS` → `{ ...state, isRegistering: false, registrationSuccess: true, registrationError: null }`
   - `REGISTER_FAILURE` → `{ ...state, isRegistering: false, registrationSuccess: false, registrationError: action.payload.error }`
3. Add `register(dispatch, userData, registeredUsers)` action creator:
   - Dispatch `REGISTER_REQUEST`
   - Check for duplicate email in `registeredUsers`
   - If duplicate: dispatch `REGISTER_FAILURE` with error message
   - If unique: append new user to `registeredUsers`, save to `localStorage`, dispatch `REGISTER_SUCCESS`
   - Simulate email: `console.info("Confirmation email sent to:", userData.email)`

**Depends on:** TASK-002 (tests must be written first)

---

### ✅ Checkpoint 1: Auth context tests pass for all registration reducer cases and action creators.

---

## Phase 2 — Constants Update

### TASK-004 — Add passwordRegExp to constants [PARALLEL]
**Type:** Implementation  
**File:** `src/constants/common.js` *(modify)*  
**Description:** Export a `passwordRegExp` regex constant for use in Yup validation.

```js
export const passwordRegExp = /^(?=.*[0-9])(?=.*[!@#$%^&*]).{8,}$/;
```

**Note:** This regex is used in the Yup schema in TASK-006. No test file needed for a pure constant — it is covered by the Yup schema tests in TASK-005.

---

## Phase 3 — RegisterForm Component (TDD)

### TASK-005 — Write tests for RegisterForm component [PARALLEL]
**Type:** Test  
**File:** `src/components/RegisterForm.test.jsx` *(create new)*  
**Description:** Write unit and integration tests for the `RegisterForm` component BEFORE implementing it.

**Test cases to cover:**
- ✅ Happy path: Renders all four fields (name, email, password, confirmPassword)
- ✅ Happy path: Renders a "Register" submit button
- ✅ Happy path: Renders a "Back to Login" link
- ✅ Happy path: Valid form submission calls `register` action with correct data
- ✅ Happy path: Success message is displayed after successful registration
- ✅ Negative path: Empty form submission shows required field errors for all fields
- ✅ Negative path: Invalid email format shows "Please enter a valid email address."
- ✅ Negative path: Password < 8 chars shows "Password must be at least 8 characters."
- ✅ Negative path: Password without number shows "Password must contain at least one number."
- ✅ Negative path: Password without special char shows appropriate error
- ✅ Negative path: Mismatched passwords shows "Passwords do not match."
- ✅ Negative path: Duplicate email shows "This email address is already in use."
- ✅ Edge case: Password exactly 8 chars with number and special char passes validation
- ✅ Edge case: Name with exactly 2 characters passes validation
- ✅ Edge case: Name with 1 character fails with "Name must be at least 2 characters."

**Depends on:** TASK-003

---

### TASK-006 — Create RegisterForm component
**Type:** Implementation  
**File:** `src/components/RegisterForm.jsx` *(create new)*  
**Description:** Implement the registration form as a standalone Formik component.

**Implementation details:**
- Import `useContext` from React; consume `AuthDispatchContext` and `AuthStateContext`
- Import `useLocalStorage` hook to read `registeredUsers`
- Define `RegisterSchema` using Yup (name, email, password, confirmPassword)
- Render Formik form with four `<Field component={Input} />` fields
- On submit: call `register(authDispatch, { name, email, password }, registeredUsers)`
- On `REGISTER_SUCCESS` (via `registrationSuccess` from context): show success message div with class `auth-success`
- On duplicate email: use `setFieldError("email", "This email address is already in use.")`
- Render "Already have an account? Sign In" link that calls `onSwitchToLogin` prop
- Wrap submit in `try/catch`; log errors with `console.error`

**Props:**
```js
RegisterForm.propTypes = {
  onSwitchToLogin: PropTypes.func.isRequired
}
```

**Depends on:** TASK-005 (tests must be written first), TASK-003, TASK-004

---

### ✅ Checkpoint 2: RegisterForm renders correctly, all validation rules work, success/error states display properly.

---

## Phase 4 — Auth Page Update (TDD)

### TASK-007 — Write tests for AuthPage view toggling [PARALLEL]
**Type:** Test  
**File:** `src/pages/auth.test.jsx` *(create new)*  
**Description:** Write integration tests for the `AuthPage` component covering view toggling BEFORE implementing it.

**Test cases to cover:**
- ✅ Happy path: AuthPage renders Login form by default
- ✅ Happy path: Clicking "Sign Up Now!" link renders the RegisterForm
- ✅ Happy path: Clicking "Back to Login" from RegisterForm renders the Login form
- ✅ Negative path: Login form is not visible when register view is active
- ✅ Negative path: Register form is not visible when login view is active
- ✅ Edge case: View toggle does not reset login form values (login form state preserved)
- ✅ Edge case: Navigating to register and back to login clears registration success state

**Depends on:** TASK-006

---

### TASK-008 — Update AuthPage to support view toggling
**Type:** Implementation  
**File:** `src/pages/auth.jsx` *(modify)*  
**Description:** Update the auth page to toggle between Login and Register views.

**Changes:**
1. Add `import React, { useContext, useState } from "react"` (add `useState`)
2. Add `import RegisterForm from "components/RegisterForm"`
3. Add `const [view, setView] = useState("login")` inside `AuthPage`
4. Implement `goToRegister`: `(e) => { e.preventDefault(); setView("register"); }`
5. Add `goToLogin`: `(e) => { e.preventDefault(); setView("register"); }` → `setView("login")`
6. Conditionally render:
   ```jsx
   {view === "register"
     ? <RegisterForm onSwitchToLogin={goToLogin} />
     : <Formik ...>{/* existing login form */}</Formik>
   }
   ```
7. Remove `console.log("location => ", location)` debug statement

**Depends on:** TASK-007 (tests must be written first), TASK-006

---

### ✅ Checkpoint 3: AuthPage correctly toggles between Login and Register views. All navigation links work.

---

## Phase 5 — SCSS Updates

### TASK-009 — Add registration styles to auth SCSS [PARALLEL]
**Type:** Implementation  
**File:** `src/assets/scss/pages/_auth.scss` *(modify)*  
**Description:** Add styles for the success message and toggle link.

**Styles to add:**
```scss
.auth-success {
  background: $green-light-bg;
  border: 1px solid $primary-green;
  color: $primary-green;
  border-radius: 8px;
  padding: 12px 16px;
  margin-bottom: 16px;
  font-size: 14px;
  text-align: left;
}

.auth-toggle-link {
  color: $primary-green;
  text-decoration: underline;
  cursor: pointer;
  &:hover {
    color: darken($primary-green, 10%);
  }
}
```

**Depends on:** None (can run in parallel with all other tasks)

---

## Phase 6 — Final Verification

### TASK-010 — Run full test suite and verify build
**Type:** Verification  
**Description:** Run `npm test -- --watchAll=false` and `npm run build` to confirm:
- All new tests pass
- No existing tests are broken
- Build completes without errors or warnings

**Depends on:** TASK-001 through TASK-009

---

### TASK-011 — Self-review checklist
**Type:** Review  
**Description:** Perform a final code review pass against the Definition of Done.

**Checklist:**
- [ ] All new work has unit tests: happy paths, negative paths, ≥3 edge cases ✅
- [ ] All tests in the app pass ✅
- [ ] Application builds without errors or warnings ✅
- [ ] No regression introduced — login flow unaffected ✅
- [ ] Code self-reviewed: correctness, performance, security, maintainability ✅
- [ ] No dead code, no `console.log` debug artifacts ✅
- [ ] New functions and complex logic have inline comments ✅
- [ ] No secrets, API keys, or sensitive data hardcoded ✅
- [ ] New code follows existing architectural patterns ✅
- [ ] Errors handled gracefully with meaningful user-facing messages ✅

**Depends on:** TASK-010

---

## Task Summary Table

| Task ID | Type | File | Parallel | Depends On |
|---------|------|------|----------|-----------|
| TASK-001 | Fix | `src/App.test.js` | No | — |
| TASK-002 | Test | `src/contexts/auth.test.js` | Yes | TASK-001 |
| TASK-003 | Impl | `src/contexts/auth.jsx` | Yes | TASK-002 |
| TASK-004 | Impl | `src/constants/common.js` | Yes | — |
| TASK-005 | Test | `src/components/RegisterForm.test.jsx` | Yes | TASK-003 |
| TASK-006 | Impl | `src/components/RegisterForm.jsx` | No | TASK-005, TASK-003, TASK-004 |
| TASK-007 | Test | `src/pages/auth.test.jsx` | Yes | TASK-006 |
| TASK-008 | Impl | `src/pages/auth.jsx` | No | TASK-007, TASK-006 |
| TASK-009 | Impl | `src/assets/scss/pages/_auth.scss` | Yes | — |
| TASK-010 | Verify | — | No | TASK-001–009 |
| TASK-011 | Review | — | No | TASK-010 |
