# ATN-1806 - Task Breakdown (TDD Order)

All test tasks precede their corresponding implementation tasks. Tasks marked [PARALLEL] can be executed concurrently with other [PARALLEL] tasks in the same group.

---

## TASK-01 — [TEST] passwordRegExp constant validation [PARALLEL]

**Phase:** 1
**File:** `src/constants/common.test.js` (new)
**Covers:** FR-3, AC-3

Write unit tests for the `passwordRegExp` constant:

- Happy path: password with 8+ chars, a digit, and a special character matches
- Negative path: password without a digit does not match
- Negative path: password without a special character does not match
- Edge case: exactly 8 characters with digit and special char matches
- Edge case: 7 characters (too short) does not match
- Edge case: empty string does not match
- Edge case: only special characters and digits but no letters — still matches (regex allows it)

---

## TASK-02 — [IMPL] Add passwordRegExp to constants [PARALLEL]

**Phase:** 1
**File:** `src/constants/common.js`
**Depends on:** TASK-01 tests passing
**Covers:** FR-3, AC-3

Add the following export to `src/constants/common.js`:

```js
export const passwordRegExp = /^(?=.*[0-9])(?=.*[!@#$%^&*])[a-zA-Z0-9!@#$%^&*]{8,}$/;
```

Verify TASK-01 tests pass after this change.

---

## TASK-03 — [TEST] Auth context register action creator [PARALLEL]

**Phase:** 2
**File:** `src/contexts/auth.test.js` (new)
**Covers:** FR-4, FR-5, AC-5, AC-7, Story 2, Story 4

Write unit tests for the `register` action creator and new reducer cases:

- Happy path: `register` with valid unique data dispatches REGISTER_REQUEST then REGISTER_SUCCESS, persists to localStorage, returns `{ success: true }`
- Happy path: reducer handles REGISTER_REQUEST — sets `isRegistering: true`, `registrationSuccess: false`
- Happy path: reducer handles REGISTER_SUCCESS — sets `isRegistering: false`, `registrationSuccess: true`
- Happy path: reducer handles REGISTER_FAILURE — sets `isRegistering: false`, `registrationError` to payload
- Negative path: duplicate email (same case) returns `{ success: false, error: "Email address is already in use." }` and dispatches REGISTER_FAILURE
- Negative path: duplicate email (different case, e.g. "User@Example.com" vs "user@example.com") is caught — case-insensitive check
- Edge case: `registeredUsers` key does not exist in localStorage — initializes as empty array
- Edge case: localStorage throws — dispatches REGISTER_FAILURE and returns error

---

## TASK-04 — [IMPL] Extend auth context with register support

**Phase:** 2
**File:** `src/contexts/auth.jsx`
**Depends on:** TASK-03 tests passing
**Covers:** FR-4, FR-5, AC-5, AC-7

1. Extend `initialState` to add `isRegistering: false`, `registrationSuccess: false`, `registrationError: null`
2. Add `REGISTER_REQUEST`, `REGISTER_SUCCESS`, `REGISTER_FAILURE` cases to the reducer
3. Export new `register` action creator function

Verify TASK-03 tests pass after this change.

---

## Story 2 Checkpoint
- `register` action creator persists to localStorage under `registeredUsers`
- REGISTER_SUCCESS sets `registrationSuccess: true`
- Duplicate email returns error (case-insensitive)

---

## TASK-05 — [TEST] RegisterPage component rendering and validation [PARALLEL]

**Phase:** 3
**File:** `src/pages/register.test.jsx` (new)
**Covers:** FR-1, FR-2, FR-3, FR-5, AC-1, AC-2, AC-3, AC-4, AC-8, Story 1, Story 3, Story 5

Write rendering and validation tests for `RegisterPage`:

- Happy path: renders form with Full Name, Email, Password, Confirm Password fields and a submit button
- Happy path: renders "Create Account" heading
- Happy path: renders "Sign In" link
- Negative path: submitting empty form shows required error messages for all four fields
- Negative path: entering a password shorter than 8 characters shows password validation error
- Negative path: entering a password without a digit shows password validation error
- Negative path: entering a password without a special character shows password validation error
- Negative path: entering non-matching confirm password shows "Passwords do not match." error
- Edge case: entering an invalid email format shows email validation error
- Edge case: full name with 1 character shows "Full name must be at least 2 characters." error
- Edge case: full name with exactly 2 characters passes validation

---

## TASK-06 — [TEST] RegisterPage submission — success and duplicate email [PARALLEL]

**Phase:** 3
**File:** `src/pages/register.test.jsx` (extend)
**Covers:** FR-4, FR-5, AC-5, AC-6, AC-7, Story 2, Story 4

Write submission and integration tests for `RegisterPage`:

- Happy path: valid form submission calls `register` action creator and shows success message
- Happy path: success message contains confirmation text
- Happy path: after successful submission, `history.push("/auth")` is called after 2 seconds (use fake timers)
- Negative path: duplicate email submission shows inline error on the email field
- Negative path: general registration failure shows submit error message
- Edge case: success message has `role="alert"` for accessibility
- Edge case: error message has `role="alert"` for accessibility

