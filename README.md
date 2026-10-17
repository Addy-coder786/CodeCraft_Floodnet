# FloodNet — Urban Flood Nowcasting System (Mumbai pilot)

FloodNet forecasts **street-level flooding 0–3 hours ahead** for a ~4.7 km² pilot area around
**Hindmata / Dadar–Parel–Matunga, Mumbai**. It couples rainfall, terrain and the real underground stormwater
network in one simulation, then shows the result on a web dashboard and uses it to suggest flood-aware routes.

### 🌐 Live demo: **[code-craft-floodnet.vercel.app](https://code-craft-floodnet.vercel.app/)**

Hosted on Vercel (Mumbai region). Scenarios, the 2005 replay, the ECMWF forecast and the radar-derived source work
there; **live IMD rainfall is not enabled** on the demo (it needs an IP-registered IMD key — see
[`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md)). A full 180-minute run can take a minute or more.

```
rainfall ──► runoff ──► 2D surface flow on a 10 m terrain grid ⇄ drainage network (pipe capacity, surcharge, blockage)
                                        │
                                        ▼
        street-level depth (cm), 0–180 min ──► hotspots · alert drafts · flood-aware routing · dashboard
```

> **Research prototype — not an operational warning system.** The physics is deliberately simplified and several
> parameters are estimated (see [Limitations](#limitations)). No accuracy figure is claimed anywhere in this repo.

---

## Contents

- [Features](#features)
- [Repository layout](#repository-layout)
- [Quick start](#quick-start)
- [Running the tests](#running-the-tests)
- [Configuration](#configuration)
- [Data and provenance](#data-and-provenance)
- [Limitations](#limitations)
- [Documentation](#documentation)
- [Contributing](#contributing)
- [Licence](#licence)

## Features

- **Coupled hydrology** — rainfall → runoff → storage-cell 2D surface routing on a contour-derived terrain model,
  exchanging water both ways with a graph model of the real MCGM drainage network (Manning pipe capacity, manhole
  surcharge back onto the street, user-set blockage). Mass balance is reported for every run.
- **Rainfall sources** — design-storm scenarios, a replay of the 26 July 2005 Mumbai storm, live IMD station
  observations (needs credentials), the ECMWF forecast via Open-Meteo, and an experimental source decoded from the
  public IMD Mumbai-Veravali radar image. The active source is always named in the UI.
- **Street-level output** — depth in centimetres per road segment, every 5 minutes over a 180-minute default horizon.
- **Flood-aware routing** — direct vs. lower-flood-risk routes on the real OSM road network, per vehicle type.
- **Alerts & hotspots** — ranked hotspots and a CAP 1.2 alert *draft* (always for human review, never issued).
- **Dashboard** — React + Leaflet map, timeline, scenario/route/provenance panels, 2D/3D terrain view, guided
  briefing with voice (English / हिन्दी / मराठी).

## Repository layout

| Path | What it is |
|---|---|
| `backend/floodnet/` | Python package: terrain, drainage, rainfall, simulation, routing, alerts, FastAPI app |
| `backend/tests/` | pytest suite (`backend/scripts/` holds validation and demo-check scripts) |
| `frontend/` | Production dashboard — React + Vite + Leaflet + three.js |
| `frontend-prototype/` | Early plain HTML/JS prototype (served only if the React build is missing) |
| `data/processed/pilot/` | Pre-built pilot dataset the app loads at start-up (network, roads, terrain, scenarios) |
| `docs/` | Requirements, decisions, architecture, validation reports, source audits, deployment |
| `research/mumbai/` | Data-source research for the Mumbai pilot |
| `app.py`, `pyproject.toml`, `uv.lock`, `vercel.json` | Vercel deployment entrypoint and manifest |
| `Dockerfile` | Container image for the API |

## Quick start

### Prerequisites

- **Python 3.11+** (the Vercel manifest pins 3.12; local development has also been run on 3.14)
- **Node.js 20.19+ or 22.12+** (required by Vite 8)
- Git. No database, GPU or API key is needed for the default scenarios.

### 1. Clone

```bash
git clone https://github.com/Addy-coder786/CodeCraft_Floodnet.git
cd CodeCraft_Floodnet
```

### 2. Backend

```bash
cd backend
python -m venv .venv
# Windows:      .venv\Scripts\activate
# macOS/Linux:  source .venv/bin/activate
pip install -e ".[dev,radar]"
```

### 3. Frontend build

Build the frontend **before** starting the backend — the backend decides at start-up whether `frontend/dist`
exists and serves it if so.

```bash
cd ../frontend
npm ci
npm run build
```

### 4. Run

```bash
cd ../backend
python -m uvicorn floodnet.api.main:app --port 8000
```

Open **http://localhost:8000/static/** — pick a scenario, run it, scrub the timeline, and click two points on the
map for a flood-aware route. API docs: **http://localhost:8000/docs**.

### Frontend development (hot reload)

Keep the backend running on port 8000, then:

```bash
cd frontend
npm run dev        # http://localhost:5173, proxies /api to the backend on :8000
```

The pilot data in `data/processed/pilot/` is already built. To rebuild it from the original MCGM and OSM sources
(needs internet; the raw downloads are not stored in this repo):

```bash
cd backend
python -m floodnet.data.build_pilot
```

## Running the tests

```bash
# backend (from backend/, venv active)
python -m pytest tests -q

# frontend (from frontend/)
npm run test:unit
npm run lint
```

Tests that call real external APIs (IMD, ECMWF) are opt-in and skipped by default — see `backend/tests/` and
`docs/LIVE_RAINFALL_AUDIT.md`.

## Configuration

Everything works without configuration. Optional settings go in a `.env` file at the repo root — copy
[`.env.example`](.env.example), which documents every variable. The main ones:

| Variable | Purpose |
|---|---|
| `IMD_API_KEY`, `IMD_EMAIL`, `IMD_PASSWORD` | Live IMD observations. The IMD key is bound to one public IP. Without them, live mode reports itself as unavailable instead of substituting data |
| `GOOGLE_TTS_API_KEY` | Neural voice for the guided briefing (otherwise the browser voice is used) |
| `FLOODNET_RADAR_NOWCAST` | `1` enables the experimental radar-image nowcast |
| `FLOODNET_CORS_ORIGINS` | Allowed origins when the frontend is hosted separately |

**Never commit `.env`** — it is git-ignored. Never put secrets in `VITE_*` variables; Vite embeds them in the
public JavaScript bundle. Deployment (Vercel / Docker): [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md). The public demo
at <https://code-craft-floodnet.vercel.app/> is the all-in-one Vercel deployment described there.

## Data and provenance

Every component is tagged in code, API output and UI as REAL, ESTIMATED or SYNTHETIC.

| Component | Tag | Source |
|---|---|---|
| Drainage geometry, connectivity, sizes, inverts, ground levels | REAL | MCGM ArcGIS REST, BRIMSTOWAD layers 6 & 7 (snapshot 2026-09-08) |
| Terrain | REAL (interpolated) | MCGM 20 cm contours (layer 301) + manhole ground levels, 10 m grid |
| Roads, buildings | REAL | © OpenStreetMap contributors, ODbL |
| Historical flooding spots | REAL | MCGM Flooding Spots (layer 344), vintage ~2017 |
| Manning n, inlet capacity, manhole storage, impervious fractions | ESTIMATED | Stated rules / cited ranges in module docstrings |
| Design-storm scenarios | SYNTHETIC | `backend/floodnet/data/scenarios.py` |
| 26 July 2005 replay | REAL (transcribed) | Chitale Committee report |
| Blockage | SYNTHETIC scenario input | Never an observed condition |

MCGM data carries the copyright notice "MCGM, Esri India" and **no open licence was found**. Credit MCGM wherever
the data is shown. Full detail: [`data/processed/pilot/PROVENANCE.md`](data/processed/pilot/PROVENANCE.md).

## Limitations

- Storage-cell (diffusive) surface routing, capacity-based pipe hydraulics — not full Saint-Venant equations.
- Terrain is interpolated from 20 cm contours, not LiDAR; expect roughly ±10–20 cm depth uncertainty from terrain.
- Rainfall is applied uniformly over the pilot unless a spatial source is active; no tide at outfalls.
- Several hydraulic parameters are estimated (see the table above).
- Validated only against historical flooding-spot locations and an offline EPA SWMM cross-check — no depth or
  timing accuracy is claimed. Details: [`docs/VALIDATION.md`](docs/VALIDATION.md).

## Documentation

| Document | Contents |
|---|---|
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | System architecture |
| [`docs/REQUIREMENTS.md`](docs/REQUIREMENTS.md) | Problem-statement requirements and their status |
| [`docs/PRD.md`](docs/PRD.md) | Product requirements |
| [`docs/DECISIONS.md`](docs/DECISIONS.md) | Design decisions with evidence |
| [`docs/VALIDATION.md`](docs/VALIDATION.md) | What is and isn't scientifically validated |
| [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) | Vercel and Docker deployment |
| [`docs/FINAL_STATUS.md`](docs/FINAL_STATUS.md) | One-page current status |
| [`research/mumbai/`](research/mumbai/) | Data-source research |

## Contributing

Contributions are welcome — see **[CONTRIBUTING.md](CONTRIBUTING.md)** for setup, workflow and the project's
data-honesty rules.

## Licence

**No licence has been chosen yet.** Until a `LICENSE` file is added, default copyright applies and the code may not
be reused without the authors' permission. Third-party data keeps its own terms (OpenStreetMap: ODbL; MCGM: no
open licence found).
