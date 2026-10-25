# Apex — Vision Board & Habit System

**Apex** is a sleek, single-file Progressive Web App (PWA) that aligns daily habit execution with long-term life goals. It targets desktop browsers and iPhone Home Screen installation, and includes **rooms**: shared spaces for up to **10 people** where everyone checks in on their own habits, sees how the others are doing, and sends cheers. One phone can be in as many rooms as you like.

| Item | Value |
|---|---|
| GitHub repo | https://github.com/sudheery007/vision-board-habit-tracker |
| Live app (GitHub Pages) | https://sudheery007.github.io/vision-board-habit-tracker/ |
| Firebase project | `visionboard-habittracker` |
| Storage key (localStorage) | `apex_vision_habit_os_v1` |

> For step-by-step Firebase / Pages / iPhone setup, the upgrade order and troubleshooting, see [SETUP.md](SETUP.md). This document describes the app itself.

---

## ✨ Key Features

### 1. Today (check-in)
* **Compact hero:** a progress ring, "2 of 4 done" and your longest streak.
* **1-tap check-in:** tap a habit row to tick it. Shared habits show how many people have finished ("3 of 6").
* **Team strip:** a row of progress rings for everyone else in the room (up to 9), tap one to cheer them.

### 2. Habits (week view)
* **One week header** (last 7 days) shared by every card, each card holding seven 44 pt day buttons. Tap any day to log or catch up.
* **Weekly goal ring** per habit (e.g. 6/7), streak and "Shared" tag. Tap a card's title to edit or delete it.
* **Just me or Everyone:** a habit is either yours or shared with the whole room; in a shared habit each person checks in separately and keeps their own streak.

### 3. Vision
* **4 Core Life Pillars:** *Technical & AI*, *Career & Leadership*, *Family & Wealth*, *Health & Vitality* (pillar filter button).
* **Mine / Shared / All** filter. Compact cards (cover, pillar, owner, progress); tap one for milestones, the "why", edit and delete.
* **Custom covers:** Unsplash image URLs or built-in gradient presets.

