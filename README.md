# Drone Fleet Telemetry API

A production-grade real-time drone fleet monitoring system with FastAPI, WebSockets, Redis, and React.

## Features

- **Real-time Telemetry**: Live GPS, battery, altitude, speed via WebSockets
- **Fleet Management**: REST API for fleet overview, drone details, missions
- **Drone Commands**: Send land, take_off, return_to_base, emergency_stop
- **Telemetry History**: Store and retrieve historical data
- **Anomaly Detection**: Alerts for signal loss, low battery
- **Health Monitoring**: System health checks and metrics
- **JWT Authentication**: Secure API access
- **Redis**: Pub/Sub for message broadcasting
- **Docker**: Full containerized deployment

## Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   React     │────▶│   FastAPI  │────▶│   Redis    │
│  (Port3000) │     │(Port 9001) │     │(Port 6379) │
└──────────────┘     └─────┬──────┘     └──────────────┘
                            │
                    ┌────────┴────────┐
                    │               │
              ┌─────▼─────┐   ┌────▼────┐
              │Simulator │   │Anomaly  │
              └─────────┘   └────────┘
```

## Tech Stack

- **Backend**: FastAPI, Python, Pydantic, Redis, JWT
- **Frontend**: React 18, Vite, TailwindCSS, Leaflet
- **DevOps**: Docker Compose

## Quick Start

### Prerequisites

- Python 3.11+
- Node.js 18+
- Redis (optional - system works without it)

### Backend

```bash
cd backend
python -m venv venv
venv/Scripts/activate  # Windows
source venv/bin/activate  # Linux/Mac

pip install -r requirements.txt

# Run (port 9001 for Windows permissions)
python -m uvicorn main:app --port 9001
```

### Frontend

```bash
cd frontend
npm install

# Runs on port 3000
npm run dev
```

### Access

- **API**: http://localhost:9001
- **WebSocket**: ws://localhost:9001/ws/telemetry
- **Frontend**: http://localhost:3000
- **Docs**: http://localhost:9001/docs

### Login

```
Username: admin
Password: admin123
```

## API Endpoints

| Endpoint | Method | Description |
|----------|-------|------------|
| `/health` | GET | Health check |
| `/fleet` | GET | List all drones |
| `/fleet/summary` | GET | Fleet statistics |
| `/fleet/{id}/command` | POST | Send drone command |
| `/fleet/{id}/telemetry/history` | GET | Telemetry history |
| `/missions` | POST | Create mission |
| `/alerts` | GET | List alerts |
| `/ws/telemetry` | WS | WebSocket stream |
| `/auth/login` | POST | Get JWT token |

## WebSocket Messages

### Client → Server
```json
{"subscribe": ["drone_id"]}
{"unsubscribe": ["drone_id"]}
```

### Server → Client
```json
{"type": "snapshot", "data": {...}}
{"type": "telemetry", "data": {...}}
{"type": "alert", "data": {...}}
{"type": "status_change", "data": {...}}
```

## Docker Compose

```bash
docker-compose up --build
```

## Environment Variables

| Variable | Default | Description |
|---------|---------|-------------|
| `PORT` | 9001 | API port |
| `REDIS_URL` | redis://localhost:6379 | Redis URL |
| `JWT_SECRET_KEY` | your-secret-key | JWT secret |
| `SIMULATOR_ENABLED` | true | Enable simulator |

## Fleet Summary Response

```json
{
  "total_drones": 5,
  "active_drones": 2,
  "idle_drones": 2,
  "error_drones": 1,
  "average_battery_pct": 78,
  "recent_alerts_count": 3
}
```

## License

MIT