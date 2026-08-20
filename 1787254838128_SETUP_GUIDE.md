# RHI-PSCC Group Tournament — Setup Guide

This is the new Group-Tournament format (Experts captain Groups, cross-group matches
in Mixed/Intermediate/Novice/Singles categories) as a standalone, single-file app —
same architecture pattern as the existing RHI-PSCC event pages (live Firestore sync,
real Firebase Auth admin login, GitHub Pages hosting), rebuilt from scratch per your
request rather than modified in place, so the existing `/2026/badminton/` doubles
bracket keeps running completely untouched.

## What's different from the doubles-bracket app

- **Roster has ranks.** Every player is Expert, Intermediate, or Novice, not just a name.
- **No single-elimination playoff.** This format is pure round-robin: every Expert
  captains a Group, everyone else is dealt into a Group, and every match is
  cross-group only (a Group's own teams never play each other). Standings are just
  total wins across all matches — no bracket advancement.
- **"Shuffle" + "move people around" is two steps, matching what you asked for:**
  1. Admin clicks **Shuffle into groups** — this randomly deals Intermediate and
     Novice players into Groups (evenly, respecting rank), and randomly assigns
     which Expert captains which Group.
  2. Admin then opens the **Groups & teams** tab, where every single slot has a
     dropdown — exactly like the workbook's Team Assignments table. Move anyone,
     anywhere, anytime — a late add, a no-show, a skill-level correction, or just
     rebalancing. Nothing is locked after the shuffle.
- **Ties are allowed and expected** (matches the workbook's "Tie · replay" rule) —
  a tied score counts as played but awards no point to either side.

## Quick path — this reuses the same Firebase project as your other events

Per your confirmation, this keeps the same `rhi-badminton` Firebase project and the
same admin login as the existing doubles-bracket app, but writes to a **separate**
Firestore document (`TOURNAMENT_ID = "rhi-aug21-groups"` vs. the old app's
`"rhi-aug21"`), so nothing about this build can read, write, or otherwise affect the
old tournament's data. The only thing shared is the admin email/password — that's
just a login credential, not a data link.

1. Create a new repo (per your call to build this from scratch rather than adding
   to `rhi-pscc.github.io`) and add this `index.html` at whatever path you want the
   URL to have, e.g. `/index.html` for the repo root, or `/2026/group-tournament/`
   to mirror the existing repo's folder convention.
2. Enable GitHub Pages for that repo (Settings → Pages → deploy from `main`).
3. Push. Share the resulting `https://<your-username>.github.io/<repo>/` URL —
   that's the whole rollout, no separate hosting or database setup needed, since
   Firebase is already wired in.
4. The same admin email/password from the other RHI-PSCC events works here too —
   nothing to recreate in the Firebase console.

If you'd rather this be on a completely separate Firebase project instead (zero
shared infrastructure, not just a separate document), let me know — that needs you
to create a new free Firebase project yourself (project creation requires your own
Google sign-in), and I'll wire its config into `index.html` in place of the current
block.

## How it works day-to-day

1. **Roster tab**: paste or type in the full roster (`Name, Rank` per line, or add
   one at a time). This is also where you fix exceptions later — add a late signup,
   remove a no-show, correct someone's skill level.
2. Click **Shuffle into groups**. Every Expert captains a Group; Intermediate and
   Novice players are dealt in evenly and paired into teams automatically.
3. **Groups & teams tab**: review the auto-built lineup. Every slot is a dropdown —
   move any player to fix an exception, rebalance, or just because you want to.
   A self-audit banner warns if anyone ends up unassigned or double-booked.
4. **Schedule tab**: set court count and minutes-per-match; matches are automatically
   packed into rounds (a round never double-books a team or an Expert). Enter
   scores here as matches finish — the winning Group is derived from the score
   automatically, same as the doubles-bracket app.
5. **Standings tab**: live-updating Group point totals and a champion banner once
   every match has a result.
6. Everyone else just opens the link — read-only, live-updating, no sign-in needed.
   The **"Find your match"** bar works the same way as the other event pages.

## Verification note

The Group/Team engine (who plays whom, how many rounds are needed) was tested with
a standalone randomized harness — 800 fuzzed roster shapes across 1–6 Experts and
0–19 Intermediate/Novice players each, at 5 different court counts — with zero
scheduling conflicts (no Expert or team ever double-booked in the same round). It
also reproduces the source workbook's numbers on the actual Aug 21 roster exactly:
15 matches, 6-round minimum at 3 or 4 courts (the 4th court doesn't help — same
"only 1 Expert-match at a time" bottleneck the workbook identified), 15 rounds at
1 court.

## Changing things later

- **Reset an event**: Firebase console → Firestore Database → `tournaments`
  collection → delete the `rhi-aug21-groups` document. The app recreates it fresh
  next time someone saves.
- **Admin password**: change it anytime in Firebase console → Authentication →
  Users — updates for every event sharing this project at once, no redeploy needed.
