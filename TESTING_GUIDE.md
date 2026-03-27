# HyperAgent HTML Prototype - Testing Guide

This guide explains how to test the single-file `index.html` prototype and how to generate synthetic data you can upload to it.

## 1) Run the prototype

Because this is a single HTML file, you can run it in two ways:

### Option A (quick): open directly
- Double-click `index.html` in your file explorer.

### Option B (recommended): serve locally
```bash
python -m http.server 8000
```
Then open: `http://localhost:8000/index.html`

---

## 2) Basic end-to-end smoke test

1. **Connect device**
   - In section **Device Connection**, set:
     - Device ID: `hvac_main`
     - Type: `HVAC`
     - Protocol: `REST`
   - Click **Connect Device**.

2. **Load telemetry**
   - Click **Load Demo Dataset** OR upload a CSV in section **Hardware Data Upload**.

3. **Generate actions**
   - Click **Run 1 Decision Cycle**.
   - Confirm rows appear in decision table.

4. **Validate learning**
   - Set feedback score to `0.6` then click **Apply Feedback**.
   - Confirm policy weights change.

5. **Validate API facade**
   - In API Playground, call:
     - `GET /api/v1/devices`
     - `GET /api/v1/telemetry`
     - `GET /api/v1/decisions`
     - `GET /api/v1/evolution`

6. **Validate DGM mode**
   - Enable DGM mode and click **Run Evolution Cycle** a few times.
   - Confirm generation and fitness updates.

7. **Validate dashboard page**
   - Click **Open Live Dashboard** from `index.html` (or open `dashboard.html` directly).
   - Confirm the dashboard shows:
     - Daily consumption (kWh)
     - AI actions in last 24h
     - Self-correction score
     - DGM fitness and evolution notes
   - Upload an additional real-time CSV in dashboard to verify optimization suggestions appear.

---

## 3) CSV schema expected by the HTML app

Header (required):
```csv
timestamp,deviceId,powerKw,temperature,pricePerKwh,occupancy
```

- `timestamp`: ISO-8601 datetime, e.g., `2026-03-26T10:15:00Z`
- `deviceId`: string, e.g., `hvac_main`, `ev_charger_1`
- `powerKw`: number
- `temperature`: number (°C assumed)
- `pricePerKwh`: number
- `occupancy`: `0` or `1`

---

## 4) Prompt to synthesize custom test data

Use this prompt with any LLM to generate CSV test data:

```text
Generate synthetic home-energy telemetry as CSV with exactly these columns:
timestamp,deviceId,powerKw,temperature,pricePerKwh,occupancy

Requirements:
1) Create 240 rows (15-minute intervals over 60 hours).
2) Use deviceIds from this list: hvac_main,ev_charger_1,water_heater_1,solar_inverter_1,battery_1.
3) Keep numeric ranges realistic:
   - powerKw: 0.1 to 9.5
   - temperature: 16.0 to 30.0
   - pricePerKwh: 0.08 to 0.45
   - occupancy: mostly 1 from 06:00-23:00 local time, mostly 0 overnight.
4) Add realistic behavior patterns:
   - Higher pricePerKwh during 17:00-21:00.
   - EV charger mostly active 00:00-05:00.
   - HVAC power rises when temperature deviates from 22.
   - Solar inverter has higher power between 09:00-16:00 and near-zero at night.
5) Inject anomalies in ~3% of rows (price spikes, sudden load spikes, occupancy surprises).
6) Output valid CSV only, no markdown, no explanation.
```

---

## 5) Ready-to-use sample mini CSV

```csv
timestamp,deviceId,powerKw,temperature,pricePerKwh,occupancy
2026-03-26T06:00:00Z,hvac_main,3.4,19.2,0.12,1
2026-03-26T06:15:00Z,water_heater_1,1.9,19.3,0.12,1
2026-03-26T06:30:00Z,ev_charger_1,0.0,19.5,0.13,1
2026-03-26T12:00:00Z,solar_inverter_1,4.8,24.1,0.18,1
2026-03-26T18:30:00Z,ev_charger_1,0.2,23.0,0.41,1
2026-03-26T22:30:00Z,hvac_main,2.7,20.1,0.16,0
```

Save as `synthetic_energy.csv` and upload in the app.
