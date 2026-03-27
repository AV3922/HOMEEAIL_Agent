# Internal Technical Documentation: HyperAgent Home Energy System

## Purpose
This document explains the internals of the single-file HTML prototype and provides implementation guidance for converting it into a production-grade platform.

## Current Prototype Scope
- Front-end only runtime (`index.html`)
- In-memory state (no persistence)
- Simulated API router (`apiRouter`)
- Rule-based + weighted decision logic
- Feedback adaptation for policy weights
- DGM-inspired evolution loop (generation/fitness/mutations)

## Architecture (Prototype)
1. **UI Layer**
   - User workflows for onboarding, ingestion, control and inspection.
2. **Core Agent Layer**
   - Decision engine (`decideForRow`, `runDecisionCycle`)
   - Learning updater (`applyFeedback`)
   - Evolution module (`runEvolutionCycle`)
3. **API Facade Layer**
   - In-browser endpoint routing for testability:
     - `/api/v1/devices`
     - `/api/v1/telemetry`
     - `/api/v1/decisions`
     - `/api/v1/recommendations`
     - `/api/v1/policy`
     - `/api/v1/evolution`

## Data Contracts
### Telemetry row schema
```json
{
  "timestamp": "ISO-8601 string",
  "deviceId": "string",
  "powerKw": "number",
  "temperature": "number",
  "pricePerKwh": "number",
  "occupancy": "0|1"
}
```

### Decision row schema
```json
{
  "at": "ISO-8601 string",
  "target": "deviceId",
  "action": "maintain|defer_charging|preheat_home|reduce_load|opportunistic_run",
  "reason": "string",
  "mode": "automatic|recommendation"
}
```

## Decision Model Details
Weighted score:
```
score = pricePerKwh*costWeight + powerKw*carbonWeight - abs(22-temperature)*comfortWeight + autonomyWeight
```

Rule priority:
1. EV high-tariff deferral
2. HVAC comfort restoration if occupied and cold
3. High score => reduce load
4. Low score => opportunistic run
5. Else => maintain

## Learning / Adaptation Details
- Input: feedback score in [-1, +1]
- Update: small-step gradient-like adjustment using `alpha=0.05`
- Policy bounds:
  - `costWeight` ∈ [0.1, 0.7]
  - `comfortWeight` ∈ [0.1, 0.5]
  - `carbonWeight` ∈ [0.1, 0.4]
  - `autonomyWeight` ∈ [0.05, 0.3]

## DGM-inspired Evolution Module
- Tracks generations and computes fitness from policy + activity + noise.
- Keeps recent mutation notes (rolling window).
- If evolution enabled and fitness > threshold, autonomy can increase.
- Research reference: https://github.com/jennyzzt/dgm

## Productionization Plan
1. **API Service**
   - Implement real backend with FastAPI/Node/Go.
   - Add POST endpoints for telemetry ingest, control execution, and feedback.
2. **Data Layer**
   - Time-series DB for telemetry.
   - Relational DB for devices, decisions, recommendations, model metadata.
3. **Connector Layer**
   - Protocol adapters/workers for MQTT, Modbus, BACnet, REST webhooks.
4. **Model Layer**
   - Start with rule policy + learned weights.
   - Add model registry, feature store, rollout controls.
5. **Safety & Security**
   - OAuth2/JWT, RBAC, audit logs, signed command channel.
   - Action guardrails and fallback modes.

## Testing Guidance
- Unit-test parsing, scoring, adaptation, and evolution functions.
- Integration-test all endpoint contracts.
- Scenario-test tariff spikes, occupancy shifts, sensor outliers, and control failures.
- Add synthetic load tests for hourly decision throughput and latency.
