# The Family Cup 🏆

A self-contained, single-file lawn-games tournament tracker. Everything lives in
`index.html` — no build step, no dependencies to install. Double-click it and it
runs.

There are two ways to use it:

- **Solo / offline (default).** Open `index.html` with no URL parameters. It works
  exactly as it always has: fully editable on one device, saved to `localStorage`.
  No internet or Firebase required.
- **Live (optional).** Add a Firebase Realtime Database and the app can host a
  shared "cup" that family members watch live from their own phones while you run
  it from yours.

The live layer is **purely additive**. If you never touch the Firebase config, the
app stays 100% local.

---

## Live sync: how it works

A **cup** is one tournament instance identified by a slug, e.g.
`fourth-of-july-2026`. The app reads it from the URL:

- **View link** — `.../index.html?cup=<id>` — read-only. Anyone can watch rounds,
  standings, the bracket, gifs, and the champion reveal, but cannot change results.
  They see a small **"Live · viewing"** badge.
- **Host link** — `.../index.html?cup=<id>&host=<code>` — full control. The host
  taps winners, sets advancers, resets, and edits gifs. Only a host link whose
  `<code>` matches the cup's stored host code unlocks editing.

The host writes the full tournament state to `cups/{cupId}` in the database
(debounced). Every open client subscribes and re-renders within a second or two of
each change. State is also cached to `localStorage` so a reload is instant and a
brief drop offline still shows the last-known board.

When the champion is first crowned, viewers are automatically pulled to the
champion screen so everyone sees the reveal together.

---

## One-time Firebase setup (~5 minutes)

1. Go to <https://console.firebase.google.com> and **Add project** (any name). You
   can skip Google Analytics.
2. In the left nav, open **Build ▸ Realtime Database ▸ Create Database**.
   - Pick a location.
   - Choose **Start in test mode** (see security note below).
3. Register a web app: **Project Overview ▸ Add app ▸ Web (`</>`)**. Give it a
   nickname; you do **not** need Firebase Hosting.
4. Firebase shows you a `firebaseConfig` object. Copy those values.
5. Open `index.html`, find the block near the top marked
   **`FIREBASE CONFIG`**, and paste your values in:

   ```js
   window.FIREBASE_CONFIG = {
     apiKey:            "AIza...",
     authDomain:        "your-project.firebaseapp.com",
     databaseURL:       "https://your-project-default-rtdb.firebaseio.com",
     projectId:         "your-project",
     storageBucket:     "your-project.appspot.com",
     messagingSenderId: "0000000000",
     appId:             "1:0000000000:web:abcdef",
   };
   ```

   > **`databaseURL` is required** for Realtime Database and is the one people most
   > often forget. If it's missing, live sync silently stays off and the app runs
   > local-only. You'll find it on the Realtime Database page; it ends in
   > `.firebaseio.com` or `.firebasedatabase.app`.

6. Host `index.html` anywhere your family can reach it over **https** (any static
   host works — Firebase Hosting, GitHub Pages, Netlify, etc.). The live share
   links use whatever URL the file is served from. Opening the file locally by
   double-click still works for testing, but phones on other networks need a real
   URL to reach it.

That's it — leave the config empty anytime you want to go back to pure offline use.

---

## Running a live cup

1. Open your hosted `index.html`. On the home screen tap **Start the Family Cup**.
   (With Firebase configured, this opens the lobby; with no config it just plays
   local-only.)
2. Name the cup (e.g. *Fourth of July 2026*) and tap **Create live cup**. This
   generates a random host code, writes the starting state to the database, puts
   you in host mode, and shows two copyable links plus a **Start the Family Cup →**
   button.
3. **Copy the view link and text it to everyone.** Keep the host link for yourself.
   Then tap **Start the Family Cup →** to begin.
4. Run the tournament from your phone. Tap a winner and every viewer's screen
   updates within a second or two — no refreshing.

The home screen shows a single primary button that adapts to context: **Start the
Family Cup** when nothing has been played, **Resume the Cup** once a game is in
progress, or **Watch live** for viewers. The reset button (↻, host only) clears the
game so the button returns to *Start*.

You can reopen the share links anytime from the **🔗** button in the top bar (host
only). To reopen an existing cup on another device, use its host link, or type its
id into **Open an existing cup** in the lobby (that opens it as a viewer).

---

## Security note (read before a bigger event)

**Test-mode rules are fine for a backyard family cup.** They let anyone with the
database URL read and write for ~30 days, which is exactly what you want for a
low-stakes event among people you trust. The practical control is simple: only
share the **view** link; keep the **host** link to yourself.

Be aware of the trade-off: with open rules, a determined guest who opens the
browser console could read or overwrite the data (including the host code, which is
stored with the cup). For a family lawn-games afternoon that's a non-issue.

**To lock it down later**, replace the test rules in **Realtime Database ▸ Rules**.
A reasonable step up that keeps the app working without adding a login is to scope
access to the `cups` path and stop rules from expiring:

```json
{
  "rules": {
    "cups": {
      "$cupId": {
        ".read": true,
        ".write": true
      }
    }
  }
}
```

For real access control you'd add Firebase Authentication and gate `.write` on the
authenticated host — beyond what a family event needs, but the path is there if you
grow into it.

---

## Files

- `index.html` — the entire app (styles, logic, and the Firebase sync bridge).
- `README.md` — this file.

### Editing the tournament format

Players, pools, rounds, and knockout games are defined in the `const T = { … }`
block near the top of the script in `index.html`. Editing them does not affect the
sync layer.
