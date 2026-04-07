# Real-time WebSocket Chat Application

## Project Overview
A real-time multi-user chat application using FastAPI WebSockets, containerized with Docker, reverse proxied with Nginx, and deployed on AWS EC2 with automated CI/CD.

## Live Application
**Public IP**: `http://54.172.238.35`

## Architecture
<img width="1440" height="1312" alt="image" src="https://github.com/user-attachments/assets/9a208f14-202f-4339-9c72-2bcab488a48e" />

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