### 4. Team
* **Seat meter** (10 pills in each member's colour) and one row per person: progress ring, "2 of 4 today", streak.
* **Cheers:** a one-tap 👏 on each row, or tap a person for four quick cheers or a written note. A red dot on the Team tab shows unread cheers; recent cheers are listed below the team.
* **Invite** (QR code, Copy link, Share…) and an "Invite someone · N seats open" tile; a full room says "Room is full (10/10)".
* **Owner tools:** the room owner can remove a member to free a seat (their past check-ins stay in the room).

### 5. Rooms, seats and identity
* **Several rooms per phone.** Tap the room pill in the header for **Your rooms**: seats taken ("6 of 10"), who you are in each room, new-cheer counts and a "Full" mark. Join or create from there.
* **Identity is per room, set once.** You pick a name and one of 10 colours when you join; nobody switches identity on a phone, and you check in only as yourself.
* **Join flow:** enter a code or open an invite link → preview ("Book Club · 7 of 10") → *I'm new here* (name + free colour) or *I'm already a member* (pick your name; your seat comes back, no extra seat used).
* **Leaving a room** (Me → Leave) frees your seat; if the owner leaves, the longest-standing member becomes owner.
* **Room name** is shared by everyone; only the owner can change it (Me → profile).
* **Separate data per room:** each room keeps its own habits, visions and cheers. Switching flushes unsent edits, keeps a local copy (`apex_room_cache_v1_<CODE>`) and subscribes to the new room.
* **Invite links** `?room=CODE` open the join screen with the code filled in (old `&profile=p1|p2` links preselect that seat).

### 6. Me
* **Profile** (name, colour, room name), **My rooms**, **Home Screen app** (install steps + generated icon), **Sync** status, **Appearance** (Dark / Light).
* **Insights:** *Life pillar balance* and *90-day consistency* heatmap.
* **Your data:** back up to JSON, restore from JSON, export habits to CSV, add sample data, leave room.

### 7. Live Cloud Sync (Firebase)
* **One Firestore document per room** at `rooms/{roomCode}` (see below), synced in real time via `onSnapshot`.
* **Anonymous Auth:** devices sign in anonymously; the room code acts as the shared secret. Rules require an authenticated user, a room code of **12+ characters** and **at most 10 members**.
* **Debounced push:** local edits are pushed ~0.4 s after a change inside a Firestore transaction (read → merge → write).
* **Conflict-safe merge:** records merge by `id`; check-ins merge **per person and per habit** (timestamped `historyAt`), so everyone can check in simultaneously without overwriting each other.
* **Deletion tombstones:** a `deleted` map propagates removed visions, habits and cheers so they don't reappear.
* **Offline resilience:** edits stay in local storage and are retried on reconnect (`online`), when the app returns to the foreground (`visibilitychange`), or after 15 s following a failed push.
* **Per-device settings (not synced):** theme, tab, filters, and who you are in each room.
* **Status** (Me → Sync, and the dot on the room pill):

| Status | Meaning |
|---|---|
| **Live Cloud Sync** (green) | Connected and syncing |
| **Local Mode** | No Firebase config, auth failed, unauthorized domain, or opened via `file://` |
| **Local Fallback** | Firestore listener error (rules not published, room code too short, or an old cached app version) |
| **Offline – will retry** | Push failed; edits are queued |
| **Room is full** / **Room data too large** | The write was refused (10 seats / Firestore document size) |

### 8. iOS Safari PWA & Native Feel
* **Dynamic 512×512 Touch Icon:** rendered on an HTML5 `<canvas>` at startup and injected into `<link rel="apple-touch-icon">` and an inline Web App Manifest (built from a Blob URL).
* **Safe areas:** `viewport-fit=cover` with the insets read through two CSS variables (`--sat`, `--sab`). The header's top padding is applied **inline** so Tailwind utilities can never override it (an earlier version put the header under the Dynamic Island).
* **Touch-first:** every control is at least 44 × 44 pt and no text is under 12 px; pop-ups are bottom sheets (drag the grabber to dismiss) that clear the notch and home bar; inputs are 16 px so iOS doesn't zoom.
* **Dark & Light mode:** theme tokens in CSS variables; dark by default; applied before first paint.
* **Toasts** appear below the Dynamic Island.

### 9. Local Persistence & Portable Backups
* **Hybrid storage:** instant saves to `localStorage` plus cloud sync when configured.
* **Restore** adds a backup's habits and visions to the current room and puts back **your** check-ins; it never deletes anything and never touches other people's check-ins.

---

## 🧭 App Structure

Five tabs (bottom bar on phones, sidebar on desktop):

| Tab | Purpose |
|---|---|
| **Today** | Hero ring, today's habits, the team strip |
| **Habits** | Week view, weekly goals, add / edit / delete |
| **Vision** | Goal cards with milestones |
| **Team** | Seats, people, cheers, invites |
| **Me** | Profile, rooms, appearance, insights, backups, leave room |

The header has two controls: the **room pill** and **Add (+)**. Sheets: Your rooms, New room, Create (habit / vision), habit editor, vision editor, vision detail, pillar filter, member, profile, invite, Home Screen app, pillar balance, consistency. Full screen: the join flow (code → who are you → room full).

## 🪑 Data model

```
rooms/{CODE}                       (document)
  roomCode, name, ownerId
  members: { <memberId>: { name, color, uid, joinedAt } }     <- the seat list; rules cap it at 10 entries
  payload: JSON string { cheers, visions, habits, deleted, updatedAt }
  updatedBy, updatedAt
```
* **Member ids** are random (`m…`). Rooms made by the two-person app keep `p1` and `p2` as the first two member ids, so no habit, check-in or cheer was rewritten when seats arrived.
* **Habits** `owner` is `'shared'` or a member id; `partnerHistory[memberId][YYYY-MM-DD]` holds each person's check-ins and `historyAt[memberId]` their last-edit time (used by the merge).
* **Visions** `owner` is `'shared'` or a member id. **Cheers** go `from` a member `to` a member (or `'all'`).
* **Seat changes** (join, claim, remove, leave) are Firestore transactions; your own name/colour and the room name ride along on a normal push.
* **Local state** (`apex_vision_habit_os_v1`) keeps the device's room list `rooms: [{code, name, me}]`, the active room's data and per-device preferences. Opening v1 data migrates it automatically (`partners` → `members`, `activeIdentity` → `me`).

## 📁 Repository Layout

```
Habit Tracker/
├── index.html               # The entire app (HTML, CSS, JS, Firebase config)
├── SETUP.md                 # Firebase, GitHub Pages, upgrade and iPhone setup guide
├── project_documentation.md # This file
├── archive/                 # Older app versions (local only, git-ignored)
└── .gitignore
```

---

## 🚀 Deployment Overview

The app is a single static `index.html` with no build step, hosted on **GitHub Pages** and backed by a free **Firebase** project. Full instructions are in [SETUP.md](SETUP.md); in short:

1. **Firebase:** enable Anonymous sign-in, create Firestore (production mode), and publish the rules below.
2. **GitHub Pages:** *Settings → Pages → Deploy from a branch → `main` / root*, then add `sudheery007.github.io` to Firebase **Authorized domains**.
3. **Rooms:** on the live URL, tap **Create a room**, then share the invite QR / link.
4. **iPhone:** open the live URL in Safari → **Share → Add to Home Screen**.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rooms/{room} {
      function signedIn() { return request.auth != null; }
      function codeOk() { return room.size() >= 12; }
      function seatsOk() {
        return !('members' in request.resource.data)
            || request.resource.data.members.size() <= 10;
      }
      function keepsSeats() {
        return !('members' in resource.data) || ('members' in request.resource.data);
      }
      allow read: if signedIn() && codeOk();
      allow create: if signedIn() && codeOk() && seatsOk();
      allow update: if signedIn() && codeOk() && seatsOk() && keepsSeats();
    }
  }
}
```

**Updating the app:** edit `index.html`, then `git add -A && git commit -m "update" && git push`. Pages redeploys in about a minute (reload twice on iPhone if it looks stale).

**Local testing:** `python3 -m http.server 8000` in the project folder, then open `http://localhost:8000` (`localhost` is authorized in Firebase by default).

