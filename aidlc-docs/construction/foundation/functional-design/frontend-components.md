# Frontend Components — Foundation Unit

## Component Hierarchy

```
App
├── AuthProvider (Context)
│   ├── ThemeProvider (Context)
│   │   ├── SyncProvider (Context)
│   │   │   ├── Router
│   │   │   │   ├── PublicRoutes
│   │   │   │   │   ├── LoginPage
│   │   │   │   │   ├── RegisterPage
│   │   │   │   │   ├── ForgotPasswordPage
│   │   │   │   │   └── ConfirmPasswordPage
│   │   │   │   ├── OnboardingRoutes
│   │   │   │   │   └── ProfileSetupWizard
│   │   │   │   └── ProtectedRoutes
│   │   │   │       ├── AppLayout
│   │   │   │       │   ├── NavigationBar
│   │   │   │       │   ├── SyncStatusIndicator
│   │   │   │       │   └── PageContent (outlet)
│   │   │   │       └── [Feature modules render here]
│   │   │   └── ErrorBoundary
```

---

## Auth Pages

### LoginPage

| Prop/State | Type | Description |
|------------|------|-------------|
| email | string (state) | Email input value |
| password | string (state) | Password input value |
| error | string | null (state) | Login error message |
| isLoading | boolean (state) | Loading state during API call |

**User Interactions:**
- Enter email and password → submit form
- Click "Forgot Password?" → navigate to ForgotPasswordPage
- Click "Create Account" → navigate to RegisterPage

**API Integration:** `POST /auth/login`

---

### RegisterPage

| Prop/State | Type | Description |
|------------|------|-------------|
| email | string (state) | Email input |
| password | string (state) | Password input |
| confirmPassword | string (state) | Password confirmation |
| error | string | null (state) | Registration error |
| isLoading | boolean (state) | Loading state |
| step | "form" | "verify" (state) | Current step (form or verification code) |
| verificationCode | string (state) | 6-digit code input |

**Form Validation (client-side):**
- Email: valid format (regex)
- Password: 8+ chars, uppercase, lowercase, number, special char
- Confirm password: must match password
- All fields required

**User Interactions:**
- Fill form → submit → show verification code input
- Enter code → submit → redirect to Profile Setup
- Click "Already have an account?" → navigate to LoginPage

**API Integration:** `POST /auth/register`, `POST /auth/confirm`

---

### ForgotPasswordPage

| Prop/State | Type | Description |
|------------|------|-------------|
| email | string (state) | Email input |
| step | "email" | "code" (state) | Current step |
| code | string (state) | Reset code |
| newPassword | string (state) | New password |
| confirmPassword | string (state) | Confirm new password |

**API Integration:** `POST /auth/forgot-password`, `POST /auth/confirm-password`

---

### ProfileSetupWizard

Multi-step wizard for initial profile configuration.

| Step | Fields | Required |
|------|--------|----------|
| 1. Goals | dailyCalorieTarget, weightGoal | dailyCalorieTarget required |
| 2. Body Info | height, weight, age, activityLevel | All optional |
| 3. Preferences | dietaryPreferences[], cookingSkillLevel | All optional |
| 4. Macros | macroTargets (protein, carbs, fat) | Optional (defaults provided) |

**State:**

| State       | Type                 | Description           |
| ----------- | -------------------- | --------------------- |
| currentStep | number (1-4)         | Active wizard step    |
| formData    | Partial<UserProfile> | Accumulated form data |
| isLoading   | boolean              | Saving state          |

**User Interactions:**

- Fill fields → Next step
- Back button → previous step
- Skip button → skip optional fields, move to next
- Finish → save profile, redirect to Dashboard

**API Integration:** `PUT /auth/profile`

---

## Shared UI Components

### NavigationBar

| Prop | Type | Description |
|------|------|-------------|
| currentPath | string | Active route for highlighting |

**Items:**
- Dashboard (home icon)
- Log Meal (plus icon)
- Ingredients (fridge icon)
- Recipes (book icon)
- Profile (user icon)

**Behavior:** Bottom navigation on mobile, side navigation on desktop.

---

### SyncStatusIndicator

| State | Display | Color |
|-------|---------|-------|
| idle | Hidden (or subtle checkmark) | Green |
| syncing | Spinning icon + "Syncing..." | Blue |
| error | Warning icon + "Sync failed" (tap to retry) | Red |
| offline | Cloud-off icon + "Offline" | Gray |

**Props:**

| Prop | Type | Source |
|------|------|--------|
| syncStatus | SyncState | From SyncProvider context |

---

### LoadingSpinner

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| size | "sm" | "md" | "lg" | "md" | Spinner size |
| fullPage | boolean | false | Center in viewport |

---

### ErrorMessage

| Prop | Type | Description |
|------|------|-------------|
| message | string | Error text to display |
| onRetry | () => void | null | Optional retry callback |

---

### Toast

| Prop | Type | Description |
|------|------|-------------|
| message | string | Toast text |
| type | "success" | "error" | "info" | "warning" | Toast variant |
| duration | number | Auto-dismiss time (ms), default 3000 |

---

### ImageUpload

| Prop | Type | Description |
|------|------|-------------|
| onImageSelected | (file: File) => void | Callback with selected file |
| maxSizeMB | number | Maximum file size (default 10) |
| accept | string | Accepted formats (default "image/*") |
| disabled | boolean | Disable when offline |

---

### SearchInput

| Prop | Type | Description |
|------|------|-------------|
| placeholder | string | Input placeholder text |
| onSearch | (query: string) => void | Debounced search callback |
| results | SearchResult[] | Autocomplete results |
| onSelect | (item: SearchResult) => void | Selection callback |
| debounceMs | number | Debounce delay (default 300) |

---

## Context Providers

### AuthProvider

| Value | Type | Description |
|-------|------|-------------|
| user | User | null | Current authenticated user |
| isAuthenticated | boolean | Whether user is logged in |
| isLoading | boolean | Auth state loading |
| login | (email, password) => Promise | Login function |
| register | (email, password) => Promise | Register function |
| logout | () => Promise | Logout function |
| refreshSession | () => Promise | Refresh tokens |

---

### SyncProvider

| Value | Type | Description |
|-------|------|-------------|
| syncState | SyncState | Current sync status |
| isOnline | boolean | Network connectivity |
| pendingChanges | number | Unsynced mutation count |
| triggerSync | () => Promise | Manual sync trigger |
| queueMutation | (mutation) => void | Add to offline queue |

---

### ThemeProvider

| Value | Type | Description |
|-------|------|-------------|
| theme | "light" | "dark" | Current theme |
| toggleTheme | () => void | Switch theme |