---

## TASK-07 — [IMPL] Create RegisterPage component

**Phase:** 3
**File:** `src/pages/register.jsx` (new)
**Depends on:** TASK-05, TASK-06 tests passing; TASK-02, TASK-04 complete
**Covers:** FR-1, FR-2, FR-3, FR-4, FR-5, AC-1 through AC-8, Story 1 through Story 5

Create `src/pages/register.jsx` with:
- `RegisterSchema` Yup object with fullName, email, password, confirmPassword validation
- `RegisterPage` functional component using `useContext(AuthDispatchContext)`, `useHistory`, `useState`
- Formik form with `Field` + `component={Input}` for all four fields
- `onSubmit` handler calling `register` action creator, handling success and error paths
- Success message displayed inline with `role="alert"`
- Submit error displayed inline with `role="alert"`
- "Sign In" link navigating to `/auth`

Verify TASK-05 and TASK-06 tests pass after this change.

---

## Story 3 Checkpoint
- Password validation errors shown inline for: too short, no digit, no special char
- Confirm password mismatch error shown inline

## Story 5 Checkpoint
- All four required fields show inline error when submitted empty

---

## TASK-08 — [TEST] App routing — /register route exists [PARALLEL]

**Phase:** 4
**File:** `src/App.test.js` (extend) or `src/App.routing.test.js` (new)
**Covers:** FR-1, AC-1, Story 1

Write routing tests:

- Happy path: navigating to `/register` renders `RegisterPage` component
- Happy path: navigating to `/auth` renders `AuthPage` component (regression)
- Edge case: navigating to `/` renders `HomePage` (regression — no regression introduced)

---

## TASK-09 — [IMPL] Add /register route to App.js

**Phase:** 4
**File:** `src/App.js`
**Depends on:** TASK-08 tests passing; TASK-07 complete
**Covers:** FR-1, AC-1, Story 1

1. Import `RegisterPage` from `"pages/register"`
2. Add `<RouteWrapper path="/register" component={RegisterPage} layout={AuthLayout} />` inside `<Switch>`, before the `/auth` route

Verify TASK-08 tests pass after this change.

---

## Story 1 Checkpoint
- `/register` route renders `RegisterPage` wrapped in `AuthLayout`

---

## TASK-10 — [TEST] goToRegister navigation wiring [PARALLEL]

**Phase:** 5
**File:** `src/pages/auth.test.jsx` (new or extend)
**Covers:** FR-1, AC-1, Story 1

Write navigation tests for `AuthPage`:

- Happy path: clicking "Sign Up Now!" calls `history.push("/register")`
- Negative path: clicking "Sign Up Now!" does NOT navigate to `/#` (no default anchor behavior)
- Edge case: `console.log` debug statement is not present in auth.jsx (code quality check via source inspection)

---

## TASK-11 — [IMPL] Wire goToRegister and remove console.log in auth.jsx

**Phase:** 5
**File:** `src/pages/auth.jsx`
**Depends on:** TASK-10 tests passing
**Covers:** FR-1, AC-1, Story 1

1. Replace no-op `goToRegister` stub with `history.push("/register")`
2. Remove `console.log("location => ", location)` on line 19

Verify TASK-10 tests pass after this change.

---

## TASK-12 — [IMPL] Create _register.scss and update _index.scss [PARALLEL]

**Phase:** 6
**File:** `src/assets/scss/pages/_register.scss` (new), `src/assets/scss/pages/_index.scss` (modify)
**Covers:** Visual presentation of RegisterPage

1. Create `src/assets/scss/pages/_register.scss` with styles for `.register-title`, `.register-success`, `.register-error`
2. Add `@import "register";` to `src/assets/scss/pages/_index.scss`

No unit tests required for SCSS. Visual regression verified manually.

---

## Definition of Done

- [ ] All new tests pass (TASK-01, TASK-03, TASK-05, TASK-06, TASK-08, TASK-10)
- [ ] All existing tests continue to pass (no regression)
- [ ] Application builds without errors or warnings (`npm run build`)
- [ ] `passwordRegExp` exported from `src/constants/common.js`
- [ ] Auth context handles REGISTER_REQUEST, REGISTER_SUCCESS, REGISTER_FAILURE
- [ ] `register` action creator exported from `src/contexts/auth.jsx`
- [ ] `src/pages/register.jsx` created with full Formik + Yup form
- [ ] `/register` route added to `src/App.js` using `AuthLayout`
- [ ] `goToRegister` in `src/pages/auth.jsx` navigates to `/register`
- [ ] `console.log` removed from `src/pages/auth.jsx`
- [ ] `src/assets/scss/pages/_register.scss` created
- [ ] `@import "register"` added to `src/assets/scss/pages/_index.scss`
- [ ] Duplicate email check is case-insensitive
- [ ] Success message displayed and redirect to `/auth` after 2 seconds
