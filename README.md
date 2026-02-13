# Observability Platform Infrastructure

This repository contains the infrastructure and deployment
configurations for the AI Agent Observability Platform. It orchestrates
all required services using Docker Compose for development, testing, and
production environments.

The platform includes:

-   Flask backend API\
-   Next.js frontend dashboard\
-   PostgreSQL database\
-   Ollama (local LLM service)\
-   pgAdmin (optional database UI)\
-   Nginx reverse proxy (optional, recommended for production)

All services communicate through a dedicated Docker network:
observability-net.

------------------------------------------------------------------------

# Prerequisites

Before using this repository, ensure you have:

-   Docker 24.x or later\
-   Docker Compose 2.x or later\
-   Git\
-   Access to:
    -   observability_project-backend\
    -   observability_project-frontend\
-   Basic understanding of Docker networking and environment variables

Verify installations:

docker --version docker compose version

------------------------------------------------------------------------

# Repository Structure

observability_project-infra/ ├── docker-compose.yml \# Production
compose file ├── docker-compose.dev.yml \# Development compose file ├──
docker-compose.test.yml \# Testing compose file ├── .env.example \#
Environment variables template ├── nginx/ │ └── nginx.conf \# Reverse
proxy configuration └── README.md

------------------------------------------------------------------------

# Quick Start (Development)

This setup is intended for local development with hot reload enabled.

## 1. Clone Infrastructure Repository

git clone https://github.com/your-org/observability_project-infra.git cd
observability_project-infra

## 2. Copy Environment File

cp .env.example .env

Update values as needed.

## 3. Clone Backend and Frontend (as Siblings)

workspace/ ├── observability_project-infra/ ├──
observability_project-backend/ └── observability_project-frontend/

Example:

cd .. git clone
https://github.com/your-org/observability_project-backend.git git clone
https://github.com/your-org/observability_project-frontend.git

## 4. Start Development Stack

cd observability_project-infra docker compose -f docker-compose.dev.yml
up --build

## 5. Access Services

Frontend: http://localhost:3000\
Backend: http://localhost:5000\
PostgreSQL: localhost:5432\
Ollama: http://localhost:11434\
pgAdmin: http://localhost:5050

Development mode uses volume mounts for hot reload.

------------------------------------------------------------------------

# Production Deployment

Production deployment uses prebuilt images (preferably from GHCR).

## 1. Configure Environment Variables Securely

-   Do not commit .env\
-   Use strong passwords\
-   Set secure secret keys\
-   Configure production API URLs

## 2. Start Production Stack

docker compose up -d

This uses docker-compose.yml.

## 3. Nginx Reverse Proxy (Recommended)

Nginx provides unified access:

-   / → Frontend\
-   /api → Backend

Edit nginx/nginx.conf and rebuild if needed:

docker compose up -d --build nginx

## 4. SSL / HTTPS

Recommended approaches:

-   Use Let's Encrypt with Certbot\
-   Use a cloud load balancer with SSL termination\
-   Mount certificate volumes inside Nginx container

------------------------------------------------------------------------

# Environment Variables

  Variable                   Description
  -------------------------- ------------------------------
  POSTGRES_DB                Database name
  POSTGRES_USER              Database username
  POSTGRES_PASSWORD          Database password
  DATABASE_URL               PostgreSQL connection string
  BACKEND_SECRET_KEY         Flask secret key
  NEXT_PUBLIC_API_URL        Public backend API URL
  OLLAMA_MODEL               Default LLM model name
  PGADMIN_DEFAULT_EMAIL      pgAdmin login email
  PGADMIN_DEFAULT_PASSWORD   pgAdmin password

Example:

POSTGRES_DB=observability POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
DATABASE_URL=postgresql://postgres:postgres@postgres:5432/observability
BACKEND_SECRET_KEY=change_this NEXT_PUBLIC_API_URL=http://localhost:5000
OLLAMA_MODEL=llama3

------------------------------------------------------------------------

# Service Details

## Backend (Flask API)

-   Port: 5000\
-   Depends on: PostgreSQL, Ollama\
-   Network: observability-net\
-   Health check endpoint: /health\
-   Development: volume mounted\
-   Production: GHCR image

## Frontend (Next.js)

-   Port: 3000\
-   Depends on: Backend\
-   Network: observability-net\
-   Development: volume mounted\
-   Production: optimized static build

## PostgreSQL

-   Port: 5432\
-   Persistent volume for data\
-   Network alias: postgres\
-   Used by: Backend

## Ollama

-   Port: 11434\
-   Used by: Backend\
-   Stores local LLM models

Pull model example:

docker exec -it ollama ollama pull llama3

## pgAdmin (Optional)

-   Port: 5050\
-   Connect to host: postgres

## Nginx (Optional)

-   Port: 80 / 443\
-   Reverse proxy for frontend and backend

------------------------------------------------------------------------

# GitHub Container Registry (GHCR) Integration

## Authenticate

echo \$GHCR_TOKEN \| docker login ghcr.io -u USERNAME --password-stdin

## Pull Images

docker pull ghcr.io/your-org/observability-backend:latest docker pull
ghcr.io/your-org/observability-frontend:latest

## Tagging Strategy

-   latest → stable production\
-   vX.Y.Z → versioned release\
-   dev → development builds

------------------------------------------------------------------------

# Updating and Maintenance

## Pull Latest Images

docker compose pull docker compose up -d

## Restart Services

docker compose restart

## View Logs

docker compose logs -f docker compose logs -f backend

## Backup Database

docker exec -t postgres pg_dump -U postgres observability \> backup.sql

## Restore Database

docker exec -i postgres psql -U postgres observability \< backup.sql

------------------------------------------------------------------------

# Troubleshooting

## Port Conflicts

lsof -i :5000

Change exposed port in compose file if needed.

## Database Connection Errors

-   Ensure DATABASE_URL uses hostname postgres\
-   Verify PostgreSQL container is running

## Ollama Model Issues

docker exec -it ollama ollama pull llama3

Ensure sufficient disk space.

## CORS Problems

-   Confirm NEXT_PUBLIC_API_URL is correct\
-   Verify backend CORS configuration

## Permission Issues

sudo chown -R $USER:$USER .

------------------------------------------------------------------------

# Architecture Diagram

                   +------------------+
                   |      Nginx       |
                   |  (Reverse Proxy) |
                   +--------+---------+
                            |
          -----------------------------------------
          |                                       |

+----------------+ +----------------+ \| Frontend \|
\<----------------\> \| Backend \| \| Next.js \| \| Flask API \|
+--------+-------+ +--------+-------+ \| \| \|
+-------------+-------------+ \| \| \| +----------------+
+------------------+ +----------------+ \| PostgreSQL \| \| Ollama \| \|
pgAdmin \| \| Database \| \| LLM Service \| \| DB Admin UI \|
+----------------+ +------------------+ +----------------+

All services connected via Docker network: observability-net

------------------------------------------------------------------------

# Related Repositories

Backend: https://github.com/your-org/observability_project-backend\
Frontend: https://github.com/your-org/observability_project-frontend

------------------------------------------------------------------------

# Notes

-   Backend and frontend must be cloned as siblings to this repository.\
-   Development uses bind mounts for hot reload.\
-   Production uses prebuilt container images (preferably GHCR).\
-   Nginx is optional but recommended for production.\
-   All services communicate internally using observability-net.
