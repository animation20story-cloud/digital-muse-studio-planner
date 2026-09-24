# Digital Muse Studio — Planner Creation Studio

A single-file, zero-build front end that implements the full Planner Creation Studio spec: planner type & format, required title/content, grouped true multi-select page/section types, sections, layout, aesthetic, typography, color, decorative elements, Custom Value everywhere, Exclude, Smart Randomize with cohesion logic (Creative Direction Lock), two generated blueprint versions, Copy/Save/Favorite, and real navigation to Saved & Favorites / History / Sign Out.

## What's included

- **`index.html`** — the entire app (HTML + CSS + JS, no framework, no build step).

## Run it locally

Just open `index.html` in a browser, or serve it:

```bash
npx serve .
```

## Deploy to GitHub + Vercel

1. Create a new GitHub repo and push this folder:
   ```bash
   git init
   git add .
   git commit -m "Digital Muse Studio — Planner Creation Studio"
   git branch -M main
   git remote add origin https://github.com/<you>/<repo>.git
   git push -u origin main
   ```
2. Go to [vercel.com/new](https://vercel.com/new), import the repo.
3. Vercel auto-detects a static site (no framework, no build command needed) — click **Deploy**. That's it.

You can also deploy straight from the CLI without GitHub:

```bash
npm i -g vercel
vercel --prod
```

## How the current build behaves

- **Studio / Saved & Favorites / History / Sign Out** are real navigation (hash-routed views — `#studio`, `#saved`, `#history`), not decorative buttons or scroll links.
- **Saved & Favorites and History persist** in the browser via `localStorage`, so generated concepts, saves, and favorites survive a refresh and are restorable ("Open Concept" / "Open in Studio").
- **Sign Out** clears the current session lock (`sessionStorage`) only — it never touches saved/favorites/history data, matching the spec's requirement that signing out is non-destructive.
- **Smart Randomize** picks a cohesive "aesthetic family" (e.g. Glam & Luxe, Girly & Sweet, Vintage & Retro, Clean & Modern, etc.), then pulls typography/colors/decorative elements/components from that same family, respects anything the user already picked, and filters out anything listed in Exclude. Version 2 stays inside the same family and only varies layout emphasis, typography order, color emphasis, and decorative placement — never contradicting Version 1's direction (Creative Direction Lock).
- **Custom Value** is a real text input everywhere the spec requires it, and custom text flows directly into Your Selections and the generated blueprint — nothing is collected and discarded.

## What's a placeholder vs. production-ready

This build is fully functional as a client-side app, but two things are explicitly stubbed for demo purposes and are called out in the UI itself:

1. **Sign-in / access code check** — currently accepts any non-empty code. Swap the check inside `access-unlock-btn`'s click handler for a real API call to your `/api/verify-access` route once that's live.
2. **Data persistence** — currently `localStorage` (per-browser, not per-customer). The three functions that touch storage — `saveConcept()`, `pushHistory()`, and `getSaved()/getHistory()` — are isolated on purpose so they can be swapped for `fetch()` calls to Supabase-backed API routes without touching any of the generator/randomize/blueprint logic.

## Production architecture (per spec)

```
Beacons purchase
      ↓
Stripe checkout.session.completed
      ↓
Vercel webhook  /api/issue-access
      ↓
Generate unique permanent customer access code
      ↓
Store customer/access info in Supabase
      ↓
Resend sends personalized customer email
      ↓
Customer clicks "✨ Open Planner Creation Studio"
      ↓
Customer enters private access code
      ↓
Planner Creation Studio unlocks
```

To wire this up for real:

- Add `/api` routes (Vercel Serverless/Edge Functions) for `issue-access` (Stripe webhook target) and `verify-access` (checked on sign-in).
- Add a Supabase table for customers/access codes and one for saved concepts/favorites/history keyed by customer id.
- Point `saveConcept()`, `pushHistory()`, `getSaved()`, and `getHistory()` at `fetch()` calls to your own API routes instead of `localStorage`.
- Keep the access code in `sessionStorage`/an httpOnly cookie once verified server-side, and re-check it on load.

None of that requires rebuilding the generator — the UI, Smart Randomize logic, and blueprint generation are already fully decoupled from storage.
