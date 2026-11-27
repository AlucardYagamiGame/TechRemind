# Architecture

This file records **how** the application is built and **why**. Keep it short and up to date: AI agents read it before every task (it is loaded automatically through `opencode.jsonc`), and so will anyone who reviews your project.

## Provided infrastructure

The starter contains infrastructure only, with no product features.

| Part | Location | Technology |
| --- | --- | --- |
| Frontend | `apps/web` | React 19, TypeScript, Vite, Tailwind CSS 4 |
| Backend | `apps/api` | Node.js, Express 5, TypeScript |
| Unit tests | `apps/*/src/**/*.test.ts(x)` | Vitest, React Testing Library, Supertest |
| End-to-end tests | `e2e/` | Playwright |
| Workspace | root `package.json` | npm workspaces |

### Frontend (`apps/web`)

- `src/main.tsx` renders `src/App.tsx`, the starter page.
- `src/api.ts` contains the functions that call the backend.
- Styling uses Tailwind utility classes (`src/index.css` imports Tailwind).

### Backend (`apps/api`)

- The whole API is in `src/server.ts`. It exports the Express `app` (so tests can use it without a network port) and starts listening only when the file is run directly.
- One endpoint exists: `GET /api/version` → `{ "name": "api", "version": "0.1.0" }`.
- In development Node.js runs the TypeScript file directly (`node --watch`); `npm run build` compiles it to `dist/`.
- In production (`npm start` runs `dist/server.js`) Express also serves the built frontend from `apps/web/dist` (`serveFrontend`). Every non-`/api` GET request gets `index.html`. The port comes from `PORT`, then `API_PORT`, then 3001.

## How the applications communicate

```
Browser ──► Vite dev server (localhost:5173) ──/api/*──► Express (localhost:3001)
```

- The frontend uses relative URLs such as `fetch('/api/version')`.
- In development, the Vite proxy (`apps/web/vite.config.ts`) forwards every `/api/*` request to Express. The browser sees a single origin, so no CORS setup is needed.
- Ports can be changed with the `API_PORT` and `WEB_PORT` environment variables.
- In production there is no Vite: Express serves the frontend and the API from the same origin, so relative `/api` URLs keep working and still no CORS is needed. The whole application deploys as one Node.js web service (build: `npm run build`, start: `npm start`).

## Decisions intentionally left open

These are yours to decide based on your requirements. Choose the simplest option that works, and record the choice below.

- **Data storage** — in memory, a JSON file, browser `localStorage`, or a database.
- **API design** — resources, URLs, request/response shapes, error format.
- **Backend structure** — when and how to split `server.ts` (routers, modules, validation).
- **Frontend structure** — pages, component folders, routing (whether you need a router at all).
- **State management** — React state and props first; add more only if needed.
- **Validation** — hand-written checks or a validation library.
- **Authentication** — only if your requirements really need it.
- **Deployment** — whether and where the application runs outside your computer (one Node.js web service is enough).

## How to document a decision

Add a short entry for every significant decision: new endpoints, data model, storage, dependencies, folder structure. Newest first.

```markdown
### YYYY-MM-DD — Short title

- **Context:** What problem or requirement led to this decision?
- **Decision:** What did you choose?
- **Alternatives:** What else did you consider, and why not?
- **Consequences:** What becomes easier or harder?
```

Also keep the "API endpoints" section up to date.

## API endpoints

| Method | Path | Description | Response |
| --- | --- | --- | --- |
| GET | `/api/version` | API name and version | `200 { name, version }` |

## Decision log

<!-- Add your decisions here, newest first. -->

### 2026-11-27 — PR title and description are generated

- **Context:** PR titles must follow Conventional Commits (they become the commit on `develop` after squash), and an empty PR description is useless for review.
- **Decision:** One GitHub Actions workflow `.github/workflows/pr.yml` (`actions/github-script`, no AI). If the title is invalid it generates one: `chore(release): <date>` for `develop` → `master`, else the first valid commit message, else the branch name. It fills the "Changes" section with the commits and changed files grouped by area, and fails if the final title is still invalid. `release` is an allowed scope, only for `chore(release)`.
- **Alternatives:** Copilot summary — manual and paid; an AI API in the workflow — paid key.
- **Consequences:** Summary, How to test and Checks are still written by hand. When an AI agent opens the PR it fills the whole template (`AGENTS.md`).

### 2026-11-27 — Git workflow: develop / master, squash, rebase

