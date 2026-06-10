# ATN-1806 - Account Creation - User Registration
## Task Breakdown (TDD Order)

**Ticket:** ATN-1806

---

## Task List

Tasks are ordered so that each test task immediately precedes the implementation task it covers.
Tasks marked [PARALLEL] can be executed concurrently with other [PARALLEL] tasks once their prerequisites are met.

---

### TASK-01 â€” [PARALLEL] Write tests for passwordRegExp constant

**Type:** Test
**File:** `src/constants/common.test.js` (create new)
**Covers:** Phase 1 â€” passwordRegExp constant
**Prerequisite:** None

Write unit tests for the `passwordRegExp` regular expression before it is implemented.

Test cases:
- Valid password: `"Secret1!"` â€” must match
- Valid password: `"Abcdef1@"` â€” must match
- Too short (7 chars): `"Abc1!ab"` â€” must not match
- No digit: `"Abcdef!!"` â€” must not match
- No special character: `"Abcdef12"` â€” must not match
- Empty string: `""` â€” must not match
- Exactly 8 chars with digit and special: `"Abcde1!a"` â€” must match

Definition of done:
- [ ] All 7 test cases written and failing (red) before implementation

---

### TASK-02 â€” [PARALLEL] Implement passwordRegExp constant

**Type:** Implementation
**File:** `src/constants/common.js` (modify)
**Covers:** Phase 1 â€” passwordRegExp constant
**Prerequisite:** TASK-01

Add the following export to `src/constants/common.js`:

```js
export const passwordRegExp = /^(?=.*[0-9])(?=.*[!@#$%^&*()_+\-=[\]{};':"\\|,.<>/?]).{8,}$/;
```

Definition of done:
- [ ] `passwordRegExp` exported from `src/constants/common.js`
- [ ] All TASK-01 tests pass (green)
- [ ] Existing `phoneRegExp` export unchanged

---

### TASK-03 â€” Write tests for register action creator

**Type:** Test
**File:** `src/contexts/auth.test.js` (create new)
**Covers:** Phase 2 â€” auth context extension
**Prerequisite:** TASK-02

Write unit tests for the `register(dispatch, userData)` action creator before it is implemented.

Test cases:
- Happy path: new email â€” dispatches REGISTER_REQUEST then REGISTER_SUCCESS, saves user to localStorage, returns `{ success: true }`
- Duplicate email (exact match): dispatches REGISTER_REQUEST then REGISTER_FAILURE, does not modify localStorage, returns `{ success: false, error: "Email is already registered" }`
- Duplicate email (case-insensitive): `"User@Example.com"` vs stored `"user@example.com"` â€” treated as duplicate
- Empty localStorage `registeredUsers`: defaults to `[]`, registers successfully
- Multiple existing users: new unique email appended to existing array

Definition of done:
- [ ] All 5 test cases written and failing (red) before implementation
- [ ] `localStorage` mocked in test setup using `jest.spyOn(Storage.prototype, 'getItem')` and `jest.spyOn(Storage.prototype, 'setItem')`

---

### TASK-04 â€” Extend auth context with registration actions

**Type:** Implementation
**File:** `src/contexts/auth.jsx` (modify)
**Covers:** Phase 2 â€” auth context extension
**Prerequisite:** TASK-03

Changes:
1. Add `isRegistering: false` to `initialState`
2. Add `REGISTER_REQUEST`, `REGISTER_SUCCESS`, `REGISTER_FAILURE` cases to the reducer
3. Export new `register(dispatch, userData)` action creator

Definition of done:
- [ ] `initialState` contains `isRegistering: false`
- [ ] Reducer handles all three new action types without throwing
- [ ] `register` function exported and handles duplicate check with case-insensitive comparison
- [ ] All TASK-03 tests pass (green)
- [ ] Existing LOGIN_*, LOGOUT_* reducer cases unchanged

---

### TASK-05 â€” Write tests for RegisterPage component

**Type:** Test
**File:** `src/pages/register.test.jsx` (create new)
**Covers:** Phase 3 â€” RegisterPage component
**Prerequisite:** TASK-04

Write integration tests for the `RegisterPage` component using `@testing-library/react`.

Test cases:
- Renders form with full name, email, and password fields
- Renders a submit button
- Shows required field errors when form is submitted empty
- Shows password validation error when password does not meet requirements
- Shows "Email is already registered" error when duplicate email submitted
- Shows success message on valid submission
- Redirects to `/auth` after 2 seconds on successful registration (use `jest.useFakeTimers`)
- "Already have an account?" link navigates to `/auth`

Definition of done:
- [ ] All 8 test cases written and failing (red) before implementation
- [ ] Tests use `MemoryRouter` and mock `AuthDispatchContext`
- [ ] `jest.useFakeTimers()` used for redirect delay test

---

### TASK-06 â€” Create RegisterPage component

**Type:** Implementation
**File:** `src/pages/register.jsx` (create new)
**Covers:** Phase 3 â€” RegisterPage component
**Prerequisite:** TASK-05

Create the RegisterPage component following the Formik + Yup + Field + Input pattern.

