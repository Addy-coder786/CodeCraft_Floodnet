# Contributing to FloodNet

Thanks for your interest in FloodNet. This guide explains how to set up the project, make a change and open a pull
request.

## Ground rules

FloodNet produces flood forecasts, so honesty about data matters more than features:

1. **Never fabricate.** No invented data, results, metrics, model performance or citations. If something is
   unknown, say so.
2. **Label what isn't real.** Anything assumed, estimated or synthetic must be tagged (`ESTIMATED`, `SYNTHETIC`) in
   code, API output and UI — follow the existing provenance tags.
3. **Check licences before adding data or dependencies.** Say where new data comes from and under what terms.
   Mark anything you could not verify as `[UNVERIFIED]`.
4. **Keep it simple.** Prefer the smallest change that solves the problem; extra complexity needs a reason.
5. **Never commit secrets.** `.env` is git-ignored — keep it that way. No API keys in code, tests or `VITE_*`
   variables.

## Getting set up

Follow the [Quick start](README.md#quick-start) in the README. In short:

```bash
git clone https://github.com/<your-username>/CodeCraft_Floodnet.git   # your fork
cd CodeCraft_Floodnet
cd backend && python -m venv .venv && <activate> && pip install -e ".[dev,radar]"
cd ../frontend && npm ci && npm run build
cd ../backend && python -m uvicorn floodnet.api.main:app --port 8000
```

## Workflow

1. **Open an issue first** for anything larger than a small fix, so the approach can be agreed before you write code.
2. **Fork** the repository and create a branch from `main`:
   ```bash
   git checkout -b fix/short-description      # or feat/…, docs/…, test/…
   ```
3. **Make your change**, with tests where it makes sense (see below).
4. **Run the checks** — all must pass:
   ```bash
   cd backend && python -m pytest tests -q
   cd ../frontend && npm run test:unit && npm run lint && npm run build
   ```
5. **Commit** using [Conventional Commits](https://www.conventionalcommits.org/), as the existing history does:
   ```
   feat(backend): add tide boundary at outfalls
   fix(frontend): keep timeline position after re-run
   docs: explain storage-area banding
   ```
6. **Push** to your fork and **open a pull request** against `main`. Describe what changed, why, and how you tested
   it. Link the issue it closes.

## Where things live

| Change | Look in |
|---|---|
| Physics (surface flow, pipes, runoff) | `backend/floodnet/terrain/`, `drainage/`, `simulation/` |
| Rainfall sources | `backend/floodnet/rainfall/` (`provider.py` holds the provider classes) |
| Routing | `backend/floodnet/routing/router.py` |
| API endpoints | `backend/floodnet/api/main.py`, `schemas.py` |
| Dashboard UI | `frontend/src/components/`, shared logic in `frontend/src/lib/` |
| Pilot data pipeline | `backend/floodnet/data/` (`build_pilot.py` rebuilds `data/processed/pilot/`) |

## Tests

- **Backend:** pytest in `backend/tests/`. Physics changes should keep mass balance closed and the scientific
  regression tests (`test_scientific_regression.py`) passing — if a number legitimately changes, explain why in
  the PR.
- **Frontend:** Node's built-in test runner, files named `*.test.mjs` next to the code in `frontend/src/lib/`.
- **Live API tests** (IMD, ECMWF) are opt-in and skipped by default; don't make the default suite depend on network
  access or credentials.

## Code style

- Match the surrounding code — naming, comment density and structure.
- Python: standard library and the existing dependencies first; explain any new dependency in the PR.
- JavaScript: the frontend is linted with `oxlint` (`npm run lint`).
- Keep documentation in `docs/` small and single-purpose; update the relevant doc when behaviour changes.

## Reporting bugs

Open an issue with the steps to reproduce, what you expected, what happened, and your OS / Python / Node versions.
Include the scenario and horizon if it's a simulation result.

## Security

If you find a security problem (for example a way to leak credentials), please **don't** open a public issue —
contact the maintainers privately through GitHub first.

## Licence

No licence has been chosen for this project yet (see the README). Until one is added, please ask the maintainers
before contributing substantial code, so the terms of your contribution are clear.
