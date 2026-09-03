# FlowState

FlowState is an airflow and indoor air quality planner. It simulates how CO₂, airborne particles ("virus" units), and temperature spread through a room based on fan placement, window ventilation, and occupants, then recommends where to put a fan to keep the air cleanest.

## How it works

- You define a room (in feet), then add **fans**, **windows**, and **occupants** by dragging them onto a canvas.
- The backend runs a simplified physics simulation (advection + diffusion + sources/sinks) over a 28×44 grid to model how CO₂, particles, and heat move through the space.
- Results render as a color-coded heatmap (green to yellow to red) with optional airflow vector arrows.
- An optimizer can brute-force test fan positions and recommend the one that minimizes CO₂/particle buildup.
- Outdoor air quality (PM2.5, PM10, NO₂, O₃, CO) can be pulled in from the free [Open-Meteo Air Quality API](https://open-meteo.com/) based on your browser's geolocation.

## Project structure

```
FlowState/
├── backend/
│   ├── server.js        # HTTP server + routing (static files + API endpoints)
│   ├── simulation.js    # Core grid-based airflow/CO2/virus/temp simulation
│   ├── optimizer.js     # Random-search fan placement optimizer
│   ├── airquality.js    # Fetches outdoor air quality from Open-Meteo
│   └── standards.js     # CO2 thresholds (good/moderate/poor/dangerous)
├── frontend/
│   ├── index.html       # Landing page
│   ├── home.html        # Alternate landing page
│   ├── sim.html         # Main simulation UI (room canvas + controls)
│   ├── sim.js            # Simulation UI logic (drag/drop, API calls, drawing)
│   └── style.css        # Shared styling
├── package.json
└── package-lock.json
```

## Getting started

**Requirements:** Node.js 18+ (uses the built-in `fetch` API).

```bash
# from the repo root
npm install

# start the server
node backend/server.js
```

The server listens on **http://localhost:3000**.

- `http://localhost:3000/` for the landing page
- `http://localhost:3000/sim` for the simulation tool
- `http://localhost:3000/health` for a health check

> **Note:** `package.json` lists `express` as a dependency, but `backend/server.js` is currently implemented with Node's built-in `http` module rather than Express. Either remove the unused dependency or migrate `server.js` to use it, depending on which direction you want to take the project.

## API endpoints

| Method | Route          | Description                                                        |
|--------|----------------|----------------------------------------------------------------------|
| GET    | `/health`      | Returns `{ ok: true }`                                              |
| GET    | `/airquality`  | Query params `lat`, `lon`; returns outdoor PM2.5/PM10/NO₂/O₃/CO   |
| POST   | `/simulate`    | Body: room dimensions, fans, windows, occupants, outdoor conditions; returns the CO₂/virus/temp grid and stats |
| POST   | `/optimize`    | Same-shaped body; returns the recommended fan position and its score |

## Simulation model (`simulation.js`)

- Room is discretized into a 28×44 grid.
- Each step computes a velocity field from fans (radial push) and windows (directional ventilation), advects and diffuses the CO₂/virus/temperature fields, adds emissions from occupants, and pulls values near open windows toward outdoor baselines.
- Runs for 40 timesteps per simulation call.
- Outdoor CO₂ defaults to about 420 ppm (`standards.js`).

## Optimizer (`optimizer.js`)

Runs 80 random fan placements through the simulation and picks the one with the lowest weighted score across average/max CO₂, average/max virus concentration, and deviation from a comfortable 21°C.

## License

ISC (see `package.json`).