Key implementation details:
- `RegisterSchema` validates `fullName` (min 2 chars, required), `email` (valid email format, required), `password` (matches `passwordRegExp`, required)
- `onSubmit` calls `register(authDispatch, values)`
- On `success: true`: set local state `registered: true`, call `setTimeout(() => history.push("/auth"), 2000)`
- On `success: false`: call `setFieldError("email", result.error)`, call `setSubmitting(false)`
- Renders a success message when `registered` state is `true`
- Renders a link to `/auth` with text "Already have an account? Sign in"

Definition of done:
- [ ] Component renders without errors
- [ ] All TASK-05 tests pass (green)
- [ ] No `console.log` or debug statements
- [ ] Password field uses `type="password"`
- [ ] Submit button disabled while `isSubmitting` is true

---

### TASK-07 â€” [PARALLEL] Write tests for /register route in App.js

**Type:** Test
**File:** `src/App.test.js` (modify)
**Covers:** Phase 4 â€” /register route
**Prerequisite:** TASK-06

Add a test that verifies the `/register` route renders the `RegisterPage` component.

Test cases:
- Navigating to `/register` renders the registration form heading
- Navigating to `/auth` still renders the login form (regression check)

Definition of done:
- [ ] Both test cases written and failing (red) before route is added
- [ ] Tests use `MemoryRouter` with `initialEntries` prop

---

### TASK-08 â€” [PARALLEL] Add /register route in App.js

**Type:** Implementation
**File:** `src/App.js` (modify)
**Covers:** Phase 4 â€” /register route
**Prerequisite:** TASK-07

Add the following inside the `Switch` block, after the `/auth` `RouteWrapper`:

```jsx
import RegisterPage from "pages/register";

<RouteWrapper
  path="/register"
  component={RegisterPage}
  layout={AuthLayout}
/>
```

Definition of done:
- [ ] `/register` route renders `RegisterPage` wrapped in `AuthLayout`
- [ ] All TASK-07 tests pass (green)
- [ ] Existing routes (`/`, `/checkout`, `/auth`) unchanged

---

### TASK-09 â€” Write tests for goToRegister navigation wiring

**Type:** Test
**File:** `src/pages/auth.test.jsx` (create new)
**Covers:** Phase 5 â€” goToRegister navigation wiring
**Prerequisite:** TASK-08

Write tests for the `goToRegister` function in `src/pages/auth.jsx`.

Test cases:
- Clicking the "Register" link calls `history.push("/register")`
- Clicking the "Register" link does not navigate to any other route
- The "Register" link is present and visible on the auth page

Definition of done:
- [ ] All 3 test cases written and failing (red) before implementation
- [ ] `useHistory` mocked using `jest.mock("react-router-dom", ...)`

---

### TASK-10 â€” Wire goToRegister navigation stub

**Type:** Implementation
**File:** `src/pages/auth.jsx` (modify)
**Covers:** Phase 5 â€” goToRegister navigation wiring
**Prerequisite:** TASK-09

Replace the no-op body of `goToRegister` at line 24 with:

```jsx
const goToRegister = (e) => {
  e.preventDefault();
  history.push("/register");
};
```

Definition of done:
- [ ] `goToRegister` calls `history.push("/register")`
- [ ] All TASK-09 tests pass (green)
- [ ] No other changes to `src/pages/auth.jsx`

---

## Per-Story Checkpoints

| User Story | Tasks Covered        | Checkpoint                                                                 |
|------------|----------------------|----------------------------------------------------------------------------|
| US-1       | TASK-09, TASK-10     | Clicking "Register" on `/auth` navigates to `/register`                   |
| US-2       | TASK-05, TASK-06     | Valid form submission shows success message and redirects after 2 seconds  |
| US-3       | TASK-01, TASK-02, TASK-05, TASK-06 | Password errors shown inline for invalid passwords            |
| US-4       | TASK-03, TASK-04, TASK-05, TASK-06 | Duplicate/invalid email errors shown inline                   |
| US-5       | TASK-05, TASK-06     | Empty form submission shows required field errors inline                   |

---

## Definition of Done Checklist

- [ ] All new code has unit tests covering happy paths, negative paths, and at least 3 edge cases
- [ ] All tests in the app pass â€” not only the ones added
- [ ] The application builds without any errors or warnings
- [ ] No regression introduced â€” existing functionality is unaffected
- [ ] Code has been self-reviewed: correctness, performance, security, and maintainability
- [ ] No dead code, no duplicate code, commented-out blocks, or debug artifacts left behind
- [ ] New functions and complex logic have inline comments explaining purpose and intent
- [ ] No secrets, API keys, or sensitive data are hardcoded or logged
- [ ] New code follows the existing architectural patterns and conventions of the codebase
- [ ] Errors are handled gracefully with meaningful user-facing messages
- [ ] `passwordRegExp` covers all three security requirements (length, digit, special char)
- [ ] Duplicate email check is case-insensitive
- [ ] Redirect to `/auth` occurs after exactly 2 seconds on successful registration
