# Branch 2 Virtual Lobby — Setup Guide

A GitHub Pages web app for team attendance, backed by a private Google Sheet.

**Architecture (2 pieces, ~10 min setup):**

```
Staff phones/laptops
        │  (one link: yourname.github.io/lobby)
        ▼
   index.html  ← GitHub Pages (the pretty part)
        │  POST events / GET today's status
        ▼
   Code.gs     ← Apps Script Web App (the honest part)
        │
        ▼
  Private Google Sheet ("EventLog" tab)   ← only YOU can open this
```

Because the Sheet itself is never shared with staff, all the metadata
(IP, location, true server timestamps, edit history, user agent) is
invisible to them by default — the page only ever receives names,
times, and statuses back from the server.

---

## Step 1 — The backend (Google Sheet + Apps Script)

1. Create a **new** Google Sheet. Name it anything (e.g. "B2 Lobby Log"). **Do not share it.**
2. **Extensions → Apps Script**, delete the default code, paste in `Code.gs`, save.
3. **Deploy → New deployment → ⚙ → Web app**
   - Description: `lobby v1`
   - Execute as: **Me**
   - Who has access: **Anyone**  *(required so the GitHub page can reach it — the URL is unguessable, and it only accepts/serves attendance events)*
4. Click **Deploy**, authorize when asked, and **copy the Web app URL** (ends in `/exec`).

> Any time you later change `Code.gs`, use **Deploy → Manage deployments → ✏ → New version** so the same URL keeps working.

## Step 2 — The frontend (GitHub Pages)

1. In `index.html`, find the `CONFIG` block near the top of the `<script>`:
   - Paste your `/exec` URL into `scriptUrl`.
   - Replace the 15 placeholder employees with real names + IDs.
   - Photos: drop image files into your repo (e.g. `photos/ana.jpg`) and set `photo:'photos/ana.jpg'` — or leave `''` for auto-generated colored-initials avatars (they look intentional, not broken).
2. Create a GitHub repo (e.g. `lobby`), upload `index.html` (and your `photos/` folder).
3. Repo **Settings → Pages → Deploy from a branch → main / (root)** → Save.
4. Your link is `https://<username>.github.io/lobby/` — share it with the team.

## How the team uses it

| Situation | What they do |
|---|---|
| Morning sign-in | Tap your photo → **Telework** / **Office** / **More options…** (RDO, Off, Vacation, Sick Leave, Rotation, Site Visit) |
| Afternoon sign-out | Tap your photo → **Sign out for the day** |
| Dr. appointment mid-day | Tap your photo → **Step out for a bit** → later, tap again → **I'm back**. The gap is logged as its own pair of timestamps — no need to fake a sign-out. |
| Forgot to tap on time | 🕐 icon (or **Fix my times**) → set any of the four times to when it actually happened. Saved as a correction — the original stays in the log. |
| Something to tell the team | ✎ icon → post to **The Lobby Board**, shown to everyone for the day. |

The header ticker shows live counts of who's in / telework / office, and a rotating tagline keeps it human ("Load-tested for 15 engineers. Do not exceed rated capacity.").

## What lands in the Sheet

Every action appends one row to `EventLog`:

`Server Timestamp · Date · Employee ID · Name · Event · Status · Client Time · Edit Target · New Time · Note · IP · Approx. Location · User Agent`

It's **event-sourced**: edits never overwrite anything — a correction is a new `EDIT` row, so you always have the original tap time *and* what it was changed to. Building a weekly summary is then just a pivot table (or ask me and I'll add a formatted "Dashboard" tab generator).

## ⚡ Daily Transmission (Quote of the Day)

A panel above the badges shows one short motivating quote per day, plus a
collapsible, scrollable archive of the last 30 days with dates.

How it works — and why it goes through the backend:

- On the **first page load of each day** (by anyone on the team), the Apps
  Script server fetches the official quote-of-the-day from
  **ZenQuotes.io** (`https://zenquotes.io/api/today`) — a real internet
  quote feed, not AI-generated — and archives it in a new `QuoteLog` tab
  of your Sheet. A script lock prevents double-fetching if two people load
  simultaneously.
- Every load after that is served from the Sheet, so the **whole team sees
  the same quote and shares one 30-day archive** across all devices —
  better than per-browser storage.
- Each quote shows the author and a **source link** (ZenQuotes' free tier
  requires the link-back, which conveniently satisfies the attribution
  requirement).
- If ZenQuotes is ever unreachable, a curated bank of real, documented
  engineering quotes (von Kármán, Tredgold, Dyson, Edison, Royce, …) rotates
  deterministically by date instead — each linked to its Wikiquote source.
- The frontend caches the latest archive in `localStorage`, so the panel
  still paints instantly (with yesterday's data) before the network
  responds, and re-checks hourly in case a tab stays open past midnight.

To update an existing deployment: paste the new `Code.gs`, then
**Deploy → Manage deployments → ✏ → New version** (keeps the same URL),
and replace `index.html` in your GitHub repo.

## Honest limitations

- **Identity is honor-system.** The page is a public link with no login, so anyone with the URL could tap anyone's photo. You said the team is 15 honest engineers — for that, this is the right friction level. (The IP/location column gives you a sanity check; if you ever want real identity, the upgrade path is Google Sign-In on the page, which I can add later.)
- **IP location ≈ city-level at best**, and a VPN/hotspot skews it. Treat it as "home vs. office vs. traveling" signal, not GPS.
- If the ad-blocker/corporate network blocks `ipify.org`/`ipapi.co`, the IP fields just come back blank — attendance still logs fine.
- The `/exec` URL is unguessable but technically public — don't post it anywhere outside the page's source.

## Room to grow (you mentioned future tools)

The page is structured so the badge grid is just one `<section>` — new tools (links panel, VISION shortcuts, on-call rotation, birthdays, a BDPPM quick-reference) can be added as more sections under the Lobby Board without touching the attendance logic. The backend accepts a `type` field, so new event kinds are one `VALID_TYPES` entry away.
