# Viksit Bharat Run 2026 — Registration Portal

Full replica of the IDY 2027 portal, rebranded for the **Viksit Bharat Run 2026** (Wednesday, 30 September 2026, Bangkok).

---

## Files

| File | Purpose |
|---|---|
| `index.html` | Public registration portal (multi-step form + pass) |
| `dashboard.html` | Live ops dashboard (password-protected) |
| `checkin.html` | Staff check-in tool (password-protected) |
| `admin.html` | Admin panel — config, full registrations table, CSV export |
| `faq.html` | Event guide — schedule, categories, FAQs, T-shirt sizes |

---

## Before deploying

### 1. Insert your Firebase config
In every HTML file, replace the placeholder `firebaseConfig` block with your actual keys from the `idy2027-dba28` Firebase project console:

```js
const firebaseConfig = {
  apiKey: "YOUR_ACTUAL_KEY",
  authDomain: "idy2027-dba28.firebaseapp.com",
  databaseURL: "https://idy2027-dba28-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId: "idy2027-dba28",
  storageBucket: "idy2027-dba28.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

### 2. Change the passwords
All passwords are hardcoded as a simple client-side gate (same approach as IDY 2027). Change these before deploying:

| Page | Variable | Default (change this) |
|---|---|---|
| `dashboard.html` | `checkAuth()` | `vbr2026` |
| `checkin.html` | `checkAuth()` | `vbr2026staff` |
| `admin.html` | `checkAuth()` | `vbr2026admin` |

### 3. Firebase database node
All data writes to the `vbr2026/` node (separate from `idy2027/`), so both portals coexist on the same Firebase project without conflict:

```
idy2027-dba28 (Firebase project)
├── idy2027/           ← IDY 2027 portal data
│   ├── config/
│   ├── registrations/
│   └── counter
└── vbr2026/           ← VBR 2026 portal data  ← NEW
    ├── config/
    │   ├── regOpen: true
    │   └── maxReg: 500
    ├── registrations/
    └── counter
```

### 4. Set the initial config node
In Firebase Console → Realtime Database, create:
```json
{
  "vbr2026": {
    "config": {
      "regOpen": true,
      "maxReg": 500
    }
  }
}
```

### 5. Firebase database rules
Add a rule for `vbr2026/config` to be publicly readable (matching the IDY 2027 pattern):

```json
{
  "rules": {
    "idy2027": { ... existing rules ... },
    "vbr2026": {
      "config": { ".read": true, ".write": "auth != null" },
      "registrations": { ".read": "auth != null", ".write": "auth != null" },
      "counter": { ".read": true, ".write": "auth != null" }
    }
  }
}
```

> Note: Since the password gate is client-side, the current setup allows authenticated writes. For a production event with sensitive data, add proper Firebase Auth sign-in (same as the IDY 2027 auth layer).

### 6. Deploy
```bash
firebase deploy --only hosting
```
Or drag the folder into Firebase Hosting in the console.

---

## What to fill in once you have venue details
- Exact venue / address (faq.html and checkin.html header)
- Nearest BTS station
- Parking details
- Ambassador's name for the flag-off line (faq.html)
- Embassy contact email (faq.html)
