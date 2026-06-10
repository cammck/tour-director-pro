# Tour Director Pro

A tour management web app for managing tour groups, dietary requirements, and provider email templates. Built as a single-page app, hosted on Firebase Hosting, with data stored in Firestore.

---

## Project Structure

```
tour-director-pro/
├── public/
│   ├── index.html        ← The full single-page app
│   └── favicon.svg       ← App icon
├── .github/
│   └── workflows/
│       └── deploy.yml    ← GitHub Actions CI/CD pipeline
├── firebase.json         ← Firebase Hosting + Firestore config
├── firestore.rules       ← Firestore security rules
├── firestore.indexes.json
├── .firebaserc           ← Firebase project alias
└── .gitignore
```

---

## First-Time Setup

### 1. Firebase Project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Create a new project (e.g. `tour-director-pro`)
3. Add a **Web app** — note the `firebaseConfig` values shown
4. Enable **Authentication → Sign-in providers**:
   - Google
   - Email/Password
5. Create a **Firestore Database** — choose your nearest region, start in **production mode**
6. The Firestore rules are deployed automatically via CI/CD (see below)

### 2. Update `.firebaserc`

Replace `YOUR_FIREBASE_PROJECT_ID` with your actual Firebase project ID:

```json
{
  "projects": {
    "default": "your-actual-project-id"
  }
}
```

### 3. GitHub Secrets

In your GitHub repo go to **Settings → Secrets and variables → Actions** and add these secrets:

| Secret name | Where to find it |
|---|---|
| `FIREBASE_API_KEY` | Firebase console → Project settings → Your apps |
| `FIREBASE_AUTH_DOMAIN` | e.g. `your-project.firebaseapp.com` |
| `FIREBASE_PROJECT_ID` | e.g. `your-project-id` |
| `FIREBASE_STORAGE_BUCKET` | e.g. `your-project.appspot.com` |
| `FIREBASE_MESSAGING_SENDER_ID` | Firebase console → Project settings |
| `FIREBASE_APP_ID` | Firebase console → Project settings |
| `FIREBASE_SERVICE_ACCOUNT` | See below |

#### Generating the Service Account secret

The `FIREBASE_SERVICE_ACCOUNT` is a JSON key that lets GitHub Actions deploy on your behalf:

```bash
# Install Firebase CLI if you haven't already
npm install -g firebase-tools
firebase login

# Generate the service account JSON and copy it
firebase init hosting  # choose your project when prompted
```

Or manually:
1. Firebase console → **Project settings → Service accounts**
2. Click **Generate new private key**
3. Copy the entire contents of the downloaded JSON file
4. Paste it as the value of the `FIREBASE_SERVICE_ACCOUNT` secret in GitHub

### 4. Push to Deploy

Once secrets are configured, every push to `main` automatically:
1. Injects your Firebase config values into `index.html` (replacing placeholders)
2. Deploys `public/` to Firebase Hosting
3. Deploys `firestore.rules` to Firestore

Pull requests get a **preview channel** deployment with a unique URL for testing.

---

## Local Development

To run locally with your real Firebase config, either:

**Option A** — Edit `public/index.html` directly and replace the `firebaseConfig` placeholder values. Do **not** commit this file with real keys.

**Option B** — Use the Firebase emulator:

```bash
npm install -g firebase-tools
firebase emulators:start --only hosting,firestore,auth
```

---

## Firestore Data Structure

```
users/
  {userId}/
    config/
      globalConfig      ← Tour Director details, tour defaults
      navState          ← Last active view/tour (per user)
    providers/
      0, 1, 2, ...      ← Email templates (numbered documents)
    tours/
      {tourId}          ← One document per tour
```

---

## Security

- All Firestore reads and writes are scoped to the authenticated user's own `users/{uid}/` path
- Firebase config values (API keys etc.) are stored as GitHub Secrets and injected at deploy time — they are never committed to the repository
- The `firebaseConfig` in `index.html` in the repo always contains placeholder values
