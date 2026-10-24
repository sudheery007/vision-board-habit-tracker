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

## 3. Host it (pick one)
- **Netlify**: drag the folder (only `index.html` needed) onto https://app.netlify.com/drop
- **Cloudflare Pages / GitHub Pages**: upload `index.html`.

In the Firebase console → **Authentication → Settings → Authorized domains**, add your hosting domain
(e.g. `your-site.netlify.app`).

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
