# Apex Duo — Vision Board & Habit System

**Apex Duo** is a sleek, single-file Progressive Web App (PWA) that aligns daily habit execution with long-term life goals. It targets desktop browsers and iPhone Home Screen installation, and includes a real-time **Partner Room** so couples or accountability partners can track individual and shared goals, keep streaks, and send live cheers.

| Item | Value |
|---|---|
| GitHub repo | https://github.com/sudheery007/vision-board-habit-tracker |
| Live app (GitHub Pages) | https://sudheery007.github.io/vision-board-habit-tracker/ |
| Firebase project | `visionboard-habittracker` |
| Storage key (localStorage) | `apex_vision_habit_os_v1` |

> For step-by-step Firebase / Pages / iPhone setup and the status checklist, see [SETUP.md](SETUP.md). This document describes the app itself.

---

## ✨ Key Features

### 1. Vision Board (Bento Architecture)
* **4 Core Life Pillars:** *Technical & AI*, *Career & Leadership*, *Family & Wealth*, *Health & Vitality*, with a pillar filter.
* **Interactive Milestones:** Each vision card has checkable key results that drive animated progress bars and the overall Vision Mastery KPI.
* **Custom Covers:** Unsplash image URLs or built-in gradient presets.
* **Ownership Tagging:** Assign a vision to Partner 1 (Sudheer), Partner 2 (Devi), or *Together / Shared*, and filter by owner.

### 2. Daily Habit Rituals & 90-Day Heatmap
* **1-Tap Check-In:** Log today's habits from the Command Center dashboard.
* **Dual Partner Check-Ins:** Shared habits let each partner check in independently, with separate streak counters.
* **7-Day Rolling Matrix:** Back-fill or toggle any day in the past week, with per-partner indicator dots.
* **90-Day Consistency Grid:** GitHub-style heatmap over 13 weeks (91 days) showing active days and total check-ins.

### 3. Partner Accountability
* **Side-by-Side Progress Rings:** Live SVG completion rings and streak badges for both partners.
* **Live Cheer Feed:** One-tap encouragements (*🔥 Crush it!*, *👏 Proud of you!*, *💪 Finish strong!*, *❤️ Love this!*) or custom notes. The feed is capped at the latest 100 cheers.
* **Invite QR / Link:** The **Partner Room** modal (rose button in the header) generates a room code, an invite link (`?room=YOUR-CODE&profile=p1|p2`), an on-screen QR code, and a downloadable PNG QR poster card. The invite carries the room code and pre-selects the person on the scanning phone.

### 4. Live Cloud Sync (Firebase)
* **Firestore Rooms:** Data is stored in one document per room at `rooms/{roomCode}`, synced in real time via `onSnapshot`.
* **Anonymous Auth:** Devices sign in anonymously; the room code acts as the shared secret. Security rules require an authenticated user and a room code of **12+ characters**.
* **Debounced Push:** Local edits are pushed ~0.4 s after a change inside a Firestore transaction (read → merge → write).
* **Conflict-Safe Merge:** Records merge by `id`; check-ins merge **per person** (timestamped `historyAt` for `p1`/`p2`), so both partners can check in simultaneously without overwriting each other.
* **Deletion Tombstones:** A `deleted` map propagates removed visions, habits and cheers to every device so they don't reappear.
* **Offline Resilience:** Edits stay in local storage and are retried on reconnect (`online`), when the app returns to the foreground (`visibilitychange`), or after 15 s following a failed push.
* **Per-Device Settings (not synced):** Theme, active tab, filters and the "who am I" identity.
* **Status Badge:**

| Badge | Meaning |
|---|---|
| **Live Cloud Sync** (green) | Connected and syncing |
| **Local Mode** | No Firebase config, auth failed, unauthorized domain, or opened via `file://` |
| **Local Fallback** | Firestore listener error (rules not published or room code too short) |
| **Offline – will retry** | Push failed; edits are queued |

