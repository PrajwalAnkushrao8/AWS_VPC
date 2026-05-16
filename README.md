# Production-Grade AWS 2-Tier Deployment with Docker, Nginx, ALB & Cloudflare

A production-inspired full-stack deployment project showcasing AWS networking, Dockerized application deployment, reverse proxying with Nginx, internal container networking, Application Load Balancer integration, and Cloudflare DNS configuration.

---

# Tech Stack

## Frontend
- React
- Nginx
- Docker

## Backend
- Node.js
- Express.js
- Docker

## Database
- MongoDB Atlas

## Cloud & Infrastructure
- AWS EC2
- AWS VPC
- Public & Private Subnets
- Internet Gateway
- NAT Gateway
- Route Tables
- Application Load Balancer (ALB)
- Cloudflare DNS

---

# AWS Architecture

```text
                    Internet
                        │
                        ▼
                  Cloudflare DNS
                        │
                        ▼
              AWS Application Load Balancer
                        │
                        ▼
                  EC2 Ubuntu Instance
                        │
         ┌──────────────┴──────────────┐
         │                             │
         ▼                             ▼
Frontend Container               Backend Container
(Nginx + React)                  (Node.js API)
         │                             │
         └──────────────┬──────────────┘
                        │
                Internal Docker Network
                        │
                        ▼
                  MongoDB Atlas
```

---

# VPC Design

## VPC CIDR

```text
10.0.0.0/16
```

## Public Subnets

| Subnet       | CIDR        |
|--------------|-------------|
| public-sub1A | 10.0.1.0/24 |
| public-sub1B | 10.0.2.0/24 |

## Private Subnets

| Subnet        | CIDR        |
|---------------|-------------|
| private-sub1A | 10.0.3.0/24 |
| private-sub1B | 10.0.4.0/24 |

## Networking Components

- Internet Gateway for public internet access
- NAT Gateway for outbound internet access from private subnets
- Separate route tables for public and private traffic
- Multi-AZ subnet architecture

---

# Docker Deployment

## Create Docker Network

```bash
docker network create app-network
```

## Build Backend Image

```bash
cd backend

docker build -t backend:v1 .
```

## Run Backend Container

```bash
docker run -d \
  --name backend \
  --network app-network \
  --env-file .env \
  backend:v1
```

## Build Frontend Image

```bash
cd ../frontend

docker build -t frontend:v1 .
```

## Run Frontend Container

```bash
docker run -d \
  --name frontend \
  --network app-network \
  -p 80:80 \
  frontend:v1
```

---

# Internal Container Communication

A user-defined Docker bridge network was used for internal communication between frontend and backend containers.

## Benefits

- Internal DNS resolution
- Container isolation
- Secure backend communication
- Backend not publicly exposed

---

# Nginx Reverse Proxy

Nginx was used to:

- Serve React static files
- Reverse proxy API requests
- Internally communicate with the backend container

## Nginx Configuration

```nginx
server {
    listen 80;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    location /api/ {

        resolver 127.0.0.11 valid=30s;

        set $backend http://backend:3001;

        proxy_pass $backend;

        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;

        proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;

        proxy_cache_bypass $http_upgrade;
    }
}
```

---

# Application Load Balancer (ALB)

The ALB was configured to:

- Expose the frontend application publicly
- Forward traffic to the EC2 instance
- Integrate with Cloudflare DNS

Only the frontend container was exposed publicly.

The backend container remained internal inside the Docker network.

---

# Cloudflare Integration

Cloudflare was used for:

- DNS management
- Domain mapping
- SSL support
- Traffic proxying

## Traffic Flow

```text
User
  │
  ▼
Cloudflare
  │
  ▼
AWS ALB
  │
  ▼
EC2 Instance
```

---

# Troubleshooting

## Nginx Upstream Error

```text
host not found in upstream "backend"
```

## Fix

```nginx
resolver 127.0.0.11 valid=30s;
```

## Verify Docker DNS

```bash
docker run --rm --network app-network alpine cat /etc/resolv.conf
```

### Expected Output

```text
nameserver 127.0.0.11
```

---

# Live Application

```text
https://rawi.in

```

---

# Key Learnings

- AWS VPC networking
- Public and private subnet design
- NAT Gateway routing
- Docker container networking
- Reverse proxy architecture
- Internal service communication
- ALB integration
- Cloudflare DNS configuration
- Production-inspired infrastructure design

---

# Author

**Prajwal Ankushrao**
