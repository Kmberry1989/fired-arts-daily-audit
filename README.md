# Fired Arts Daily Public-Info Audit

Live report: https://kmberry1989.github.io/fired-arts-daily-audit/

One repo for the daily Fired Arts Studio public-info audit report. Report #1
(Saturday, November 7, 2026) is the current report: **16 open issues**
(10 confusing for customers, 6 minor, 0 coverage gaps) and **3 resolved**
(Issues 15, 16, 17 — crossed out, kept for reference). Issue 19 (Nov 21
adult/teen flyer-vs-website times/prices) is the newest finding.

The report source of truth is the audit report + issues register kept with the
audit tooling; this repo hosts the published page only. The existing Muse
artifact ("Fired Arts Daily Public-Info Audit") is left alone — this Pages
site is the shareable, writable home for the same report.

## Read aloud

The sticky filter bar has a **Read aloud** button (speaker icon). It reads the
report aloud with the device voice — summary first, then the issues, sources,
events table, and what checked out fine — while the spoken word blooms (glow)
and zooms (scales up) and the page smooth-scrolls to keep it in view. Pause /
Resume and Stop sit next to it. Closed issue details open while reading and
are restored when it stops.

The feature lives in the page shell: the button in the filter bar, the
`.ra-word` / `.ra-active` styles, and the `READ-ALOUD` script block at the end
of `index.html`. Word spans are added at runtime only — the file on disk is
never rewritten by the feature, so the morning rebuild's exact-string
replacements still match. **Do not strip it on rebuild** (see the daily-audit
cron, step 7). Check: `node read-aloud-test.js` from
`~/workspace/goals/fired-arts-daily-public-info-audit/hidden_files/` (jsdom;
simulates the voice API and drives start / pause / resume / stop).

## How resolutions work

- **Firestore is the source of truth for user-initiated resolutions.**
  Collection `auditResolutions` in Firebase project `fired-arts-dashboard`,
  one document per issue, doc ID = the issue number as a string (`"1"`–`"19"`).
  Fields: `resolved` (boolean), `resolvedAt` (server timestamp),
  `resolvedBy` (email), `note` (string, optional).
- Sign in with Google (button at the top of the page), then tap a checkbox.
  Checked writes `resolved=true` with your email and a server timestamp;
  unchecked writes `resolved=false`. Tapping while signed out asks you to
  sign in first. No anonymous writes.
- **Write allowlist:** `kylematthewberry@gmail.com`,
  `rochelleberry731@gmail.com` (enforced in the page and in the Firestore
  rules; non-allowlisted sign-ins are signed out).
- On load the page reads the whole `auditResolutions` collection and renders
  from it, falling back to the baked-in report state for any issue with no
  document. **A page shows resolved if EITHER Firestore or the baked-in
  state says resolved.**

## Reconciliation rule (for the morning audit run)

A user's explicit Firestore state wins on the next run:

- If Firestore says `resolved=false` for an issue the register says was
  resolved-by-evidence, the cron **re-opens it** (the register/report go back
  to open, and the page shows it open once the baked-in state is rebuilt).
- Evidence-based resolutions are still recorded by the cron in the register
  and report.md as before — the cron is the only writer of baked-in state.
- The page OR-merges Firestore + baked-in state, so a Firestore
  `resolved=true` shows an issue resolved immediately even before the next
  morning's report rebuild.

## Setup notes

- Firebase web client config lives in `window.FIREBASE_CONFIG` at the top of
  the page script in `index.html` (project `fired-arts-dashboard`; public
  client config, recovered from the production Arts Fired bundle). If it is
  `null`, sign-in and write-back are off and the page shows the baked-in
  state only.
- Writes also need the Firestore rules for `auditResolutions` published and
  `kmberry1989.github.io` added under Authentication → Settings → Authorized
  domains in the Firebase console. Until both land, checkbox writes fail
  with a plain message on the page — the report itself still reads fine.
- GitHub Pages serves `index.html` from the `main` branch root. No build step.
