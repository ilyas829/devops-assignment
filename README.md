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
```
Issues Found and Fixed
Issue 1: Backend binding to localhost
Problem: --host 127.0.0.1 - container only accessible internally
Fix: Changed to --host 0.0.0.0
Impact: Nginx couldn't connect to backend

Issue 2: Missing Docker network
Problem: No explicit network defined
Fix: Added chat-network bridge network
Impact: Unreliable container communication

Issue 3: Nginx pointing to wrong host
Problem: proxy_pass http://localhost:8000/ws
Fix: Changed to http://backend:8000/ws
Impact: Nginx was looking in its own localhost

Issue 4: WebSocket headers commented out
Problem: Upgrade headers were commented
Fix: Uncommented proxy_set_header Upgrade and Connection
Impact: WebSocket upgrade failed, connection downgraded to HTTP

Issue 5: Frontend files not mounted
Problem: Volume mounting commented in docker-compose.yml
Fix: Uncommented - ./frontend:/usr/share/nginx/html:ro
Impact: 404 error when accessing root URL

Issue 6: Docker permission denied
Problem: permission denied while trying to connect to docker API
Fix: sudo usermod -aG docker ubuntu && newgrp docker
Impact: Couldn't run docker commands without sudo

Steps to Deploy
Manual Deployment
bash
# 1. Clone repository
git clone https://github.com/ilyas829/devops-assignment.git
cd devops-assignment

# 2. Fix Docker permissions
sudo usermod -aG docker ubuntu
newgrp docker

# 3. Deploy application
docker-compose up -d --build

# 4. Verify deployment
docker-compose ps
curl http://localhost/health

# 5. Access application
# Open browser: http://54.172.238.35
Automated Deployment (CI/CD)
Push code to GitHub main branch

GitHub Actions automatically deploys

Access updated application at http://54.172.238.35

Verify Deployment
bash
# Check container status
docker-compose ps

# View logs
docker-compose logs backend
docker-compose logs nginx

# Test endpoints
curl http://localhost/health          # Should return {"status":"healthy"}
curl -I http://localhost/              # Should return 200 OK
Test WebSocket Functionality
Open http://54.172.238.35 in multiple browser tabs

Send messages from different tabs

Verify real-time message delivery across all tabs

Check browser console for WebSocket connection status (should show "Connected ✅")

Container Auto-Restart
Both containers have restart: always policy:

Automatically restart if crashed

Restart on system reboot

No manual intervention needed

Health Check
Backend includes /health endpoint for monitoring:


Live URL: http://54.172.238.35


