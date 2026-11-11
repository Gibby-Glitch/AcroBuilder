# Ducks 2026-2027 Season — Acro Score Builder

A single-page, dropdown-driven scoring tool for the Level 3/4 Acro team: Compulsory (fixed SV 10.0, base/flyer role assignment), Optional (Acro/Pyramid/Toss/Tumbling point-code calculators), and Team Routine (segment builder — supports multiple synced performer groups per segment, up to 10, with move up/down ordering and a live requirements tally).

Five tabs: **Compulsory**, **Optional**, **Team Routine**, **My Export** (a plain-text sheet for you — who's in what position, for your own reference), and **Judges** (a printable scoring-grid layout styled after a standard meet judges' sheet — skill names/descriptions, athlete names, and start values, with blank score boxes). On the Judges tab, use the **Print / Save as PDF** button (or Ctrl/Cmd+P) to print or save it for handing to judges at a meet.

Any skill/score card has a "Skill/score not listed — enter manually" checkbox — check it to type in a custom skill name and point value instead of picking from the chart, for anything the booklet tables don't cover yet.

No build step — it's one static `index.html` file. Out of the box it saves to each browser's own `localStorage` (so a device keeps its own copy). Follow **Turn on live sync** below to make every device — iPad, laptop, whatever — see the same data update in real time.

## Put it on GitHub

```bash
cd acro-score-builder-site
git init
git add .
git commit -m "Acro score builder"
gh repo create acro-score-builder --public --source=. --push
```

(No `gh`? Create an empty repo on github.com, then `git remote add origin <url>` and `git push -u origin main`.)

## Deploy on Vercel

**Option A — Vercel dashboard:** go to vercel.com → New Project → Import the GitHub repo you just created. No framework preset needed ("Other" is fine) — Vercel will serve `index.html` as-is. Click Deploy.

**Option B — CLI:**
```bash
npm i -g vercel
cd acro-score-builder-site
vercel --prod
```

Either way you'll get a live `https://....vercel.app` URL you can share with coaches or open on a tablet at the gym.

## Turn on live sync (optional, free)

Without this, every device keeps its own local copy — an iPad filling out Group 1 won't show up on your laptop. Adding a free Firebase project makes every open tab/device see the same data update live (a small "Synced live" badge appears in the top bar once it's working).

1. Go to [console.firebase.google.com](https://console.firebase.google.com) → **Add project** → give it any name → you can decline Google Analytics.
2. In the left sidebar, open **Build → Firestore Database** → **Create database** → start in **production mode** → pick any region close to you.
3. Still in Firestore, go to the **Rules** tab and paste this (it allows read/write to just this one shared doc — fine for a small team tool that isn't handling sensitive data, but anyone with the link could technically read/edit it):
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /acroScoreBuilder/{docId} {
         allow read, write: if true;
       }
     }
   }
   ```
   Click **Publish**.
4. Back in the project, click the gear icon → **Project settings** → scroll to **Your apps** → click the **</>** (web) icon → register an app (no hosting needed) → copy the `firebaseConfig` object it shows you.
5. Open `index.html`, find this line near the top of the `<script>` block:
   ```js
   var FIREBASE_CONFIG = null;
   ```
   Replace `null` with the config object you copied, e.g.:
   ```js
   var FIREBASE_CONFIG = {
     apiKey: "AIza...",
     authDomain: "your-project.firebaseapp.com",
     projectId: "your-project",
     storageBucket: "your-project.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
6. Commit and push (`git add -A && git commit -m "enable sync" && git push`) — Vercel redeploys automatically. Reopen the site and the badge should read **Synced live**.

Everyone who opens that URL now shares one sheet.

## Access code

The page is gated by a simple access code — on first visit (per browser/device), a screen asks for it before the app shows. This isn't real security (the code lives in the page's own source, so anyone who looks at the file can read it), but it stops casual/accidental access, which is normally enough for a team tool and needs no separate login service.

To change the code, open `index.html` and find:
```js
var ACCESS_CODE = "DUCKS2027"; // change this to whatever code you want your team to use
```
Edit the text in quotes, commit and push. Set it to `""` (empty string) to turn the gate off entirely. A device that already entered the old code stays unlocked until it clears its browser data — that's fine for a code *change*, but if you need to actually lock out a device, you'd need to bump `ACCESS_STORAGE_KEY`'s name too (e.g. `"acroAccessUnlocked_v2"`) so it re-prompts everyone.

## Notes

- Everything runs client-side in plain HTML/CSS/JS — nothing to configure beyond the optional Firebase step above, no server code.
- Without Firebase configured, data entered on the page lives in that browser only — two people on two devices won't see each other's entries.
- Edit `index.html` directly to update the roster, skill tables, or point values — the data lives near the top of the `<script>` block in clearly labeled constants (`ATHLETES`, `COMPULSORY`, `TOSS_VALUES`, etc.).
