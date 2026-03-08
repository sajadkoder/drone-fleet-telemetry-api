# Drone Fleet Telemetry API

A production-grade real-time drone fleet monitoring system built with FastAPI, WebSockets, Redis, and React.

## Features

- **Real-time Telemetry Streaming**: Live GPS, battery, altitude, speed, and mission status via WebSockets
- **Fleet Management**: REST API for fleet overview, drone details, and mission control
- **Drone Commands**: Send commands to drones (land, take_off, return_to_base, emergency_stop, pause, resume)
- **Telemetry History**: Store and retrieve historical telemetry data for analysis
- **Anomaly Detection**: Automated alerting for signal loss, low battery, geofence breaches
- **Health Monitoring**: Detailed system health checks and metrics
- **JWT Authentication**: Secure API access with token-based auth
- **Redis Pub/Sub**: Efficient message broadcasting for real-time updates
- **Docker Deployment**: Full containerized stack with docker-compose
- **Graceful Error Handling**: Retry logic with exponential backoff for Redis connections
- **System Metrics**: CPU, memory, and fleet statistics via `/metrics` endpoint
- **OpenAI Integration**: Optional AI-enhanced alert messages

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   React Front   │────▶│   FastAPI API   │────▶│     Redis       │
│   (Port 3000)   │     │   (Port 8000)   │     │   (Port 6379)   │
└─────────────────┘     └────────┬────────┘     └─────────────────┘
                                 │
                        ┌────────┴────────┐
                        │                 │
                 ┌──────▼──────┐   ┌──────▼──────┐
                 │  Simulator   │   │  Anomaly    │
                 │   Engine     │   │  Detection  │
                 └──────────────┘   └─────────────┘
```

## Tech Stack

- **Backend**: FastAPI, Python 3.11+, Pydantic, Redis, JWT
- **Frontend**: React 18, Vite, TailwindCSS, Recharts, React- **Database**: storage & pub/sub Redis (telemetry-Leaflet
)
- **DevOps**: Docker, Docker Compose

## Quick Start

### 3.11 Prerequisites

- Python Redis (or Docker+
- Node.js 18+
-)

### Option 1: Local Development

**Backend:**
 backend
pip install```bash
cd -r requirements.txt
set PYTHONPATH=.
uvapp --reload --
```

**Frontendicorn backend.main:port 8012:**
```bash
cd frontend
npm install
npm run dev
```

**Access:**
- API: http://localhost:8012
- Frontend: http://localhost:3000
- API Docs: http://localhost:8012/docs

### Option 2: Docker Compose

```bash
docker-compose up --build
```

Access:
- API: http://localhost:8000
- Frontend: http://localhost:3000

## API Endpoints

### Health & Monitoring
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Basic health check |
| GET | `/health/detailed` | Detailed component status |
| GET | `/metrics` | System metrics (CPU, memory, fleet stats) |

### Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/login` | Get JWT token |
| POST | `/auth/register` | Register new user |

### Fleet
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/fleet` | List all drones |
| GET | `/fleet/{drone_id}` | Get drone details |
| GET | `/fleet/summary` | Fleet statistics |
| POST | `/fleet/{drone_id}/command` | Send command to drone |
| GET | `/fleet/{drone_id}/telemetry/history` | Get telemetry history |

### Drone Commands
Send commands to control drones:
```json
POST /fleet/{drone_id}/command
{
  "command": "land" | "take_off" | "return_to_base" | "emergency_stop" | "pause" | "resume"
}
```

### Missions
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/missions` | List all missions |
| POST | `/missions` | Create new mission |
| GET | `/missions/{id}` | Get mission details |
| POST | `/missions/{id}/abort` | Abort mission |

### Alerts
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/alerts` | List all alerts |
| GET | `/alerts?severity=critical` | Filter by severity |
| GET | `/alerts/{drone_id}` | Get alerts for drone |

### WebSocket
| Endpoint | Description |
|----------|-------------|
| `/ws/telemetry` | Real-time telemetry stream |

## Default Credentials

```
Username: admin
Password: admin123
```

## Environment Variables

### Backend
| Variable | Default | Description |
|----------|---------|-------------|
| `REDIS_URL` | `redis://localhost:6379/0` | Redis connection |
| `JWT_SECRET_KEY` | `your-secret-key` | JWT signing key |
| `DEBUG` | `false` | Debug mode |
| `SIMULATOR_ENABLED` | `true` | Enable drone simulator |
| `OPENAI_API_KEY` | - | OpenAI API key for alert enhancement |

### Frontend
| Variable | Default | Description |
|----------|---------|-------------|
| `VITE_API_URL` | `http://localhost:8000` | API base URL |
| `VITE_WS_URL` | `ws://localhost:8000` | WebSocket URL |

## Project Structure

```
drone-fleet-telemetry-api/
├── backend/
│   ├── main.py              # FastAPI app entry point
│   ├── config.py            # Settings configuration
│   ├── redis_client.py      # Redis connection with retry logic
│   ├── auth/
│   │   ├── jwt_handler.py   # JWT token handling
│   │   └── routes.py        # Auth endpoints
│   ├── fleet/
│   │   ├── models.py        # Pydantic models
│   │   ├── routes.py        # Fleet endpoints
│   │   └── service.py       # Fleet logic
│   ├── telemetry/
│   │   ├── simulator.py     # Drone simulator
│   │   ├── publisher.py     # Redis publisher
│   │   └── websocket_gateway.py  # WebSocket handler
│   └── anomaly/
│       └── engine.py        # Anomaly detection
├── frontend/
│   ├── src/
│   │   ├── App.jsx          # Main React app
│   │   ├── components/      # UI components
│   │   └── hooks/          # Custom hooks
│   └── package.json
├── docker-compose.yml
└── README.md
```

## Simulated Drones

The system simulates 5 drones:
1. **Falcon-1** - High altitude patrol
2. **Eagle-2** - Urban surveillance
3. **Hawk-3** - Agricultural monitoring
4. **Phoenix-4** - Search and rescue
5. **Condor-5** - Long-range delivery

Each drone streams:
- GPS coordinates (lat/lng)
- Altitude (meters)
- Battery percentage
- Speed (m/s)
- Status (active, standby, returning, landed)
- Signal strength

## Anomaly Detection

The system monitors for:
- **Signal Loss**: Signal strength <25%
- **Low Battery Warning**: Battery <25%
- **Critical Battery**: Battery <10%
- **Mission Abort**: When mission is aborted
- **GPS Deviation**: Unusual movement patterns

## System Metrics

The `/metrics` endpoint returns:
- Uptime
- CPU and memory usage
- Fleet statistics (drones, alerts, missions)
- WebSocket connection count

## License

MIT
