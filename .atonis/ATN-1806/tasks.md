# ATN-1806 — Account Creation - User Registration
## Task Breakdown (TDD Order)

---

### Overview

Tasks are ordered so each test task immediately precedes the implementation task it covers. Tasks marked [PARALLEL] can be executed concurrently once their prerequisites are complete. All tasks follow the Definition of Done checklist at the end of this document.

---

### TASK-01 — [TEST] Write tests for passwordRegExp constant

**Type:** Test
**Phase:** 1
**Files:** `src/constants/common.test.js` (create)

Write unit tests for the `passwordRegExp` constant before it is implemented.

Test cases:
- Accepts a valid password with digit and special character (e.g., `"Secure1!"`)
- Rejects a password shorter than 8 characters (e.g., `"Ab1!"`)
- Rejects a password with no digit (e.g., `"Secure!!"`)
- Rejects a password with no special character (e.g., `"Secure11"`)
- Rejects an empty string
- Accepts a password with multiple special characters and digits (e.g., `"P@ssw0rd#2"`)

**Acceptance:** All tests fail (red) before TASK-02 is implemented.

---

### TASK-02 — [IMPL] Add passwordRegExp to constants [PARALLEL]

**Type:** Implementation
**Phase:** 1
**Depends on:** TASK-01
**Files:** `src/constants/common.js` (modify)

Add the `passwordRegExp` export to `src/constants/common.js`.

```js
// Enforces: min 8 chars, at least 1 digit, at least 1 special character
export const passwordRegExp = /^(?=.*[0-9])(?=.*[!@#$%^&*()_+\-=[\]{};':"\\|,.<>/?]).{8,}$/;
```

**Acceptance:** All TASK-01 tests pass (green).

---

### TASK-03 — [TEST] Write tests for auth context register action

**Type:** Test
**Phase:** 2
**Files:** `src/contexts/auth.test.js` (create)

Write unit tests for the `register` action creator and the new reducer cases before they are implemented.

Test cases:
- `REGISTER_REQUEST` sets `isRegistering: true` and `registrationError: null`
- `REGISTER_SUCCESS` sets `isRegistering: false` and `registrationError: null`
- `REGISTER_FAILURE` sets `isRegistering: false` and `registrationError` to the error string
- `register()` with a new email appends user to `registeredUsers` in `localStorage` and returns `{ success: true }`
- `register()` with a duplicate email (exact match) dispatches `REGISTER_FAILURE` and returns `{ success: false }`
- `register()` with a duplicate email (different case) dispatches `REGISTER_FAILURE` (case-insensitive check per A-3)
- `register()` when `localStorage` throws dispatches `REGISTER_FAILURE` and returns `{ success: false }`

**Acceptance:** All tests fail (red) before TASK-04 is implemented.

---

### TASK-04 — [IMPL] Extend auth context with registration actions [PARALLEL]

**Type:** Implementation
**Phase:** 2
**Depends on:** TASK-03
**Files:** `src/contexts/auth.jsx` (modify)

1. Add `isRegistering: false` and `registrationError: null` to `initialState`.
2. Add `REGISTER_REQUEST`, `REGISTER_SUCCESS`, `REGISTER_FAILURE` cases to the reducer.
3. Export the `register` action creator.

**Acceptance:** All TASK-03 tests pass (green). Existing LOGIN/LOGOUT tests unaffected.

---

### Story 1 Checkpoint — Form Access
_Prerequisite for Story 1: TASK-02 and TASK-04 complete._

---

### TASK-05 — [TEST] Write tests for RegisterPage component

**Type:** Test
**Phase:** 3
**Depends on:** TASK-02, TASK-04
**Files:** `src/pages/register.test.jsx` (create)

Write unit and integration tests for `RegisterPage` before it is implemented.

**Story 1 — Form Access:**
- Renders the registration form with Full Name, Email Address, and Password fields
- Renders a "Create Account" submit button

**Story 2 — Successful Registration:**
- Submitting with valid inputs calls `register` and displays the success message
- Success message contains text confirming registration
- After success, `history.push("/auth")` is called (mock timer / fake timers)

**Story 3 — Password Validation Failure:**
- Submitting with a password shorter than 8 characters shows inline password error
- Submitting with a password missing a digit shows inline password error
- Submitting with a password missing a special character shows inline password error

**Story 4 — Duplicate or Invalid Email:**
- Submitting with an invalid email format shows inline email error
- Submitting with a duplicate email (mocked `register` returning `{ success: false }`) shows form-level error message

**Story 5 — Empty Required Fields:**
- Submitting with all fields empty shows error messages for fullName, email, and password fields
- Submitting with only fullName empty shows fullName error
- Submitting with only email empty shows email error
- Submitting with only password empty shows password error

**Edge Cases:**
- Full name with exactly 1 character shows validation error (min 2 chars per A-6)
- Full name with exactly 2 characters passes validation
- Password with exactly 7 characters fails validation
- Password with exactly 8 characters, a digit, and a special character passes validation

**Acceptance:** All tests fail (red) before TASK-06 is implemented.

---

### TASK-06 — [IMPL] Create RegisterPage component

