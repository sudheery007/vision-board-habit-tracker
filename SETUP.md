# Apex – Live Sync Setup (2 iPhones + web)

One static file (`index.html`), one free Firebase project, hosted on GitHub Pages.

| Item | Value |
|---|---|
| GitHub repo | https://github.com/sudheery007/vision-board-habit-tracker |
| Live app (after Pages is on) | https://sudheery007.github.io/vision-board-habit-tracker/ |
| Firebase project | `visionboard-habittracker` |

## Status checklist
- [x] App consolidated into `index.html` (old versions in local `archive/`, git-ignored)
- [x] Firebase config pasted into `FIREBASE_CONFIG` in `index.html`
- [x] Code pushed to GitHub (`main`)
- [ ] Firebase: Anonymous sign-in enabled
- [ ] Firebase: Firestore created + rules published (step 1)
- [ ] GitHub Pages enabled (step 2)
- [ ] Firebase: `sudheery007.github.io` added to Authorized domains (step 2)
- [ ] Private room code generated and shared (step 3)
- [ ] Added to Home Screen on both iPhones (step 4)

## 1. Firebase console (https://console.firebase.google.com → `visionboard-habittracker`)
1. **Build → Authentication → Get started → Sign-in method → Anonymous → Enable.**
2. **Build → Firestore Database → Create database** → *Production mode* → nearby region.
3. **Firestore → Rules** → paste and **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rooms/{room} {
      allow read, write: if request.auth != null
                         && room.size() >= 12;   // rejects short/guessable room codes
    }
  }
}
```

The Firebase web config is already in `index.html`. Do **not** paste the console's `import { initializeApp } ...`
snippet; the app loads the SDK from Google's CDN itself.

## 2. Turn on GitHub Pages
1. Repo → **Settings → Pages → Build and deployment → Source: Deploy from a branch → `main` / `/ (root)` → Save.**
2. Wait ~1 minute, then open https://sudheery007.github.io/vision-board-habit-tracker/
3. Firebase console → **Authentication → Settings → Authorized domains → Add domain** → `sudheery007.github.io`
   (domain only, no path). Without this, sign-in fails and the app stays in Local Mode.

Updating the app later: edit `index.html`, then
```
git add -A && git commit -m "update" && git push
```
Pages redeploys in about a minute. (Reload twice on iPhone, or remove and re-add the Home Screen icon, if it looks stale.)

## 3. Pick a private room code
The room code is the "password" to your data; anyone who has it can read and edit the room.
1. On the **live URL** (not `localhost`), tap **Partner Room** (rose button in the header).
2. Tap **Generate** next to Room Code (gives something like `APEX-K7QM-X3RT-9HWD`), then **Save & Sync Room**.
3. Send the invite **QR / link** to the other phone. It carries the room code and pre-selects the person.
   Don't type codes by hand, and don't use the old default `APEX-SD`.

Pick the code once, before daily use. Changing it later moves only that device to the new room.

## 4. Install on each iPhone
Open the live URL in **Safari → Share → Add to Home Screen.**
Then open Partner Room and make sure the right identity (Sudheer / Devi) is selected on each phone.
Open the app from the invite link at least once so the room code is saved on that phone.

## How sync behaves
- Edits push to the cloud ~0.4 s after you change something and appear live on the other devices.
- Check-ins merge per person, so you and your partner can check in at the same moment safely.
- Deleting a habit/vision propagates to all devices.
- Offline edits are kept locally and sent when you are back online / reopen the app.
- Theme, tab, filters and "who am I" stay per-device and are not synced.
- The status dot is green with **Live Cloud Sync**; amber/**Local Mode** means no config or no connection.

## Troubleshooting
| Symptom | Likely cause |
|---|---|
| Badge says **Local Mode** | Domain not in Authorized domains, Anonymous auth not enabled, or the page was opened as a `file://`. Check the browser console. |
| Badge says **Local Fallback** | Firestore rules not published, or room code shorter than 12 chars. |
| **Offline – will retry** | No connection; edits are kept and retried automatically. |
| Phones show different data | They are in different rooms: compare the code in Partner Room. |
| Old version after an update | Pages cache; reload twice or re-add to Home Screen. |

Test locally (optional): `python3 -m http.server 8000` in this folder, open `http://localhost:8000`
(`localhost` is authorized in Firebase by default).
