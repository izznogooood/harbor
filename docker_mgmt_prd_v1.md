# Product Requirements Document (PRD)

# Notes

- How do you add mounts / variables etc
- Standard folders / variables
- file permissions

## Product Name
Self-Hosted Docker Management Dashboard (SDMD)

## Objective
Develop a secure, maintainable, and scalable self-hosted application for managing Docker services (e.g. like the ones initiated by docker compose files) with user-friendly dashboard controls, built around direct Docker API integration.

## Target Users
- Self-hosting enthusiasts
- System administrators
- Small/medium business IT teams

## High-Level Requirements

### Functional Requirements

#### Docker Management
- Display running Docker services with current status.
- Display services health as ‘running’ or ‘stopped’ based on Docker status.
- Allow manual pulling/updating Docker images and restarting services.
- Support scheduled image updates with automatic service restarts.
- Provide detailed logging of Docker operations and service health checks.
- Allow users to fetch and view the last 100 lines of logs for any running or stopped service via the dashboard.
- Users can create, edit, and delete services from the dashboard. Service definitions are stored in the database and can be managed entirely via the UI/API.

#### User Interface
- Simple, responsive dashboard displaying:
  - Active Docker services.
  - service health/status indicators.
  - Controls for manual actions (pull, restart).
- Authentication required for all user interactions (Supabase self-hosted).

#### User Authentication & Authorization
- JWT-based authentication using self-hosted Supabase.
- All authenticated users have full administrative access. No role-based restrictions.

#### Scheduling
- Users can configure daily or weekly update schedules for each service/service, including time of day, via the dashboard.
- Scheduled tasks for:
  - Periodic Docker image pulls based on user-defined schedules.
  - Automatic service restarts post-update.

### Non-Functional Requirements

#### Security
- Secure Docker API access via local Unix socket (no direct exposure).
- Least privilege principles for Docker service runtime.
- Transport layer secured via TLS for web UI/API.

#### Error Propagation & User Experience

- The frontend must display clear, user-friendly error messages for all backend/API errors.
- Technical error details (from the `details` field) should be logged for troubleshooting but not shown to end users unless necessary.
- For transient errors (e.g., network issues), the frontend should offer retry options or guidance.

#### Logging & Audit
- All significant actions (login, Docker operations, scheduled tasks) are logged for audit purposes.
- Logs will be retained for a minimum of 90 days.
- Logs are append-only and cannot be modified or deleted via the UI/API to ensure audit integrity.
- All authenticated users (administrators) will have access to view audit logs.

#### Maintainability
- Clean, modular backend codebase following FastAPI best practices.
- Maintainable frontend built with React/Next.js using components from TailwindCSS and Shadcn/ui.
- Clear logging and error-handling mechanisms.

#### Scalability
- Stateless backend architecture to allow easy scaling.

#### Service Configuration
> A “service” may represent a single container or a group of containers (compose-like), but is always managed as a single unit in the dashboard. Single-container setups are defined as services with one container.

- The database stores all service configuration normally found in a docker-compose.yml file, including: `services`, `image`, `container_name`, `environment`, `volumes`, `ports`, `restart`, and `depends_on`. This enables centralized management, editing, and redeployment of Docker services from the dashboard.
- Each service definition supports these fields. The format is inspired by docker-compose, but all services are managed as first-class objects in the dashboard.

## Architecture & Technology Stack

### Backend
- **Framework:** Python FastAPI
- **Docker Integration:** docker-py (Direct Docker API)
- **Task Scheduling:** Celery
- **Database:** Direct connection to self-hosted PostgreSQL using SQLAlchemy ORM
- **Authentication:** Supabase (JWT Auth)

### Frontend
- **Framework:** React (Next.js)
- **UI Library:** TailwindCSS, Shadcn/ui
- **Charts/Visualizations:** Recharts or Chart.js

### Infrastructure
- **Deployment:** Fully self-hosted
- **Docker Containerization:** Backend, Frontend, Supabase
- **Networking:** Internal secured access only, no external exposure required.

## Tools for Development
- **Backend Prototyping:** Cursor
- **Frontend Prototyping:** Tempo Labs or Lovable

## Success Criteria
- Secure, reliable Docker service management via intuitive UI.
- Robust user authentication and authorization system.
- Easy maintainability and clear documentation.
- All components self-hosted, ensuring complete data ownership and no cloud dependencies.

## Reference Open-Source Projects
- Portainer CE (Docker API security practices)
- Yacht (Python + Docker API integration)
- Dockge (React frontend patterns)
- CasaOS (overall architecture inspiration)

## Milestones
1. Prototype backend API and Docker integration (FastAPI).
2. Integrate user authentication (Supabase JWT).
3. Develop frontend dashboard (Next.js).
4. Implement scheduled tasks (Celery).
5. Security audit and Docker integration hardening.
6. Final testing, documentation, and deployment.

