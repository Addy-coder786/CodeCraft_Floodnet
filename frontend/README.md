# FloodNet dashboard (frontend)

React + Vite + Leaflet + three.js dashboard for FloodNet. It talks to the FastAPI backend in `../backend` over
`/api/*`. Setup and the full project overview: [root README](../README.md).

```bash
npm ci
npm run build      # production build in dist/, served by the backend at http://localhost:8000/static/
npm run dev        # dev server on :5173 with hot reload; proxies /api to the backend on :8000
npm run test:unit  # unit tests (src/**/*.test.mjs)
npm run lint       # oxlint
```

| Path | Contents |
|---|---|
| `src/components/` | UI components (map, timeline, panels, 3D terrain, guided briefing) |
| `src/lib/` | Shared logic: run bundles, alerts, rainfall source labels, story stages, terrain helpers |
| `src/state/FloodNetContext.jsx` | App-wide state and API calls |
| `src/api/client.js` | API client; `VITE_API_BASE_URL` only when the API is on another origin |
| `public/` | Browser icons |
