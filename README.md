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

## Turn on live sync (optional, free) — Supabase

Without this, every device keeps its own local copy — an iPad won't show what you set up on your laptop. Adding a free Supabase project makes every device see the same data, live (the badge in the top bar changes from "Local only" to **Synced live**).

1. Go to [supabase.com](https://supabase.com) → **New project** (any name, any region, set a database password and save it somewhere).
2. In the left sidebar open **SQL Editor** → **New query**, paste this whole block, and click **Run**:
   ```sql
   create table public.team_sheets (
     id text primary key,
     payload jsonb not null,
     updated_at timestamptz not null default now()
   );
   alter table public.team_sheets enable row level security;
   create policy "team read"   on public.team_sheets for select to anon using (true);
   create policy "team insert" on public.team_sheets for insert to anon with check (true);
   create policy "team update" on public.team_sheets for update to anon using (true) with check (true);
   alter publication supabase_realtime add table public.team_sheets;
   ```
   (This lets anyone with your site link read/write that one table — fine for a small team tool with no sensitive data; the access code only keeps casual visitors out.)
3. Go to **Project Settings → API**. Copy the **Project URL** and the **anon public** key.
4. Open `index.html`, find these two lines, and paste the values between the quotes:
   ```js
   var SUPABASE_URL = "";
   var SUPABASE_ANON_KEY = "";
   ```
5. Commit and push — Vercel redeploys automatically.
6. **Open the site on the computer that already has your setup FIRST** and wait for the badge to say **Synced live** — that uploads the computer's data as the shared copy. Then open it on the iPad and your setup appears. After that, edits on either device show up on the other within a second or so.

Don't open a blank device first: if the shared copy doesn't exist yet, whichever device connects first with data becomes the shared copy.

## Access code

The page is gated by a simple access code — on first visit (per browser/device), a screen asks for it before the app shows. This isn't real security (the code lives in the page's own source, so anyone who looks at the file can read it), but it stops casual/accidental access, which is normally enough for a team tool and needs no separate login service.

To change the code, open `index.html` and find:
```js
var ACCESS_CODE = "DUCKS2027"; // change this to whatever code you want your team to use
```
Edit the text in quotes, commit and push. Set it to `""` (empty string) to turn the gate off entirely. A device that already entered the old code stays unlocked until it clears its browser data — that's fine for a code *change*, but if you need to actually lock out a device, you'd need to bump `ACCESS_STORAGE_KEY`'s name too (e.g. `"acroAccessUnlocked_v2"`) so it re-prompts everyone.

## Notes

- Everything runs client-side in plain HTML/CSS/JS — nothing to configure beyond the optional Supabase step above, no server code.
- Without Supabase configured, data entered on the page lives in that browser only — two people on two devices won't see each other's entries.
- Edit `index.html` directly to update the roster, skill tables, or point values — the data lives near the top of the `<script>` block in clearly labeled constants (`ATHLETES`, `COMPULSORY`, `TOSS_VALUES`, etc.).
