# Real-time WebSocket Chat Application

## Project Overview
A real-time multi-user chat application using FastAPI WebSockets, containerized with Docker, reverse proxied with Nginx, and deployed on AWS EC2 with automated CI/CD.

## Live Application
**Public IP**: `http://54.172.238.35`

## Architecture
┌──────────────────────────────────────────────────────────────┐
│                        User Browser                          │
│                    http://54.172.238.35                      │
└─────────────────────────────┬────────────────────────────────┘
                              │ HTTP / WebSocket
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                     AWS EC2 Instance                         │
│                      (54.172.238.35)                         │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                    Docker Host                         │  │
│  │                                                        │  │
│  │   ┌──────────────────┐       ┌───────────────────────┐ │  │
│  │   │     NGINX        │       │    Backend Service    │ │  │
│  │   │    Container     │       │   (FastAPI + WS)      │ │  │
│  │   │                  │       │                       │ │  │
│  │   │  Port: 80        │─────▶│  Port: 8000           │ │  │
│  │   │                  │       │                       │ │  │
│  │   │  Responsibilities│       │  Endpoints:           │ │  │
│  │   │  - Serve frontend│       │  - /health            │ │  │
│  │   │  - Reverse proxy │       │  - /ws (WebSocket)    │ │  │
│  │   │  - WS upgrade    │       │                       │ │  │
│  │   └──────────────────┘       └───────────────────────┘ │  │
│  │                                                        │  │
│  │        Docker Bridge Network: chat-network             │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘


WebSocket Flow:
1. Client → GET /ws (Upgrade: websocket)
2. NGINX → forwards to backend:8000/ws
3. Backend → accepts upgrade
4. Persistent bidirectional connection established

## Docker Setup

### Container Configuration
- **Backend Container**: Python 3.11 with FastAPI, runs on port 8000
- **Nginx Container**: Alpine Linux, serves frontend and proxies WebSocket

### Docker Networking
Custom bridge network `chat-network` enables:
- Container name resolution (backend:8000)
- Isolated communication between services
- No port exposure to host except through Nginx

### Auto-restart
Both containers have `restart: always` policy for production resilience.

## Nginx Reverse Proxy

### Configuration Highlights
```nginx
location /ws {
    proxy_pass http://backend:8000/ws;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