> [!NOTE]
> The room code is effectively the password to a room: anyone who has it can read it, and anyone with a free seat can join. Share invites only with people you trust.

---

## 🛠️ Tech Stack
* **Markup & Styling:** HTML5, Tailwind CSS (CDN Play, pinned to 3.4.16) plus a small set of theme-token component classes.
* **Icons & Typography:** Lucide Icons (unpkg, pinned to 0.468.0), Google Fonts (`Inter`).
* **Client-Side Graphics:** HTML5 `<canvas>` (512×512 PWA icon) and `QRious` 4.0.2 for the invite QR code.
* **State & Sync:** Browser `localStorage` + Firebase JS SDK **11.6.1** (`firebase-app`, `firebase-auth`, `firebase-firestore`) loaded as ES modules from `gstatic.com`; Anonymous Auth, Firestore transactions and real-time snapshots.
* **Rendering:** vanilla JS template strings; all user-provided text is HTML-escaped and all actions use event delegation (`data-act`), so nothing a room member types can inject markup or script.
* **Hosting:** GitHub Pages (static).

## 🧩 Known Limitations
* The Firebase web config is public in `index.html` (normal for web apps); access control relies on Anonymous Auth, the 12+ character room code and the Firestore rules above.
* Firestore security is room-code based, not per-person: anyone in a room is trusted. The seat limit is enforced by the rules, but who may rename the room or remove a member is enforced only by the app's screens.
* A room is **one Firestore document** (1 MiB limit). Check-ins older than 370 days are dropped automatically, which keeps even a busy 10-person room comfortably below the limit; if a room still grows too large the app shows "Room data too large". Splitting check-ins into one document per person would remove the limit if rooms ever outgrow this.
* Safari and the Home Screen app keep separate storage, so each looks like a new person; "I'm already a member" returns your seat.
* Merge conflicts on the same vision / habit field resolve in favor of the device with unsent edits (otherwise the cloud copy wins); check-ins merge per person by timestamp.
* Tailwind CDN and Lucide are loaded at runtime (pinned versions), so first load requires internet access.