**Type:** Implementation
**Phase:** 3
**Depends on:** TASK-05
**Files:** `src/pages/register.jsx` (create)

Implement `RegisterPage` as specified in plan.md Phase 3. Key requirements:
- Functional component using `useState`, `useEffect`, `useContext`, `useHistory`
- Formik + Yup + Field + Input pattern
- `RegisterSchema` validates `fullName`, `email`, `password`
- Calls `register(authDispatch, userData)` on submit
- Shows success state and redirects to `/auth` after 2000 ms via `setTimeout` with cleanup
- Shows form-level `formError` for duplicate email
- ARIA attributes: `aria-label` on form, `role="alert"` and `aria-live` on error/success elements
- No `console.log` calls

**Acceptance:** All TASK-05 tests pass (green).

---

### Story 2 Checkpoint — Successful Registration
_Prerequisite: TASK-06 complete. Verify Story 2 tests pass._

---

### Story 3 Checkpoint — Password Validation Failure
_Prerequisite: TASK-06 complete. Verify Story 3 tests pass._

---

### Story 4 Checkpoint — Duplicate or Invalid Email
_Prerequisite: TASK-06 complete. Verify Story 4 tests pass._

---

### Story 5 Checkpoint — Empty Required Fields
_Prerequisite: TASK-06 complete. Verify Story 5 tests pass._

---

### TASK-07 — [TEST] Write tests for /register route in App.js [PARALLEL]

**Type:** Test
**Phase:** 4
**Depends on:** TASK-06
**Files:** `src/App.test.js` (modify)

Add a test that navigates to `/register` and confirms the `RegisterPage` renders within `AuthLayout`.

Test cases:
- Navigating to `/register` renders the registration form
- Navigating to `/register` renders within the `AuthLayout` wrapper (brand logo link present)

**Acceptance:** Tests fail (red) before TASK-08 is implemented.

---

### TASK-08 — [IMPL] Add /register route to App.js [PARALLEL]

**Type:** Implementation
**Phase:** 4
**Depends on:** TASK-07
**Files:** `src/App.js` (modify)

1. Import `RegisterPage` from `"pages/register"`.
2. Add `<RouteWrapper path="/register" component={RegisterPage} layout={AuthLayout} />` inside `<Switch>`.

**Acceptance:** All TASK-07 tests pass (green). Existing routes unaffected.

---

### TASK-09 — [TEST] Write tests for goToRegister navigation wiring [PARALLEL]

**Type:** Test
**Phase:** 5
**Depends on:** TASK-08
**Files:** `src/pages/auth.test.jsx` (create)

Write tests for the `goToRegister` navigation before the stub is wired.

Test cases:
- Clicking "Sign Up Now!" navigates to `/register`
- Clicking "Sign Up Now!" does not navigate to any other route
- `e.preventDefault()` is called (default navigation suppressed)

**Acceptance:** Navigation test fails (red) before TASK-10 is implemented.

---

### TASK-10 — [IMPL] Wire goToRegister navigation in auth.jsx

**Type:** Implementation
**Phase:** 5
**Depends on:** TASK-09
**Files:** `src/pages/auth.jsx` (modify)

Replace the no-op `goToRegister` stub with:

```jsx
const goToRegister = (e) => {
  e.preventDefault();
  history.push("/register");
};
```

**Acceptance:** All TASK-09 tests pass (green). Existing login tests unaffected.

---

### TASK-11 — [IMPL] Create _register.scss and update _index.scss [PARALLEL]

**Type:** Implementation
**Phase:** 6
**Depends on:** TASK-06
**Files:**
- `src/assets/scss/pages/_register.scss` (create)
- `src/assets/scss/pages/_index.scss` (modify)

Create `_register.scss` with `.register-success` and `.form-error` styles using existing SCSS variables (`$primary-green`, `$red`). Add `@import "register"` to `_index.scss`.

No separate test task — visual styling is validated by TASK-05 render tests confirming CSS class presence.

**Acceptance:** Application builds without SCSS errors. `.register-success` and `.form-error` classes are applied correctly.

---

### Definition of Done Checklist

- [ ] All new work has unit tests covering: happy paths, negative paths, and at least 3 edge cases
- [ ] All tests in the app pass — not only the ones added in this ticket
- [ ] The application builds without any errors or warnings
- [ ] No regression was introduced — existing functionality is unaffected
- [ ] Code has been self-reviewed: correctness, performance, security, and maintainability
- [ ] No dead code, no duplicate code, commented-out blocks, or debug artifacts left behind
- [ ] New functions and complex logic have inline comments explaining purpose and intent
- [ ] No secrets, API keys, or sensitive data are hardcoded or logged
- [ ] New code follows the existing architectural patterns and conventions of the codebase
- [ ] Errors are handled gracefully with meaningful user-facing messages
- [ ] `goToRegister` in `src/pages/auth.jsx` navigates to `/register` (Story 1)
- [ ] Successful registration shows confirmation message and redirects to `/auth` after 2 seconds (Story 2, A-5)
- [ ] Duplicate email check is case-insensitive (A-3)
