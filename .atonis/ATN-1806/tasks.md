# Tasks: Account Creation — User Registration
**Ticket:** ATN-1806  
**Order:** TDD — each test task precedes its implementation task  
**[PARALLEL]** = can be worked on simultaneously with other [PARALLEL] tasks  

---

## Phase 1 — Auth Context Extension

### TASK-01 — Write tests for auth context registration reducer
**Type:** Test  
**File:** `src/contexts/auth.test.jsx` *(create)*  
**Covers:** REGISTER_REQUEST, REGISTER_SUCCESS, REGISTER_FAILURE reducer cases; `register` action creator  

Test cases to write:
- [ ] `REGISTER_REQUEST` sets `isRegistering: true` and clears `registrationError`
- [ ] `REGISTER_SUCCESS` sets `isRegistering: false` and `registrationError: null`
- [ ] `REGISTER_FAILURE` sets `isRegistering: false` and stores error message in `registrationError`
- [ ] `register` action creator dispatches `REGISTER_REQUEST` then `REGISTER_SUCCESS` on API success
- [ ] `register` action creator dispatches `REGISTER_REQUEST` then `REGISTER_FAILURE` on API error
- [ ] `register` action creator re-throws error after dispatching `REGISTER_FAILURE`
- [ ] Unknown action type still throws error (existing behaviour preserved)

**Acceptance checkpoint:** All 7 tests fail (red) before implementation.

---

### TASK-02 — Extend auth context with registration state and actions [PARALLEL]
**Type:** Implementation  
**File:** `src/contexts/auth.jsx` *(modify)*  
**Depends on:** TASK-01 (tests must exist first)  

Changes:
- [ ] Add `isRegistering: false` and `registrationError: null` to `initialState`
- [ ] Add `REGISTER_REQUEST` case to reducer
- [ ] Add `REGISTER_SUCCESS` case to reducer
- [ ] Add `REGISTER_FAILURE` case to reducer
- [ ] Add `import axios from "axios"` at top of file
- [ ] Export `register` async action creator (dispatches REQUEST → SUCCESS/FAILURE, re-throws on error)

**Acceptance checkpoint:** All TASK-01 tests pass (green).

---

## Phase 2 — Registration Page

### TASK-03 — Write tests for the registration page component [PARALLEL]
**Type:** Test  
**File:** `src/pages/register.test.jsx` *(create)*  
**Covers:** Rendering, validation, form submission, error display  

Test cases to write:

**Rendering:**
- [ ] Renders a form with Full Name, Email Address, and Password fields
- [ ] Renders a "Create Account" submit button
- [ ] Renders a "Sign In" link back to `/auth`
- [ ] All input fields have associated labels (accessibility)

**Validation — Happy Path:**
- [ ] Submitting with valid name, email, and strong password calls the register action
- [ ] Successful registration redirects to `/auth`
- [ ] Form resets after successful submission

**Validation — Negative Paths:**
- [ ] Submitting with empty name shows "Full name is required" error
- [ ] Submitting with empty email shows "Email address is required" error
- [ ] Submitting with empty password shows "Password is required" error
- [ ] Submitting with invalid email format shows "Please enter a valid email address" error
- [ ] Submitting with password shorter than 8 characters shows min-length error
- [ ] Submitting with password missing a number shows number requirement error
- [ ] Submitting with password missing a special character shows special character requirement error

**Edge Cases:**
- [ ] API returns 409 — email field shows "This email address is already registered."
- [ ] API returns generic 500 error — `registrationError` message is displayed with `role="alert"`
- [ ] Submit button is disabled and shows "Creating Account..." while `isRegistering` is true
- [ ] Name with only 1 character shows min-length error ("Name must be at least 2 characters")

**Acceptance checkpoint:** All tests fail (red) before implementation.

---

### TASK-04 — Create registration page component
**Type:** Implementation  
**File:** `src/pages/register.jsx` *(create)*  
**Depends on:** TASK-03 (tests must exist first), TASK-02 (auth context must be extended)  

