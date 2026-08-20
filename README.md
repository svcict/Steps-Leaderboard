# Step Up Challenge — Dashboard (GitHub Pages edition)

Public leaderboard, admin-only CSV upload — no server required.

## How it works

- **Anyone** who opens the page fetches `data/steps.csv` (a plain file in
  this repo) and sees the leaderboard. No login needed. If that file
  doesn't exist yet, they see "No results yet."
- **Admins** click **Admin**, paste in a GitHub personal access token that
  has write access to this repo, and upload a CSV. The browser commits
  that file straight to `data/steps.csv` via GitHub's API — GitHub Pages
  then rebuilds automatically (usually well under a minute) and everyone
  sees the update.

There's no traditional password here. The real gate is: **only someone
who can generate a token with write access to your repo can publish.**
That's normally just you and anyone you've added as a collaborator.

## 1. Create the repo and enable Pages

1. Push this folder to a new GitHub repo (public or private — Pages works
   for both, though a private repo needs GitHub Pro/Team/Enterprise to
   serve a public site).
2. In the repo: **Settings → Pages → Build and deployment → Source** =
   "Deploy from a branch." Pick the branch (e.g. `main`) and folder
   (`/root`), then save.
3. GitHub will give you a URL like
   `https://YOUR-USERNAME.github.io/YOUR-REPO/`. That's your live link.

## 2. Point the dashboard at your repo

Open `index.html`, find this block near the bottom, and fill in your repo:

```js
const GH_OWNER  = 'YOUR-GITHUB-USERNAME';
const GH_REPO   = 'YOUR-REPO-NAME';
const GH_BRANCH = 'main';
const DATA_PATH = 'data/steps.csv';
```

Commit and push. Pages will rebuild automatically.

## 3. Create a token for whoever will publish updates

Each admin needs their own **fine-grained personal access token**:

1. GitHub → Settings → Developer settings → Personal access tokens →
   Fine-grained tokens → **Generate new token**.
2. **Repository access**: "Only select repositories" → pick this one repo.
   Do not grant access to all repos.
3. **Permissions**: under Repository permissions, set **Contents** to
   **Read and write**. Leave everything else at "No access."
4. Set an expiration (30–90 days is reasonable — you'll just generate a
   new one when it expires).
5. Generate, copy the token (starts with `github_pat_`), and give it only
   to people who should be able to publish. Treat it like a password.

Paste that token into the **Admin** login on the site when uploading.
It's kept in the browser's memory only for that session — never saved,
never sent anywhere except `api.github.com`.

## CSV format

Same as before — a `Name` column plus one column per day, headers like
`2026-08-01`. Missing days can be blank or `N.A`.

## Notes on the security model

- The token is typed directly into the page and used client-side to call
  GitHub's API. That's inherently less protected than a server holding
  the credential — a malicious browser extension or a successful XSS
  attack on the page could see it. Fine-grained tokens scoped to just
  this repo, with **Contents: read-write only** and a short expiration,
  limit the damage if that ever happens. Don't use a classic
  all-repos token here.
- Every publish is a real git commit to `data/steps.csv`, so you get a
  free history/audit trail and can revert a bad upload from GitHub's
  commit history at any time.
- If you'd rather skip the in-page token flow entirely, you can always
  just edit `data/steps.csv` directly in GitHub's web UI (Edit → Commit)
  — Pages will rebuild the same way. The Admin login in the app is a
  convenience, not the only way to publish.
- If you outgrow this (need real per-person accounts, upload logs,
  instant updates, private data, etc.), that's the point where moving to
  an actual backend — Render, Vercel, or similar — with a proper
  database makes more sense than stretching the static-site model
  further.
