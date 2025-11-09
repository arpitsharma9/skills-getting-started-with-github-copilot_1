## Quick orientation — what this repo is

- Small FastAPI app that serves a single-page frontend and a tiny REST API.
- Static UI lives in `src/static/` and is mounted at the `/static` path by the server (see `src/app.py`).
- API is implemented in `src/app.py` with a minimal in-memory data store `activities` (a dict keyed by activity name).

## Big picture architecture (files to read first)

- `src/app.py` — the entire server and API surface. Key behaviors:
  - `app.mount('/static', StaticFiles(...))` — static assets served at `/static`.
  - `GET /activities` — returns the whole `activities` dict.
  - `POST /activities/{activity_name}/signup?email=...` — appends the email to `activities[activity_name]['participants']`.
  - Root `/` redirects to `/static/index.html`.
- `src/static/index.html`, `src/static/app.js` — the frontend. The JS calls `/activities` and POSTs to `/activities/{name}/signup` using a query parameter `email`.

## Important patterns & conventions (exact, not aspirational)

- In-memory data: `activities` is a plain Python dict (no Pydantic models). Code and tests (if added) should treat activity name as the unique identifier.
- Student identity: email strings are used as the student identifier in participant lists.
- No persistence: changes to `activities` are lost on restart. Any feature that needs persistence must add a DB or file store and update both the API and the UI.
- API input style: the signup endpoint expects `email` as a query parameter (example: `POST /activities/Chess%20Club/signup?email=a@b.com`). Keep this when modifying the frontend or routes unless intentionally changing the contract.

## Useful examples (copy-paste friendly)

- Fetch activities from JS (see `src/static/app.js`):

  - GET `/activities` returns a JSON object shaped like the `activities` dict in `src/app.py`.

- Signup from JS (see `signup` flow in `src/static/app.js`):

  - POST `/activities/${encodeURIComponent(name)}/signup?email=${encodeURIComponent(email)}`

  - On success the server returns JSON `{ "message": "Signed up ..." }` and 200 OK.

## How to run locally (commands used in this repo)

- Install dependencies:

  pip install -r requirements.txt

- Run in development with auto-reload (recommended):

  uvicorn src.app:app --reload --host 0.0.0.0 --port 8000

- When running, UI is available at `/static/index.html` (root redirects there). API docs: `/docs` and `/redoc`.

## Where to make common changes

- Add or change activities / default data: edit `activities` in `src/app.py`.
- Add an API route: modify `src/app.py` (no routing modules yet). Keep to the same style (function-based FastAPI endpoints).
- Modify UI behavior: edit `src/static/app.js` and `src/static/index.html`. Network calls are relative (`/activities`), so the dev server and production server should serve both API and static assets at the same origin.

## Integration points & external dependencies

- Dependencies are minimal: `fastapi` and `uvicorn` (see `requirements.txt`).
- No external services are integrated (no DB, auth provider, or cloud API) — tests or CI that assume external integrations must add mocks/stubs.

## Developer workflows and debugging tips

- Quick sanity check: start uvicorn and open `http://localhost:8000/static/index.html` to exercise both the frontend and API together.
- Debugging: because the app uses `uvicorn` + FastAPI, use the `--reload` flag during development for live reloads.
- Adding tests: there are currently no tests. When adding tests, prefer `pytest` and use the FastAPI `TestClient` to exercise endpoints in memory.

## Notes for AI agents (how to be helpful here)

- Preserve the API contract unless the issue explicitly requires a contract-breaking change. The frontend (`src/static/app.js`) depends on the current contract (query param `email`, `/activities` shape).
- Use `src/app.py` and `src/static/app.js` as the authoritative examples for request/response shapes and error handling.
- If adding persistence, update both `src/app.py` (server-side data source) and `src/static/app.js` (no change required for endpoints, but tests must adapt).
- Keep changes minimal and well-scoped: this repo is intentionally tiny—follow its existing patterns instead of introducing heavy frameworks.

---

If you'd like, I can: (1) open a PR with this file, (2) add a tiny unit test that exercises `GET /activities`, or (3) convert the `activities` dict into a small JSON-backed persistence layer — tell me which and I'll proceed.