### 5. iOS Safari PWA & Native Feel
* **Dynamic 512×512 Touch Icon:** Rendered on an HTML5 `<canvas>` at startup and injected into `<link rel="apple-touch-icon">` and an inline Web App Manifest (built from a Blob URL).
* **iPhone-Optimized Viewport:** `viewport-fit=cover`, `apple-mobile-web-app-capable`, safe-area padding for the Dynamic Island / Home Indicator, and 16 px inputs to prevent iOS auto-zoom.
* **Dark & Light Mode:** Slate dark mode by default (`#0f172a`) with a 1-tap toggle.
* **PWA Status Modal:** Shows standalone-install status and lets you download the generated icon.

### 6. Local Persistence & Portable Backups
* **Hybrid Storage:** Instant saves to `localStorage` plus cloud sync when configured.
* **Analytics Tab Tools:** **Export JSON Backup**, **Import JSON Backup**, **Export Habits to CSV** (30-day matrix), and **Reset Sample Data**.

---

## 🧭 App Structure

Four tabs (sidebar on desktop, bottom nav on mobile):

| Tab | Purpose |
|---|---|
| **Command Center** (`dashboard`) | Partner rings, today's habits, cheer feed, quick stats |
| **Vision Board** (`vision`) | Pillar-filtered vision cards and milestones |
| **Habits** (`habits`) | Habit list, 7-day matrix, 90-day heatmap |
| **Analytics** (`analytics`) | Pillar breakdown, backup / import / CSV export, PWA info |

Modals: Vision editor, Habit editor, Confirm / Reset, Custom Cheer, PWA status, and **Partner Room & Instant QR Sync**.

## 📁 Repository Layout

```
Habit Tracker/
├── index.html               # The entire app (HTML, CSS, JS, Firebase config)
├── SETUP.md                 # Firebase, GitHub Pages and iPhone setup guide
├── project_documentation.md # This file
├── archive/                 # Older app versions (local only, git-ignored)
└── .gitignore
```

---

## 🚀 Deployment Overview

The app is a single static `index.html` with no build step, hosted on **GitHub Pages** and backed by a free **Firebase** project. Full instructions are in [SETUP.md](SETUP.md); in short:

1. **Firebase:** enable Anonymous sign-in, create Firestore (production mode), and publish the rules below.
2. **GitHub Pages:** *Settings → Pages → Deploy from a branch → `main` / root*, then add `sudheery007.github.io` to Firebase **Authorized domains**.
3. **Partner Room:** on the live URL, tap **Partner Room → Generate → Save & Sync Room**, then share the invite QR / link with the other phone. Avoid the legacy default code `APEX-SD`.
4. **iPhone:** open the live URL in Safari → **Share → Add to Home Screen**, and confirm the correct identity (Sudheer / Devi) on each phone.

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

**Updating the app:** edit `index.html`, then `git add -A && git commit -m "update" && git push`. Pages redeploys in about a minute (reload twice on iPhone if it looks stale).

**Local testing:** `python3 -m http.server 8000` in the project folder, then open `http://localhost:8000` (`localhost` is authorized in Firebase by default).

> [!NOTE]
> The room code is effectively the password to your data: anyone who has it can read and edit the room. Choose it once before daily use; changing it later moves only that device to the new room.

---

## 🛠️ Tech Stack
* **Markup & Styling:** HTML5, Tailwind CSS (CDN Play), custom glassmorphic and iOS safe-area CSS.
* **Icons & Typography:** Lucide Icons (unpkg), Google Fonts (`Inter`).
* **Client-Side Graphics:** HTML5 `<canvas>` (512×512 PWA icon, downloadable QR invite cards) and `QRious` 4.0.2 for QR generation (with a canvas fallback renderer).
* **State & Sync:** Browser `localStorage` + Firebase JS SDK **11.6.1** (`firebase-app`, `firebase-auth`, `firebase-firestore`) loaded as ES modules from `gstatic.com`; Anonymous Auth, Firestore transactions and real-time snapshots.
* **Hosting:** GitHub Pages (static).

## 🧩 Known Limitations
* The Firebase web config is public in `index.html` (normal for web apps); access control relies on Anonymous Auth, the 12+ character room code and the Firestore rules above.
* Firestore security is room-code based, not per-user, so share invites only with trusted partners.
* Merge conflicts on the same vision / habit field resolve in favor of the device with unsent edits (otherwise the cloud copy wins); check-ins merge per person by timestamp.
* Tailwind CDN and Lucide `@latest` are loaded at runtime, so first load requires internet access.