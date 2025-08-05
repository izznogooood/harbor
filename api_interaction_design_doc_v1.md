# API and Interaction Design Document

[View on Eraser![](https://app.eraser.io/workspace/ui0nQr4DqIPHvjl0Z5z0/preview?elements=iYNG785yG_M5jNzF7K8URQ&type=embed)](https://app.eraser.io/workspace/ui0nQr4DqIPHvjl0Z5z0?elements=iYNG785yG_M5jNzF7K8URQ)

## Overview
This document describes the API interactions, authentication mechanisms, and the connectivity between frontend, backend, database (Supabase), and Docker daemon for the Self-Hosted Docker Management Dashboard (SDMD).

A blueprint ensuring cohesive development and clear contract between the backend, frontend, and database interactions for the SDMD.

> The API only manages “services.” Each service may contain one or more containers, but there is no separate concept of “container” or “stack” in the API. Single-container setups are defined as services with one container.

## System Architecture

### Components
- **Frontend**: React (Next.js) application with TailwindCSS and Shadcn/ui components.
- **Backend**: Python FastAPI application.
- **Authentication/Database**: Self-hosted Supabase (PostgreSQL + JWT Auth).
- **Scheduler**: Celery.
- **Docker API**: Docker daemon accessed via docker-py.

### Connectivity
- **Frontend ↔ Backend**: HTTP(S) REST API calls secured via JWT.
- **Backend ↔ PostgreSQL**: Direct database interactions using SQLAlchemy ORM.
- **Backend ↔ Supabase**: JWT authentication verification only.
- **Backend ↔ Docker Daemon**: Direct Docker API calls through a local Unix socket, secured internally.
- **Backend ↔ Celery**: Task queuing and scheduling via Redis/RabbitMQ (internally).

## Authentication

- JWT Tokens are issued by Supabase upon successful user login.
- Frontend attaches JWT tokens to Authorization header for API calls.
- Backend verifies JWT tokens issued by Supabase for all API requests. All authenticated users have full administrative access.

## API Specification

### 1. Authentication
- `POST /auth/login`
  - Description: User login.
  - Request: Username/password.
  - Response: JWT token.

- `POST /auth/logout`
  - Description: User logout, invalidate session.

### 2. Docker service Management
- `GET /api/services`
  - Description: List running services and their status.
  - Response: JSON array with service details.

- `POST /api/services`
  - Description: Create a new service.
  - Request body: Service definition (see below).
  - Response: Created service details.

- `PUT /api/services/{id}`
  - Description: Update an existing service.
  - Request body: Updated service definition.
  - Response: Updated service details.

- `DELETE /api/services/{id}`
  - Description: Delete a service.
  - Parameters: service ID
  - Response: Success/error status.

- `POST /api/services/{id}/restart`
  - Description: Restart a specific service.
  - Parameters: service ID
  - Response: Success/error status.

- `GET /api/services/{id}/logs?tail=100`
  - Description: Fetch the last N lines of logs for a specific service (default: 100).
  - Parameters: service ID, optional `tail` query param (default 100).
  - Response: Log output as plain text or JSON.

### 3. Docker Image Management
- `POST /api/images/pull`
  - Description: Pull specified Docker image.
  - Request body: JSON (image name/tag).
  - Response: Pull log/status.

### 4. Service Health
- `GET /api/health/status`
  - Description: Get a summary of all services’ running/stopped state.
  - Response: JSON array with each service’s ID, name, and status.
  - Example response:
    {
      "services": [
        { "id": "abc123", "name": "web", "status": "running" },
        { "id": "def456", "name": "db", "status": "stopped" }
      ]
    }

### 5. Schedule Management
- `GET /api/schedules`
  - Description: List all update schedules for all services.
  - Response: JSON array of schedules.

- `POST /api/schedules`
  - Description: Create a new update schedule for a service.
  - Request body: JSON (service ID, frequency [daily/weekly], time of day).
  - Response: Created schedule details.

- `PUT /api/schedules/{id}`
  - Description: Update an existing schedule.
  - Parameters: Schedule ID.
  - Request body: JSON (frequency, time of day).
  - Response: Updated schedule details.

- `DELETE /api/schedules/{id}`
  - Description: Delete a schedule.
  - Parameters: Schedule ID.
  - Response: Success/error status.

- Example schedule object:
  {
    "id": "sched123",
    "service_id": "abc123",
    "frequency": "weekly",
    "day_of_week": "Sunday",
    "time_of_day": "03:00"
  }

## Interaction Flow

### Frontend to Backend
- User logs into Frontend, receives JWT.
- User requests Dashboard view (`/api/services`). Frontend calls Backend with JWT.
- Backend authenticates via JWT verification against Supabase.
- Backend interacts with Docker daemon, retrieves data, and returns it to Frontend.

### Backend Scheduled Tasks
- Celery periodically invokes Backend tasks (e.g., pulling images, restarting services).
- Backend tasks communicate securely with Docker API and log outcomes to Supabase.

## Database Schema (Supabase)

### Tables (PostgreSQL):
- **Users**: User details (managed by Supabase Auth).
- **DockerServices**: service information, status logs.
- **DockerServiceConfigs**: Stores service definitions. Each service definition supports: `services`, `image`, `container_name`, `environment`, `volumes`, `ports`, `restart`, `depends_on`.
- **ScheduledTasks**: Schedule and status of automated tasks.

## Security

- JWT-based authentication for all API endpoints.
- All authenticated users have full administrative access. No role-based restrictions.
- Secure Docker communication via local Unix socket (not exposed externally).
- Frontend-backend API secured via HTTPS.

## Error Handling & Logging

- All API errors are returned as JSON objects with at least an `error` field and, optionally, a `details` field for more information.
- Example error response:
  {
    "error": "service not found",
    "details": "No service with ID abc123 exists"
  }
- All significant actions (login, Docker ops, scheduled tasks) logged to Supabase for audit.
- Log Retention: Logs will be retained for 90 days.
- Log Immutability: Logs are append-only; no modification or deletion via API.
- Log Access: All authenticated users (admins) can view logs.
- The frontend is expected to display user-friendly error messages based on the `error` field in API responses.
- Technical details from the `details` field should be logged for support/troubleshooting, not shown to end users unless required.
- For recoverable errors (e.g., network timeouts), the frontend should provide retry options or clear guidance to the user.
