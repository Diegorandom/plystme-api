# Plyst Me — API for a music recommendation platform

The backend API for Plyst Me, a music recommendation platform: Spotify Web API integration, user data, and the endpoints the product ran on.

- `index.js` — server · `requirements.txt` — the Python side of the stack · `newrelic.js` — APM
- `CONTRIBUTING.md` / `CODE_OF_CONDUCT.md` — the project's contribution rules

**Context:** 2019, where I was CTO. This is product code, not an exercise: it shipped, and the recommendation model was part of my responsibility.

**Security note (2026-11-10):** `secret-config.json` held a session secret and was committed in 2021.
Every branch in this repository has been rewritten to remove it, and the credential has been rotated.
GitHub keeps pull-request references immutable, so the old blob may still be reachable from some
`refs/pull/*` entries; it no longer grants access to anything.