Implementation steps:
- [ ] Import React, Formik, Form, Field, useHistory, Yup, AuthDispatchContext, AuthStateContext, register, Input
- [ ] Define `RegistrationSchema` with Yup: name (min 2, required), email (email, required), password (min 8, number match, special char match, required)
- [ ] Create `RegisterPage` functional component
- [ ] Consume `AuthDispatchContext` and `AuthStateContext` (for `isRegistering`, `registrationError`)
- [ ] Use `useHistory` for post-registration redirect to `/auth`
- [ ] Render Formik form with `initialValues: { name: "", email: "", password: "" }`
- [ ] Render three `<Field component={Input}>` fields: name, email, password
- [ ] Render API-level error message div with `role="alert"` when `registrationError` is set
- [ ] Render submit button with `disabled={isRegistering}` and conditional label
- [ ] Render "Sign In" link back to `/auth`
- [ ] Handle 409 conflict: call `setFieldError("email", "This email address is already registered.")`
- [ ] Export `RegisterPage` as default

**Acceptance checkpoint:** All TASK-03 tests pass (green).

---

## Phase 3 — Routing & Navigation

### TASK-05 — Write tests for routing and navigation [PARALLEL]
**Type:** Test  
**File:** `src/pages/register.test.jsx` *(extend)* and `src/pages/auth.test.jsx` *(create)*  
**Covers:** Route accessibility, "Sign Up Now!" link navigation  

Test cases to write:
- [ ] Navigating to `/register` renders the `RegisterPage` component
- [ ] Clicking "Sign Up Now!" on the auth page navigates to `/register`
- [ ] Clicking "Sign In" on the register page navigates to `/auth`

**Acceptance checkpoint:** All 3 tests fail (red) before implementation.

---

### TASK-06 — Add /register route to App.js
**Type:** Implementation  
**File:** `src/App.js` *(modify)*  
**Depends on:** TASK-04 (RegisterPage must exist)  

Changes:
- [ ] Import `RegisterPage` from `"pages/register"`
- [ ] Add `<RouteWrapper path="/register" component={RegisterPage} layout={AuthLayout} />` inside `<Switch>` before the closing tag

**Acceptance checkpoint:** `/register` route renders `RegisterPage` wrapped in `AuthLayout`.

---

### TASK-07 — Wire "Sign Up Now!" link in auth page
**Type:** Implementation  
**File:** `src/pages/auth.jsx` *(modify)*  
**Depends on:** TASK-06 (route must exist)  

Changes:
- [ ] Update `goToRegister` to call `history.push("/register")`
- [ ] Remove `console.log("location => ", location)` debug statement (line 19)

**Acceptance checkpoint:** All TASK-05 routing tests pass (green).

---

## Phase 4 — Quality & Verification

### TASK-08 — Self-review and Definition of Done checklist [PARALLEL]
**Type:** Review  
**Covers:** All files created/modified in this ticket  

- [ ] All new unit tests pass (happy paths, negative paths, ≥3 edge cases per story)
- [ ] All pre-existing tests still pass (no regression)
- [ ] Application builds without errors or warnings (`npm run build`)
- [ ] No `console.log`, `console.error`, or debug artifacts in new code
- [ ] No dead code or commented-out blocks
- [ ] All new functions have inline comments explaining purpose
- [ ] No secrets, API keys, or sensitive data hardcoded
- [ ] Password is never persisted to `localStorage`
- [ ] Error messages are user-friendly and descriptive
- [ ] All form inputs have accessible labels
- [ ] Error messages use `role="alert"` for screen readers
- [ ] Submit button disabled during async operation
- [ ] New code follows existing architectural patterns (Context API, Formik, Yup, RouteWrapper)

---

## Per-Story Checkpoints

| Story | Tasks | Done When |
|-------|-------|-----------|
| Story 1 — Access Registration Form | TASK-05, TASK-06, TASK-07 | "Sign Up Now!" navigates to `/register`; form renders with all 3 fields |
| Story 2 — Successful Registration | TASK-01, TASK-02, TASK-03, TASK-04 | Valid submission calls API, redirects to `/auth`, confirmation email triggered |
| Story 3 — Password Validation | TASK-03, TASK-04 | Inline errors shown for short, no-number, no-special-char passwords |
| Story 4 — Duplicate Email | TASK-03, TASK-04 | 409 response shows "email already registered" on email field |
| Story 5 — Invalid Input Handling | TASK-03, TASK-04 | All empty/invalid fields show descriptive inline errors; form not submitted |

---

## Task Dependency Graph

```
TASK-01 (test: auth context)
    └── TASK-02 (impl: auth context)
            └── TASK-04 (impl: register page)
                    └── TASK-06 (impl: add route)
                            └── TASK-07 (impl: wire link)

TASK-03 (test: register page) ──── [PARALLEL with TASK-01]
TASK-05 (test: routing)        ──── [PARALLEL with TASK-03]
TASK-08 (review/DoD)           ──── [after all above complete]
```
