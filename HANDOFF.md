# Morning Circuit — Handoff Notes

Context for picking this up locally and building on it. Written for an agent/dev with no prior context on the project.

## What this is

A installable PWA (Progressive Web App) — a website that installs to a phone home screen, works offline, and opens full-screen like a native app. No App Store, no build tooling, no framework. Plain HTML/CSS/vanilla JS in one file plus two small PWA support files.

Current feature: a guided morning mobility/activation circuit (8 exercises) with an auto-advancing countdown timer, rounds selector, and a daily streak tracker.

## Files

```
fit-tracker/
├── index.html       # everything: markup, styles, app logic
├── manifest.json     # PWA metadata (name, icons, colors, display mode)
├── sw.js              # service worker — caches the app shell for offline use
├── icon-192.png       # home screen icon
└── icon-512.png       # larger icon (splash / Android)
```

Everything is self-contained in `index.html` — no build step, no npm, no bundler. Open it directly in a browser to test locally (though the service worker won't register over `file://`, see below).

## How data is stored right now

`localStorage`, plain browser storage, no backend, no login. Key used: `routine-log`, a JSON array of ISO date strings (`"2026-08-17"`) representing days the circuit was completed. Streak is computed client-side by walking backward from today.

**Important implication:** data is per-device, per-browser. No sync across phones. If this grows into a multi-workout tracker, decide early whether that's still acceptable or whether a real backend becomes worth it (see Roadmap).

## Deploying to GitHub Pages (first-time setup)

1. **Create the repo**
   - GitHub → **+** → **New repository**
   - Name it (e.g. `fit-tracker`)
   - **Public** — GitHub Pages' free tier requires this. Public ≠ discoverable: there's no browse/discovery feed, nobody finds it without the exact URL or repo name.
   - No README needed, can start empty.

2. **Push the files** (once developing locally, this replaces the drag-and-drop upload):
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<username>/fit-tracker.git
   git push -u origin main
   ```

3. **Enable Pages**
   - Repo → **Settings** → **Pages** (left sidebar)
   - Source: **Deploy from a branch** → Branch: **main**, folder: **/ (root)** → **Save**
   - Live URL appears in a minute or two: `https://<username>.github.io/fit-tracker/`

4. **Keep it low-profile (optional, since repo is public)**
   - Add a `robots.txt` at the repo root with:
     ```
     User-agent: *
     Disallow: /
     ```
     This stops search engines from indexing the hosted site, so it won't surface in Google even though the repo is technically public.
   - Code is copyrighted to the author by default the moment it's written — no LICENSE file is required for that protection, but adding one that says "All rights reserved" makes it explicit to anyone who does stumble on the repo.

5. **Install on phone**
   - Open the `github.io` URL in Safari (iOS) or Chrome (Android)
   - iOS: Share icon → **Add to Home Screen**
   - Android: ⋮ menu → **Add to Home Screen** / **Install app**

## PWA gotchas worth knowing before editing

- **Service worker caching**: `sw.js` caches the app shell on install. If you edit `index.html` and just re-push, phones that already installed the app may keep serving the *old* cached version until the cache is invalidated. Bump the `CACHE_NAME` constant in `sw.js` (e.g. `fit-tracker-v1` → `v2`) on every deploy that changes cached files — this forces the old cache to be dropped and the new files fetched.
- **Local file testing**: opening `index.html` directly (`file://`) will NOT register the service worker — service workers require a secure context (`https://` or `localhost`). Use a local dev server (`python3 -m http.server`, or the VS Code Live Server extension) to test PWA behavior locally, or just push and test on the live Pages URL.
- **iOS Safari limits**: no Vibration API support at all (Android Chrome does support it) — rest-timer alerts on iPhone can only be sound/visual, not haptic, unless this becomes a native app later.
- **HealthKit (Apple Watch steps, RENPHO weight data)**: not reachable from a website at all, PWA or not — that data only exists on-device via Apple's HealthKit, which requires a native Swift app. Out of scope for this PWA; noted here so it isn't attempted and abandoned.

## Roadmap discussed (not yet built)

1. **Workout tracker** — push/pull/legs day logging: exercise name, sets, reps, weight per set, session history. Same localStorage pattern as the streak log, just a richer data shape (e.g. array of session objects with date, day-type, and an array of {exercise, sets: [{reps, weight}]}).
2. **Rest timer between sets** — reuse the existing ring-countdown timer component, retarget for arbitrary short durations (e.g. 60s) triggered per-set rather than the fixed circuit sequence. Add Vibration API call (`navigator.vibrate(...)`) guarded by a feature check, since it's Android-only.
3. **Apple Watch steps / RENPHO weight sync** — deferred; requires a native iOS app (Swift, HealthKit entitlement, Apple Developer account) if this becomes a priority later. Not achievable as a web app.

## Design tokens (for consistency if extending the UI)

- Background `#12141C`, surface `#1B1F2C`, surface-2 `#232838`, line `#2C3244`
- Accent amber `#E3A857` (active/primary), sage `#7FA88C` (completed/success), coral `#D9765A` (used sparingly — currently flags "back-friendly" exercises)
- Text `#EDEFF3`, muted `#8B93A7`
- Fonts: Bricolage Grotesque (display/headings), Inter (body/UI), JetBrains Mono (numbers/timer/data) — loaded via Google Fonts `<link>` in the `<head>`
