<div align="center">

# 🔭 Orrery

**A deep-space observatory for focused study.**

Pick a sky. Run a session on the orrery timer. Clear missions, earn rank, and pin observation notes to the stars.

[![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org)
[![Express](https://img.shields.io/badge/Express-5-000000?style=flat-square&logo=express&logoColor=white)](https://expressjs.com)
[![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?style=flat-square&logo=mongodb&logoColor=white)](https://mongodb.com)
[![License](https://img.shields.io/badge/License-MIT-e8a33d?style=flat-square)](LICENSE)

</div>

---

## 🌌 Overview

Orrery is a full-stack study environment built around a single idea: focus is easier when the room around you is worth sitting in. Instead of a bare timer, you get a view — five procedurally rendered deep-space scenes, each with its own generated ambience — and a session clock shaped like the instrument the project is named after.

Everything you create is yours and persists. Accounts are permanent records in MongoDB, protected by hashed passwords and server-side sessions, and are removed only when *you* choose to remove them.

> **📸 Screenshots** — add yours here before publishing:
> `docs/screenshot-airlock.png` (sign-in) and `docs/screenshot-deck.png` (observatory)

---

## ✨ Features

| | Feature | Detail |
|:--|:--|:--|
| 🌠 | **Five ambient skies** | Nebula, Solar Flare, Deep Void, Meteor Shower and Aurora Orbit — all drawn on `<canvas>` and scored with the Web Audio API, so the app ships **zero** video or audio files |
| ⏱️ | **The orrery timer** | A configurable study/break cycle rendered as an orbit. The body sweeps one full revolution per session; the ring shifts from brass to plasma when the break begins |
| 🎯 | **XP missions** | Routine (10 XP), Standard (25 XP) and Deep (60 XP) work, with instant progression feedback |
| 🎖️ | **Nine ranks** | Cadet → Archivist of Stars, on a curve of `50 × L × (L − 1)` total XP per level |
| 📊 | **Log book** | Total hours, sessions completed, missions cleared, a 14-night bar chart and a live night streak |
| 📝 | **Observation log** | Draggable notes pinned to the sky itself — position saved per user, so the board is identical on every device |
| 🔐 | **Real authentication** | bcrypt hashing, MongoDB-backed sessions, rate-limited sign-in, and per-user data isolation on every query |
| 🗑️ | **Account deletion** | One explicit, password-confirmed path that cascades through every record you own |

---

## 🛠️ Tech Stack

**Backend** — Node.js · Express 5 · MongoDB · Mongoose · bcrypt · express-session · connect-mongo · helmet · express-rate-limit · dotenv

**Frontend** — Vanilla HTML, CSS and ES modules · Canvas 2D · Web Audio API · zero build step, zero framework

---

## 🚀 Quick Start

**Prerequisites:** Node.js 18+ and a MongoDB database (local `mongod` or a free [Atlas](https://mongodb.com/cloud/atlas) cluster).

```bash
# 1 — clone and install
git clone https://github.com/<your-username>/orrery.git
cd orrery
npm install

# 2 — configure
cp .env.example .env
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
# paste that value into SESSION_SECRET, then set MONGO_URI

# 3 — launch
npm run dev
```

Open **http://localhost:3000** and register an observer. Mongoose creates the collections on first sign-up — there is no migration step.

<details>
<summary><b>Windows PowerShell one-liner for step 2</b></summary>

```powershell
$secret = node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
Set-Content -Path .env -Encoding ascii -Value @(
  "PORT=3000",
  "NODE_ENV=development",
  "MONGO_URI=mongodb://127.0.0.1:27017/orrery",
  "SESSION_SECRET=$secret",
  "SESSION_DAYS=14"
)
```

</details>

---

## ⚙️ Environment

| Variable | Required | Default | Purpose |
|:--|:--:|:--|:--|
| `MONGO_URI` | ✅ | — | Connection string. Local or Atlas |
| `SESSION_SECRET` | ✅ | — | Signs the session cookie. Use 48 random bytes |
| `PORT` | — | `3000` | HTTP port |
| `NODE_ENV` | — | `development` | Set to `production` to force secure cookies |
| `SESSION_DAYS` | — | `14` | How long a signed-in session stays valid |

The server **refuses to boot** if a required variable is missing, rather than starting half-configured.

---

## 🔌 API

All responses are JSON. Everything except signup, login and logout requires an active session.

### Authentication

| Method | Endpoint | Body | Description |
|:--|:--|:--|:--|
| `POST` | `/api/auth/signup` | `username, email, password` | Register and sign in |
| `POST` | `/api/auth/login` | `identifier, password` | Sign in with call sign **or** email |
| `POST` | `/api/auth/logout` | — | End the session, account untouched |
| `GET` | `/api/auth/me` | — | Current observer, with level and rank |
| `PATCH` | `/api/auth/preferences` | `scene, studyMinutes, breakMinutes, soundOn` | Save preferences |
| `DELETE` | `/api/auth/account` | `password` | ⚠️ Permanently delete the account and all its data |

### Resources

| Method | Endpoint | Description |
|:--|:--|:--|
| `GET` `POST` | `/api/missions` | List / create a mission |
| `PATCH` `DELETE` | `/api/missions/:id` | Update or complete (moves XP) / delete |
| `GET` `POST` | `/api/logs` | List / create an observation note |
| `PATCH` `DELETE` | `/api/logs/:id` | Edit body, tint or position / delete |
| `POST` | `/api/sessions` | Record a finished focus block — 2 XP per minute |
| `GET` | `/api/sessions/stats` | Totals, 14-day series and streak |
| `GET` | `/api/health` | Uptime probe |

---

## 🔒 Security Notes

| Concern | Approach |
|:--|:--|
| **Passwords** | bcrypt at 12 rounds. Plaintext is never stored or logged, and `passwordHash` is `select: false` so it stays out of every query that does not explicitly ask for it |
| **Uniqueness** | Unique indexes on username and email with an English collation, so `Vega` and `vega` cannot both be registered |
| **Sessions** | Stored in MongoDB via `connect-mongo`, not in memory — a restart does not sign anyone out. Cookies are `httpOnly`, `sameSite=lax`, and `secure` in production |
| **Data isolation** | Every mission, note and session carries an indexed `owner` reference, and every query filters on the session's user id |
| **Enumeration** | A wrong call sign returns the same message as a wrong password, so the form cannot be used to discover which accounts exist |
| **Brute force** | Sign-in is limited to 20 attempts per 15 minutes per IP; the wider API to 240 requests per minute |
| **Headers** | `helmet` with a content security policy restricting scripts to same-origin |

---

## 📁 Project Structure

```
orrery/
├── server.js                  boot, security middleware, static hosting
├── src/
│   ├── config/
│   │   ├── db.js              mongoose connection
│   │   └── session.js         express-session backed by MongoDB
│   ├── models/                User · Mission · LogEntry · StudySession
│   ├── middleware/            requireAuth · rateLimits · errorHandler
│   ├── controllers/           all request handling
│   ├── routes/                route tables only
│   └── utils/xp.js            the progression curve
└── public/
    ├── index.html             sign in / sign up
    ├── observatory.html       the application
    ├── css/instrument.css     design tokens and components
    └── js/
        ├── sky.js             canvas scene renderer
        ├── ambience.js        Web Audio ambience
        ├── api.js             fetch wrapper
        ├── auth.js            airlock logic
        └── app.js             observatory logic
```

---

## ⌨️ Keyboard

| Key | Action |
|:--|:--|
| `Space` | Start or pause the session |
| `Esc` | Close the open panel |

---

## 🧭 Roadmap

- [ ] Mission Control — an admin view with role-based access
- [ ] Seed script for demo data and screenshots
- [ ] Session export to CSV
- [ ] Shared observatories — study alongside a friend in real time
- [ ] Optional light theme for daytime observing

---

## 🧯 Troubleshooting

<details>
<summary><b><code>[db] could not connect: ECONNREFUSED 127.0.0.1:27017</code></b></summary>

MongoDB is not running. Start it (`net start MongoDB` on Windows, `brew services start mongodb-community` on macOS, `sudo systemctl start mongod` on Linux), or switch `MONGO_URI` to an Atlas connection string.

</details>

<details>
<summary><b><code>gyp ERR!</code> or <code>MSBuild.exe failed</code> during <code>npm install</code></b></summary>

`bcrypt` compiles native code and has no prebuilt binary for newer Node releases on Windows. Swap in the pure-JavaScript build instead of installing Visual Studio Build Tools:

```bash
npm uninstall bcrypt
npm install bcryptjs
```

Then change line 2 of `src/models/User.js` to `require('bcryptjs')`. Same API, same hash format.

</details>

<details>
<summary><b><code>[config] missing in .env</code></b></summary>

The `.env` file has not been created yet, or is missing `MONGO_URI` / `SESSION_SECRET`. See [Quick Start](#-quick-start) step 2.

</details>

---

## 📄 License

MIT — see [LICENSE](LICENSE).

<div align="center">

**Built with Node.js, Express and a lot of night sky.**

</div>