- **Context:** `master` is deployed, so it must only receive checked work.
- **Decision:** Work branches `<type>/<short-name>` from `develop`, PRs into `develop` with Squash and merge, releases `develop` → `master` with a merge commit. Work branches are updated with `git rebase origin/develop` and `git push --force-with-lease`; `develop` and `master` are never rebased. Full rules: `docs/git-workflow.md`.
- **Alternatives:** Only `master` — every merge would deploy unchecked work. Merging `develop` into work branches — noisy history.
- **Consequences:** History on `develop` is linear, one commit per PR. A rebase rewrites the branch, so it needs a force push.

### 2026-11-27 — CI and deploy are configured in the repository

- **Context:** Deploy must start automatically after a merge into `master`, but only if the code works.
- **Decision:** GitHub Actions `ci.yml` runs `npm ci` and `npm run verify` on every PR and on pushes to `develop` and `master`. Render is configured by the Blueprint `render.yaml` (region Frankfurt, branch `master`, health check `/api/version`, `autoDeployTrigger: checksPass`). Secrets use `sync: false` and are entered in the Render dashboard; `AUTH_SECRET` is generated by Render.
- **Alternatives:** Configuring the service by hand in the dashboard — settings are not visible or reviewable in the repo.
- **Consequences:** A red CI on `master` blocks the deploy. Changing a setting = editing `render.yaml` and releasing it.

### 2026-11-27 — Data storage: Neon PostgreSQL

- **Context:** User progress (question statuses and dates) is the core value of the app and must survive restarts and redeploys.
- **Decision:** Managed PostgreSQL on Neon (free plan). One Neon branch for local development, the main branch for the deployed app.
- **Alternatives:** In memory or JSON file — data is lost on restart on free hosting. MongoDB Atlas — the data is relational (users, topics, questions, per-user statuses).
- **Consequences:** Needs `DATABASE_URL` in `.env` and on Render; schema and migrations are decided in Step 5.

### 2026-11-27 — Hosting: Render free web service

- **Context:** A live link makes the portfolio project stronger; the starter runs as one Node.js service.
- **Decision:** One Render free web service (build `npm ci --include=dev && npm run build`, start `npm start`).
- **Alternatives:** Render PostgreSQL — expires after 30 days, so the database stays on Neon.
- **Consequences:** The service sleeps after 15 minutes without traffic; the first request then takes about a minute. No background jobs can be relied on.

### 2026-11-27 — Authentication: own email + password with HTTP-only cookie

- **Context:** Users need saved progress; requirements ask for simple email + password login only.
- **Decision:** Own authentication in the Express API: register, login, logout; passwords stored hashed; session in an HTTP-only cookie. Users have a role: `user` or `admin`.
- **Alternatives:** External auth service (Clerk, Auth0, Supabase Auth) — an extra service and keys for a simple requirement.
- **Consequences:** Needs `AUTH_SECRET` in `.env`. Frontend and API share one origin, so no CORS setup.

### 2026-11-27 — Status decay is calculated on read

- **Context:** `Known` becomes `For Review` after 14 days, `For Review` becomes `Unknown` after 30 days.
- **Decision:** Store the status and the date it was set; the current status is calculated when it is read.
- **Alternatives:** A scheduled job that updates statuses — does not run while the free Render service sleeps.
- **Consequences:** No scheduler; statuses are always correct at the moment they are shown.

### 2026-11-27 — Training sessions are not stored in MVP

- **Context:** Guests have no account; training history for Users is out of scope.
- **Decision:** The API builds the session queue; each User answer immediately updates that question's status; the Session Result is calculated on the client. Guests write nothing to the database.
- **Alternatives:** Storing sessions — needed only for training history, which is out of scope.
- **Consequences:** Adding training history later means adding a new table, without rewriting existing data.

### 2026-11-27 — Seeding

- **Context:** Public content lives in `data/seed.json` and must be updatable over time (new questions, enabling quiz per topic).
- **Decision:** The seed runs on every server start and never creates duplicates: topics are matched by `slug`, questions by their own `slug`. The seed never changes user progress. Two accounts are created on start: a test user (role `user`) and an admin (role `admin`); their emails and passwords come from environment variables, not from `seed.json`. The test user gets the demo progress from `seed.json` once, when the account is created; after that its progress changes like any other User's.
- **Alternatives:** Seeding by hand once — new questions and quiz updates would need a manual step after every deploy.
- **Consequences:** Updating public content = edit `seed.json`, push, redeploy. Editing a question's text keeps its progress because the `slug` does not change.
