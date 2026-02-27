# SigmaPrompt API Contract Specification (v1)

## 1. Protocol Standards
- External client APIs: REST/JSON over HTTPS
- Service-to-service APIs: gRPC over mTLS
- Realtime push: WebSocket and server-sent events for dashboard streams
- Event contracts: versioned JSON schema messages on event bus

## 2. Global API Conventions
- Base URL: `/api/v1`
- Authentication: OAuth2/JWT bearer token
- Tenant scoping: `X-Tenant-ID` required for enterprise requests
- Robot scoping: `robot_id` required in all telemetry/control endpoints
- Traceability: `X-Request-ID` propagated through all services

## 3. Authentication Service

### POST `/auth/token`
Issue access token.

**Request**
```json
{
  "client_id": "fleet-console",
  "client_secret": "***",
  "grant_type": "client_credentials",
  "scope": "telemetry:read command:write"
}
```

**Response 200**
```json
{
  "access_token": "jwt",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

## 4. Telemetry Ingestion Service

### POST `/telemetry/events`
Ingest batched robot telemetry.

**Request**
```json
{
  "robot_id": "RB-1022",
  "timestamp": "2026-01-11T05:12:00Z",
  "joint_state": [{"name": "knee_l", "position": 0.42, "torque_nm": 6.1}],
  "power": {"battery_soc": 0.77, "draw_w": 312.0},
  "thermal": {"motor_max_c": 64.2},
  "imu": {"pitch": 0.03, "roll": -0.01, "yaw": 1.22},
  "flags": ["nominal"]
}
```

**Response 202**
```json
{
  "accepted": true,
  "event_id": "evt_01J...",
  "ingested_at": "2026-01-11T05:12:00.124Z"
}
```

## 5. Real-Time Analyzer Service

### GET `/robots/{robot_id}/health`
Returns computed health and risk summary.

**Response 200**
```json
{
  "robot_id": "RB-1022",
  "health_score": 0.93,
  "anomaly_score": 0.04,
  "risk_level": "low",
  "updated_at": "2026-01-11T05:12:02Z"
}
```

## 6. Digital Twin Engine

### POST `/twins/{robot_id}/simulate`
Run an on-demand simulation mode.

**Request**
```json
{
  "mode": "joint_fatigue",
  "horizon_minutes": 180,
  "physics_backend": "mujoco",
  "parameters": {
    "payload_kg": 8.5,
    "ambient_c": 36.0
  }
}
```

**Response 200**
```json
{
  "simulation_id": "sim_7f2",
  "failure_probability": 0.28,
  "maintenance_window_hours": 72,
  "risk_heatmap_uri": "s3://sigma/twins/RB-1022/sim_7f2.png"
}
```

## 7. Swarm Coordination Service

### POST `/swarm/tasks/allocate`
Allocate tasks across active robot cluster.

**Request**
```json
{
  "swarm_id": "assembly-line-a",
  "tasks": [
    {"task_id": "pick-1", "priority": "high", "required_capabilities": ["lift", "vision"]}
  ],
  "constraints": {"max_latency_ms": 120, "safety_mode": "strict"}
}
```

**Response 200**
```json
{
  "allocation_id": "alloc_112",
  "leader_robot_id": "RB-2001",
  "assignments": [{"task_id": "pick-1", "robot_id": "RB-2009"}]
}
```

## 8. Command and Failsafe Endpoints

### POST `/robots/{robot_id}/commands`
Dispatch validated actuator-level command.

### POST `/robots/{robot_id}/failsafe/trigger`
Trigger failsafe mode (`thermal_shutdown`, `geofence_lock`, `mechanical_lock`).

### POST `/robots/{robot_id}/failsafe/recover`
Attempt controlled recovery from safe mode.

## 9. Alert Service

### GET `/alerts`
Query active and historical alerts (filter by severity, robot, window).

### POST `/alerts/ack`
Acknowledge alert with operator identity and notes.

## 10. Dashboard API

### GET `/dashboard/fleet/overview`
Returns fleet KPIs: active robots, anomaly count, energy usage, and SLA status.

### GET `/dashboard/stream`
WebSocket channel for live telemetry and incident updates.

## 11. Event Schemas (Bus)
Mandatory event types:
- `telemetry.ingested.v1`
- `anomaly.detected.v1`
- `failsafe.triggered.v1`
- `swarm.leader_elected.v1`
- `twin.simulation.completed.v1`

Each event includes:
- `event_id`
- `event_type`
- `occurred_at`
- `robot_id` or `swarm_id`
- `trace_id`
- versioned payload

## 12. Error Model
Standard error envelope:
```json
{
  "error": {
    "code": "INVALID_ARGUMENT",
    "message": "robot_id is required",
    "details": [{"field": "robot_id", "reason": "missing"}],
    "request_id": "req_..."
  }
}
```

## 13. SLO-aligned API Targets
- P95 read latency: <200 ms
- P95 command dispatch: <350 ms
- Critical failsafe API availability: 99.99%
- End-to-end decision window: <500 ms
