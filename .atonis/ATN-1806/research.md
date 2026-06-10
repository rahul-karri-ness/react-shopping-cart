# ATN-1806 - Research and Technical Analysis

## Tech Stack

| Technology | Version | Notes |
|-----------|---------|-------|
| React | 17.0.2 | Functional components with hooks throughout |
| Formik | 2.2.6 | Form state management; used in `src/pages/auth.jsx` |
| Yup | 0.32.9 | Schema-based validation; used in `src/pages/auth.jsx` |
| React Router | 5.2.0 | `useHistory`, `useLocation` hooks; `Switch`, `Route`, `Redirect` |
| Context API + useReducer | React 17 built-in | Auth state managed in `src/contexts/auth.jsx` |
| SCSS (node-sass) | 5.0.0 | Page-level partials under `src/assets/scss/pages/` |
| classnames | 2.2.6 | Conditional CSS class utility used in Input component |
| lodash.get | 4.4.2 | Safe property access used in auth context and pages |

**No new npm dependencies are required.** All needed libraries are already installed.

---

## Relevant Existing Files

### src/pages/auth.jsx (lines 1-90)
- The existing login page. Uses `Formik`, `Form`, `Field` from formik; `useHistory`, `useLocation` from react-router-dom; `Yup` for schema validation.
- **Key finding (line 24-26):** `goToRegister` is a no-op stub:
  ```js
  const goToRegister = (e) => {
    e.preventDefault();
  };
  ```
  This must be wired to `history.push("/register")` to enable navigation to the registration page.
- The `LoginSchema` (lines 9-12) is the pattern to follow for `RegisterSchema`.
- The `Field` + `component={Input}` pattern (lines 56-61) must be replicated in `RegisterPage`.

### src/contexts/auth.jsx (lines 1-86)
- Exports `AuthStateContext`, `AuthDispatchContext`, `signIn`, `signOut`, and `AuthProvider`.
- Reducer handles: `LOGIN_REQUEST`, `LOGIN_SUCCESS`, `LOGIN_FAILURE`, `LOGOUT_SUCCESS`.
- **Must extend** to handle: `REGISTER_REQUEST`, `REGISTER_SUCCESS`, `REGISTER_FAILURE`.
- Initial state (lines 5-9): `{ isLoggedIn, user, isLoggingIn }`.
- Must add `isRegistering` and `registrationSuccess` flags to state.
- `signIn` (lines 47-55) is the pattern to follow for the `register` action creator.
- localStorage is used directly (line 48): `localStorage.setItem("user", ...)`.
- Registration data must be persisted to localStorage under key `registeredUsers` as an array.

### src/constants/common.js (line 1)
- Currently exports only `phoneRegExp`.
- Must add `passwordRegExp` constant:
  ```js
  export const passwordRegExp = /^(?=.*[0-9])(?=.*[!@#$%^&*])[a-zA-Z0-9!@#$%^&*]{8,}$/;
  ```

### src/components/core/form-controls/Input.jsx (lines 1-38)
- Reusable controlled input component accepting `field` and `form` props from Formik's `Field`.
- Renders `.form-group`, `.form-control`, `.invalid-feedback` using `classnames`.
- **No changes needed.** Use as-is in `RegisterPage`.

### src/layouts/AuthLayout.jsx (lines 1-23)
- Wraps auth pages in `.auth-container > .wrapper` with brand logo.
- **No changes needed.** The `/register` route must use `AuthLayout` as its layout.

### src/layouts/RouteWrapper.jsx (lines 1-36)
- Generic route wrapper accepting `component`, `layout`, and `isPrivate` props.
- Handles private route redirects to `/auth`.
- **No changes needed.** Use as-is for the `/register` route.

### src/App.js (lines 1-51)
- Defines all application routes using `Switch` and `RouteWrapper`.
- **Must add** a new `RouteWrapper` for path `/register` using `RegisterPage` and `AuthLayout`.

