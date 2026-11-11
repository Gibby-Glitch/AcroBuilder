# Acro Score Builder

A single-page, dropdown-driven scoring tool for the Level 3/4 Acro team: Compulsory (fixed SV 10.0, athlete assignment), Optional (Acro/Pyramid/Toss/Tumbling point-code calculators), and Team Routine (segment builder with a live requirements checklist). Includes a plain-text export view for judges.

No build step, no dependencies, no backend — it's one static `index.html` file. Everything a viewer enters is saved only in their own browser (`localStorage`), so each coach/device keeps its own in-progress sheet.

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

## Notes

- Everything runs client-side in plain HTML/CSS/JS — nothing to configure, no environment variables.
- Data entered on the page lives in that browser only. There's no shared database, so two people on two devices won't see each other's entries. Ask if you'd like that added (it would need a small backend or a service like Supabase/Firebase).
- Edit `index.html` directly to update the roster, skill tables, or point values — the data lives near the top of the `<script>` block in clearly labeled constants (`ATHLETES`, `COMPULSORY`, `TOSS_VALUES`, etc.).
