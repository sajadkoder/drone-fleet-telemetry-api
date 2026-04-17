# Drone Fleet Telemetry API

A backend system that simulates a fleet of autonomous drones, streams their telemetry in real time over WebSocket, and exposes REST endpoints for fleet management, drone commands, and alerting. The frontend is a React dashboard that connects to the backend and displays live drone positions, battery levels, and system alerts.

## Tech Stack

- **Backend**: FastAPI, Python 3.11, Pydantic v2, Uvicorn
- **Database/Cache**: Redis (pub/sub and telemetry storage)
- **Auth**: JWT (python-jose, passlib/bcrypt)
- **Frontend**: React 18, Vite, TailwindCSS, React-Leaflet, Recharts
- **DevOps**: Docker Compose

## Features

- Real-time telemetry streaming (GPS, battery, altitude, speed, signal strength) via WebSocket
- Live snapshot of all drones sent on WebSocket connection
- REST API for fleet listing, drone details, and fleet summary statistics
- Drone commands: land, take_off, return_to_base, emergency_stop, pause, resume
- Telemetry history retrieval (last N frames per drone)
- Anomaly detection with debounced alerts (low battery, signal loss, mission abort)
- Optional OpenAI integration to enhance alert messages
- JWT authentication for all API and WebSocket endpoints
- System health checks and metrics endpoint
- Redis graceful fallback when Redis is unavailable
- Telemetry simulator with 5 configurable drones (deterministic UUIDs stable across restarts)

## Prerequisites

- Python 3.11 or higher
- Node.js 18 or higher
- Redis 7 or higher (optional — system works without it)

## Installation

```bash
# Backend
cd backend
python -m venv venv
venv/Scripts/activate    # Windows
source venv/bin/activate # Linux/Mac
pip install -r requirements.txt

# Frontend
cd frontend
npm install
```

## Usage

Start the backend (default port 8000; use PORT=9001 on Windows):

```bash
cd backend
PORT=9001 python -m uvicorn main:app --port 9001
```

Start the frontend (in a separate terminal):

```bash
cd frontend
npm run dev
```

Login at http://localhost:3000 with `admin` / `admin123`.

### REST Examples

Get a JWT token:

```bash
curl -X POST http://localhost:9001/auth/login  -H 'Content-Type: application/json'  -d '{  }'
```

List all drones:

```bash
curl http://localhost:9001/fleet  -H 'Authorization: Bearer <token>'
```

Get fleet summary statistics:

```bash
curl http://localhost:9001/fleet/summary  -H 'Authorization: Bearer <token>'
```

Send a command to a drone. Get the drone ID from the `/fleet` response first:

```bash
curl -X POST http://localhost:9001/fleet/<drone_id>/command  -H 'Authorization: Bearer <token>'  -H 'Content-Type: application/json'  -d '{  }'
```

Get telemetry history for a drone:

```bash
curl 'http://localhost:9001/fleet/<drone_id>/telemetry/history?limit=60'  -H 'Authorization: Bearer <token>'
```

List recent alerts:

```bash
curl http://localhost:9001/alerts  -H 'Authorization: Bearer <token>'
```

### WebSocket Example

Connect to `ws://localhost:9001/ws/telemetry?token=<token>`.

On connect, the server sends a `snapshot` message with all drones and their latest telemetry.

Every second, the server sends a `telemetry` message per drone.

To subscribe to specific drones only, send:

```json
{  }
```

To unsubscribe:

```json
{  }
```

## Project Structure

```
backend/
  main.py              API entry point, lifespan events, health endpoints, metrics
  config.py            Settings via Pydantic (from env vars, with defaults)
  redis_client.py      Async Redis singleton with retry logic and graceful fallback
  requirements.txt     Python dependencies
  auth/
    jwt_handler.py     JWT create/decode, password hashing with bcrypt
    routes.py          /auth/login, /auth/register, /auth/me, /auth/refresh
  fleet/
    models.py          Pydantic models: Drone, TelemetryFrame, Alert, Mission, DroneCommand
    routes.py          Fleet, mission, alert, command REST endpoints
    service.py         Fleet logic, telemetry cache, history storage (in-memory)
  telemetry/
    simulator.py       DroneSimulator per-drone class, TelemetrySimulator fleet manager
    publisher.py       Redis pub/sub publisher with async queue
    websocket_gateway.py WebSocket endpoints, ConnectionManager, broadcast functions
  anomaly/
    engine.py          Rule-based anomaly detection, OpenAI alert enhancement
frontend/
  src/
    App.jsx            Main app, auth state, telemetry display layout
    hooks/
      useTelemetry.js  WebSocket connection hook, drone state, fleet summary polling
    components/
      Login.jsx        Login form, calls /auth/login, stores JWT in localStorage
      DroneCard.jsx    Drone status card with battery bar, selectable for detail view
      FleetMap.jsx     Leaflet map showing drone markers color-coded by status
      AlertFeed.jsx    Scrolling alert feed with severity styling
      BatteryChart.jsx Battery history line chart fetched from /telemetry/history
```

## Environment Variables

| Variable | Description | Default |
|---|---|---|
| `PORT` | Backend listen port (use 9001 on Windows) | `8000` |
| `REDIS_URL` | Redis connection URL | `redis://localhost:6379` |
| `JWT_SECRET_KEY` | Secret for signing JWT tokens. Must change in production. | `your-secret-key-change-in-production` |
| `JWT_ACCESS_TOKEN_EXPIRE_MINUTES` | Token expiry in minutes | `1440` (24 hours) |
| `OPENAI_API_KEY` | OpenAI API key for alert enhancement (optional) | `None` |
| `OPENAI_MODEL` | OpenAI model for alert summarization | `gpt-4o-mini` |
| `SIMULATOR_ENABLED` | Start drone simulator on boot | `true` |
| `SIMULATOR_DRONES_COUNT` | Number of simulated drones | `5` |
| `SIMULATOR_TELEMETRY_INTERVAL` | Telemetry frame interval in seconds | `1.0` |

## Contributing

Open to contributions. Open an issue first to discuss changes.

## License

MIT