### src/assets/scss/pages/_index.scss (line 1-3)
- Imports `home`, `auth`, `checkout` page SCSS partials.
- **Must add** `@import "register";` to include the new register page styles.

### src/assets/scss/pages/_auth.scss (lines 1-26)
- Defines `.auth-container`, `.wrapper`, `.auth-brand`, `.auth-button` styles.
- `.auth-button` and `.auth-container` styles are shared and reused by the register page via the same class names.

### src/assets/scss/components/_form-control.scss (lines 1-32)
- Defines `.form-group`, `.form-control`, `.invalid-feedback`, `.field-group`.
- Used globally. No changes needed.

### src/assets/scss/base/_variables.scss (lines 1-13)
- SCSS variables: `$primary-green`, `$red`, `$white`, `$gray-light-bg`, etc.
- Available for use in `_register.scss`.

### src/hooks/useLocalStorage.js (lines 1-28)
- Custom hook wrapping `window.localStorage` with JSON parse/stringify.
- Used in `AuthProvider` for persisting user state.
- Registration action creator will use `localStorage` directly (consistent with `signIn` pattern).

---

## Patterns to Follow

1. **Formik + Yup validation pattern** — mirror `src/pages/auth.jsx` exactly:
   - Define a Yup schema constant above the component
   - Use `<Formik initialValues={...} validationSchema={...} onSubmit={...}>`
   - Render fields using `<Field name="..." type="..." component={Input} />`

2. **Context action creator pattern** — mirror `signIn` in `src/contexts/auth.jsx`:
   - Export a named function `register(dispatch, userData)`
   - Dispatch `REGISTER_REQUEST` before async work, `REGISTER_SUCCESS` or `REGISTER_FAILURE` after

3. **Reducer extension pattern** — add new cases to the existing `switch` in `src/contexts/auth.jsx`:
   - `REGISTER_REQUEST`: set `isRegistering: true`, `registrationSuccess: false`
   - `REGISTER_SUCCESS`: set `isRegistering: false`, `registrationSuccess: true`
   - `REGISTER_FAILURE`: set `isRegistering: false`, `registrationSuccess: false`, `registrationError: action.payload.error`

4. **Route registration pattern** — mirror existing `RouteWrapper` entries in `src/App.js`:
   ```jsx
   <RouteWrapper path="/register" component={RegisterPage} layout={AuthLayout} />
   ```

5. **SCSS partial pattern** — create `src/assets/scss/pages/_register.scss` and import it in `_index.scss`.

---

## Files to Create

| File | Purpose |
|------|---------|
| `src/pages/register.jsx` | New RegisterPage component |
| `src/assets/scss/pages/_register.scss` | Register page styles |

## Files to Modify

| File | Change |
|------|--------|
| `src/constants/common.js` | Add `passwordRegExp` export |
| `src/contexts/auth.jsx` | Add `REGISTER_REQUEST/SUCCESS/FAILURE` cases to reducer; add `register` action creator; extend `initialState` |
| `src/App.js` | Import `RegisterPage`; add `/register` route |
| `src/pages/auth.jsx` | Wire `goToRegister` to `history.push("/register")` |
| `src/assets/scss/pages/_index.scss` | Add `@import "register";` |

---

## Risks and Unknowns

| Risk | Mitigation |
|------|-----------|
| localStorage `registeredUsers` may not exist on first registration | Initialize as empty array with fallback: `JSON.parse(localStorage.getItem("registeredUsers") || "[]")` |
| Case-insensitive duplicate email check requires normalization | Normalize to lowercase before comparison and storage |
| `console.log` debug statement on line 19 of `src/pages/auth.jsx` | Remove during `goToRegister` wiring to keep code clean |
| Auth context `initialState` does not include registration fields | Extend `initialState` with `isRegistering: false`, `registrationSuccess: false`, `registrationError: null` |
| Reducer `default` case throws on unknown action types | New action types must be added before deploying; tests must cover them |
