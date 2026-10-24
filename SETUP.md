# Apex – Live Sync Setup (2 iPhones + web)

One static file (`index.html`), one free Firebase project. ~10 minutes.

## 1. Create the Firebase project
1. Go to https://console.firebase.google.com → **Add project** (disable Analytics).
2. **Build → Authentication → Get started → Sign-in method → Anonymous → Enable.**
3. **Build → Firestore Database → Create database** → *Production mode* → pick a nearby region.
4. **Firestore → Rules** → paste and **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rooms/{room} {
      allow read, write: if request.auth != null
                         && room.size() >= 8;   // blocks trivially short room codes
    }
  }
}
```

5. **Project settings (gear) → Your apps → Web (</>)** → register app → copy the `firebaseConfig`.
6. Paste the values into `FIREBASE_CONFIG` near the bottom of `index.html`.

## 2. Pick a private room code
The room code is the "password". The default `APEX-SD` is guessable.
Open the app → Partner Room → use **Generate** (or type a long code, 8+ chars), then
share the invite QR/link with the other phone. Both devices must use the same code.

## 3. Host it on GitHub Pages
The project is already a local git repo (branch `main`, `archive/` is git-ignored).

1. On https://github.com/new create a **public** repo (e.g. `apex-habit-tracker`) – no README/license, since the folder already has commits.
   (GitHub Pages is free for public repos; private repos need a paid plan.)
2. Push from the project folder:
   ```
   git remote add origin https://github.com/<your-username>/apex-habit-tracker.git
   git push -u origin main
   ```
   When prompted for a password, use a **Personal Access Token** (GitHub → Settings → Developer settings →
   Tokens (classic) → `repo` scope), or sign in via GitHub Desktop / `gh auth login`.
3. Repo → **Settings → Pages → Build and deployment → Source: Deploy from a branch → `main` / `/ (root)` → Save.**
4. After ~1 minute the app is live at `https://<your-username>.github.io/apex-habit-tracker/`.
5. Firebase console → **Authentication → Settings → Authorized domains → Add domain** →
   `<your-username>.github.io` (domain only, no path).

To update later: edit `index.html`, then `git add -A && git commit -m "update" && git push`.

> Use the live URL (not `localhost`) when you generate the invite QR/link, so the link opens on the phones.

## 4. Install on each iPhone
Open the URL in **Safari → Share → Add to Home Screen.**
Then in the app, tap Partner Room and choose your identity (Sudheer / Devi) on each phone.

## How sync behaves
- Edits push to the cloud ~0.4 s after you change something and appear live on the other devices.
- Check-ins merge per person, so you and your partner can check in at the same moment safely.
- Deleting a habit/vision propagates to all devices.
- Offline edits are kept locally and sent when you are back online / reopen the app.
- Theme, tab, filters and "who am I" stay per-device and are not synced.
- The status dot is green with **Live Cloud Sync**; amber/**Local Mode** means no config or no connection.
