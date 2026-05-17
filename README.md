# Medical Office Package Tracker

A self-contained web app for medical office front-desk staff to log incoming deliveries — medications, lab specimens, refrigerated shipments, supplies, and everything in between. One HTML file, no install, no build step, optional real-time sync across devices.

![Built with vanilla JS](https://img.shields.io/badge/Built%20with-Vanilla%20JS-f7df1e?style=flat-square&logo=javascript&logoColor=black)
![No dependencies](https://img.shields.io/badge/Dependencies-None-brightgreen?style=flat-square)
![License: MIT](https://img.shields.io/badge/License-MIT-blue?style=flat-square)

---

## Features

- **Passcode lock** on entry — soft gate to prevent casual access
- **Live clock** and "Today's Deliveries" stat cards (today / cold storage / pending / completed)
- **One-tap quick presets** for the most common package types
- **Cold/Frozen visual highlights** so refrigerated items don't get missed
- **Pending bubbles to top** of the log automatically
- **One-click "Mark Put Away"** with auto-timestamp
- **Search, status filter, temperature filter**, archive toggle
- **Edit/archive/delete** with confirmation
- **Autocomplete** for staff names (ordered/accepted/put-away by)
- **CSV export & import** — round-trip backup that preserves IDs
- **Print-friendly view** that strips the chrome
- **Dark mode by default**, light-mode toggle, remembered per device
- **Real-time multi-device sync** when Firestore is configured (optional)
- **Offline-friendly** — keeps working without a connection, syncs when back
- **Touch-optimized** for tablets at the front desk

---

## Quick Start

### Option 1 — Local only (single device)

1. Download `index.html`.
2. Double-click to open in any modern browser, or host it via GitHub Pages / Netlify / Vercel.
3. Enter the passcode (**default: `1997`**) to unlock.
4. All data is saved in that browser's local storage. Use the **Export CSV** button regularly as a backup.

That's it — no install, no account, nothing to configure.

### Option 2 — Multi-device sync (Firestore)

If multiple front-desk computers or tablets need to see the same data live, connect the app to a free Firebase / Firestore project.

#### Set up Firebase (5 minutes)

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and create a new project.
2. In the project, click **Build → Firestore Database → Create database**. Start in **production mode** (we'll set rules in step 5).
3. Pick a region close to your office (e.g. `us-central` or `us-east4`).
4. Back on the project overview page, click the **`</>`** (web app) icon to register a web app. Give it any nickname; you do **not** need Firebase Hosting.
5. Copy the `firebaseConfig` object Firebase shows you. It looks like:
   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "your-project.firebaseapp.com",
     projectId: "your-project",
     storageBucket: "your-project.appspot.com",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abcdef"
   };
   ```
6. Open `index.html` in any text editor and find the `APP_CONFIG` block near the top of the `<script type="module">` section (it's marked with a big comment header). Paste your values:
   ```js
   const APP_CONFIG = {
     passcode: '1997',
     firebase: {
       apiKey: 'AIzaSy...',
       authDomain: 'your-project.firebaseapp.com',
       projectId: 'your-project',
       storageBucket: 'your-project.appspot.com',
       messagingSenderId: '1234567890',
       appId: '1:1234567890:web:abcdef',
     },
     collectionName: 'packages',
   };
   ```
7. **Set Firestore security rules.** This is the critical step — the Firebase API key is *not* a secret (more on that below), so security comes from rules. In the Firebase console, go to **Firestore Database → Rules** and replace them with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /packages/{doc} {
         // Only allow access from your office's IP range, OR add Firebase Auth.
         // The rules below are a starting point — TIGHTEN BEFORE PRODUCTION USE.
         allow read, write: if true;
       }
     }
   }
   ```
   ⚠️ **`allow read, write: if true` lets anyone with the URL of your app read and write your data.** This is acceptable only for testing or when the app URL is kept private. For real use, see the [Hardening the Rules](#hardening-the-rules-recommended) section below.

8. Reload the app. The sync indicator in the header should turn green and say **Synced**.

---

## Changing the Passcode

Open `index.html`, find `APP_CONFIG` near the top of the script block, and change:

```js
passcode: '1997',
```

Save the file. Anyone with the new passcode (and the old one — see security note below) can unlock.

---

## ⚠️ Security Notes

Please read this section before deploying anywhere public.

### The passcode is a soft lock, not real security

- The passcode is **stored in the page source**, which means anyone who can open the file or view source can read it.
- It exists to prevent a patient or visitor at the front desk from casually clicking through the screen — that's it.
- For real authentication, replace `Lock` with **Firebase Authentication** (email/password, Google sign-in, etc.) and gate Firestore rules on `request.auth.uid`.

### Firebase API keys are not secrets

This often surprises people: the Firebase `apiKey` value is an *identifier*, not an *authentication credential*. Google publishes this explicitly. Anyone who visits your app can see it, and that's fine — it just identifies which Firebase project to talk to.

**The actual security of your data comes from Firestore security rules.** The default rules above (`allow read, write: if true`) are wide open; you must tighten them before deploying anywhere a stranger could find the URL.

### Hardening the rules (recommended)

The simplest production-ready hardening: add Firebase Auth and require signed-in users.

1. In Firebase Console → **Authentication → Sign-in method**, enable Email/Password (or Google).
2. Create an account for each front-desk staff member.
3. Update the security rules to:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /packages/{doc} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```
4. Add a Firebase Auth sign-in step to the app (this is more code than fits in a README — happy to add this as a follow-up).

For now, the simpler protection is: **don't share the app URL publicly**, and use HTTPS hosting (GitHub Pages, Netlify, and Vercel all give you HTTPS for free).

### Scope: no patient information

This app is for **delivery metadata only** — package type, courier, who ordered it, who received it, who put it away, and operational notes. No patient-identifying information is entered or stored. Keep it that way and HIPAA does not apply to the data in this system.

If your workflow ever changes and patient-identifying details start landing in the notes field (e.g. lab specimen labels with patient names), that data becomes PHI and the rules change — at that point you'd need either local-only mode or a paid Google Cloud tier with a Business Associate Agreement. Worth a one-line reminder during staff training: **package info, not patient info**.

---

## Hosting on GitHub Pages

1. Create a new GitHub repository and push these files.
2. In repo **Settings → Pages**, set source to **Deploy from a branch**, branch `main` (or `master`), folder `/ (root)`.
3. Wait a minute, then visit `https://<your-username>.github.io/<repo-name>/`.
4. **Important:** if you've added Firebase credentials, anyone who finds this URL can read/write your data unless you've also locked down Firestore rules (see above). Consider keeping the repo **private** and using a different hosting option, or stick with Option 1 (local-only).

---

## Keyboard Shortcuts

- **`Ctrl/Cmd + K`** — focus the search box
- **`Esc`** — close any open modal
- **`Backspace`** — clear the last passcode digit
- **Number keys** — also work on the lock screen

---

## File Structure

```
medical-package-tracker/
├── index.html      ← the entire app (HTML + CSS + JS in one file)
├── README.md       ← this file
├── .gitignore
└── LICENSE
```

No build step. No `node_modules`. No package.json. Just open `index.html`.

---

## Tech Stack

- **HTML / CSS / Vanilla JavaScript** (ES Modules)
- **Firebase Web SDK v10** (loaded on demand via ESM CDN, only if configured)
- **Google Fonts** — Bricolage Grotesque + Geist
- No frameworks, no bundler, no transpiler

---

## License

MIT — see `LICENSE`. Use it, modify it, deploy it. No warranty.

---

## Troubleshooting

**The app shows "Local" in the header even though I added Firebase credentials.**
- Check that `apiKey` and `projectId` are both filled in (the app needs both to attempt a connection).
- Open the browser's developer console (F12) and look for Firebase errors.
- Make sure Firestore Database is enabled in your Firebase Console (it's a separate enable step from creating the project).

**I forgot the passcode.**
- Open `index.html` in any text editor and look for `passcode:` in `APP_CONFIG`.

**My data disappeared on a different computer.**
- Local-only mode stores data per browser per device. To share across devices, set up Firestore (Option 2).

**Sync indicator says "Offline".**
- Firestore connection failed — your data is safe locally and will sync when reconnected. Check internet and Firestore rules.
