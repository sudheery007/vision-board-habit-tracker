# Apex – Live Sync Setup (iPhones + web)

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
- [ ] Firebase: Firestore created + **v2.0 rules (seat limit)** published (step 1)
- [ ] GitHub Pages enabled (step 2)
- [ ] Firebase: `sudheery007.github.io` added to Authorized domains (step 2)
- [ ] Room created, people invited (step 3)
- [ ] Added to Home Screen on every iPhone (step 4)

## 1. Firebase console (https://console.firebase.google.com → `visionboard-habittracker`)
1. **Build → Authentication → Get started → Sign-in method → Anonymous → Enable.**
2. **Build → Firestore Database → Create database** → *Production mode* → nearby region.
3. **Firestore → Rules** → paste and **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rooms/{room} {
      function signedIn() {
        return request.auth != null;
      }
      function codeOk() {
        return room.size() >= 12;            // rejects short / guessable room codes
      }
      function seatsOk() {                   // a room holds at most 10 people
        return !('members' in request.resource.data)
            || request.resource.data.members.size() <= 10;
      }
      function keepsSeats() {                // once a room has a seat list, every write must keep it
        return !('members' in resource.data)
            || ('members' in request.resource.data);
      }
      allow read: if signedIn() && codeOk();
      allow create: if signedIn() && codeOk() && seatsOk();
      allow update: if signedIn() && codeOk() && seatsOk() && keepsSeats();
    }
  }
}
```

The `seatsOk` line is what enforces the 10-person limit on the server, so it holds even if someone edits the app.
`keepsSeats` stops an old cached copy of the app (from before seats existed) from wiping a room's seat list.

Optional check (Rules Playground, simulate **create**, authenticated, path `/rooms/APEX-TEST-ROOM-0001`):
data `{"members":{"a":{},"b":{},"c":{},"d":{},"e":{},"f":{},"g":{},"h":{},"i":{},"j":{}}}` is **allowed** (10), and adding a key `"k":{}` is **denied** (11).

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

## 3. Create your room and invite people
1. Open the live URL → **Create a room** (first launch) or **room pill → New room**.
2. Enter a room name, your name and a colour. You become the owner and take seat 1 of 10.
3. The **Invite** sheet opens: send the QR code or link (**Copy link** / **Share…**). Anyone who opens it lands on the join screen
   with the code filled in, picks a name and colour, and takes the next free seat.

You never type a room code by hand. (Joining by typed code is still there: **room pill → Join a room**.)
A room holds up to **10 people**; a full room says so before anyone types anything.

## 4. Install on each iPhone
Open the live URL (or an invite link) in **Safari → Share → Add to Home Screen.**
Then open **Me → Home Screen app** if you want the install steps again.

Safari and the Home Screen app keep **separate storage**, so each one looks like a new person to the room.
If you joined in Safari and then opened the Home Screen app (or got a new phone), choose **"I'm already a member"** and pick your name:
your seat comes back and no extra seat is used.

## Upgrading from the two-person version (v1 → v2.0)
Do these in order so nobody is locked out:
1. `git push` the new `index.html` and wait for Pages to redeploy. The new app works with your **old** rules.
2. On **both phones**, open the app and reload twice (or remove and re-add the Home Screen icon).
   Your existing room opens by itself: Sudheer's phone as Sudheer, Devi's as Devi. All habits, visions, check-ins and cheers are kept.
   Sudheer and Devi become seats 1 and 2 of the room (Sudheer is the owner).
3. **Then** publish the new rules from step 1. A phone still running the old cached version is refused once they are live
   (it shows *Local Fallback*) until it reloads.
4. Check **Team** shows both people and **Me → Sync** says *Live Cloud Sync*.

If your old room still uses the short default code `APEX-SD`, it never synced (codes need 12+ characters): create a new room instead.

## How sync behaves
- Edits push to the cloud ~0.4 s after you change something and appear live on everyone's phone.
- You check in as yourself. Check-ins merge per person, so everyone can check in at the same moment safely.
- Joining, claiming a seat, removing someone and leaving run as single database transactions, so two people can never both take the last seat.
- Deleting a habit/vision propagates to everyone.
- Offline edits are kept locally and sent when you are back online / reopen the app.
- Theme, tab and filters stay per-device and are not synced. Each room remembers who you are in it, on each device.
- Check-ins older than 370 days are dropped automatically to keep a busy room well under Firestore's 1 MiB document limit.
- **Me → Sync** shows the status: green *Live Cloud Sync*; amber means no config or no connection.

## Troubleshooting
| Symptom | Likely cause |
|---|---|
| Sync says **Local Mode** | Domain not in Authorized domains, Anonymous auth not enabled, the page was opened as a `file://`, or the Firebase SDK could not load (no internet). Check the browser console. |
| Sync says **Local Fallback** | Rules not published, room code shorter than 12 chars, or this phone runs an old cached version that the new rules refuse. Reload twice. |
| **Offline – will retry** | No connection; edits are kept and retried automatically. |
| **Room is full** | All 10 seats are taken. The owner can free one (Team → tap a person → Remove). If you already belong, choose "I'm already a member". |
| **Room data too large** | The room document reached Firestore's size limit. Delete old habits you no longer use. |
| Phones show different data | They are in different rooms: compare names in **Me → My rooms** (or the room pill). |
| App asks "Who are you?" | It could not tell which seat is yours (new phone, or Safari vs Home Screen). Pick your name. |
| "You are no longer in this room" | Someone freed your seat. Choose who you are to come back, or leave the room on this phone. |
| Old version after an update | Pages cache; reload twice or re-add to Home Screen. |

Test locally (optional): `python3 -m http.server 8000` in this folder, open `http://localhost:8000`
(`localhost` is authorized in Firebase by default).
