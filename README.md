# FeTNA Shuttle Tracker — setup

Two pieces:

1. **`fetna-shuttle-tracker.html`** — the app (riders / driver / ops). Host as a static file (GitHub Pages, Netlify, etc.).
2. **`server.js` + `package.json`** — a tiny **relay** that passes driver locations to all viewers. **No database** — positions live in memory only. This is what makes two phones see each other.

You only set this up **once**.

---

## Why a relay is needed
Two phones on different networks can't talk directly. The relay is the middleman. GitHub Pages can't run it (it only serves files), so the relay runs on a free host that runs Node. The HTML stays wherever you host it and just connects to the relay over `wss://`.

---

## Step 1 — Deploy the relay (free)

### Option A: Render (easiest, ~5 min)
1. Put `server.js` + `package.json` in a GitHub repo (can be the same repo, in a `/server` folder).
2. Go to https://render.com → **New → Web Service** → connect the repo.
3. Settings: **Build command** `npm install`, **Start command** `npm start`. Instance type **Free**.
4. Deploy. You'll get a URL like `https://fetna-relay.onrender.com`. **Copy it.**

> Note: Render's free tier "sleeps" after ~15 min idle and takes ~30s to wake. For the event, hit the URL once before shuttles start, or upgrade to the cheapest always-on tier (~$7/mo) so it never naps.

### Option B: Railway / Fly.io / Glitch
Same idea — deploy a Node app, start command `npm start`, copy the https URL. Glitch is the most click-and-go (import from GitHub, it just runs).

### Run locally to test first
```
npm install
npm start
```
Open `http://localhost:3000/healthz` → should say `ok`. For phone testing you still need the public https URL (or a tunnel like `ngrok http 3000`).

---

## Step 2 — Point the app at your relay
Open `fetna-shuttle-tracker.html`, near the top find:
```js
const SERVER_URL = "PASTE_YOUR_RELAY_URL";
```
Replace with your URL, e.g.:
```js
const SERVER_URL = "https://fetna-relay.onrender.com";
```
Re-upload the HTML to GitHub Pages and **hard-refresh** both phones.

---

## Step 3 — Test it
1. Open the app on both phones. The status pill (top-right) should turn **green / "Live."**
   - If not, tap it → **Test connection**. It tells you exactly what's wrong.
2. On your phone: **Driver → code `fetna26` → pick Bus 1 → Start sharing** → allow location.
3. Drive away from the house. On the other phone, **Bus 1** should move in real time.

---

## Gotchas
- **`SERVER_URL` must be `https://`** (the relay must be https, because the page is https). Mixed http/https is blocked by browsers — the app warns you about this.
- **Free host asleep** → first connection times out; open the relay URL in a browser to wake it, then retry. The diagnostics screen flags this.
- **GPS needs https** on the driver phone — GitHub Pages is https, so you're fine. A `file://` path won't read GPS.

## After the event
The relay's CORS is wide open and the driver code is simple — fine for 3 days. For anything longer, restrict CORS to your site's origin and rotate the driver code.
