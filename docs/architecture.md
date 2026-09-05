# Simplified Stiffened Panel Analysis — System Architecture

## Overview

A single-page React frontend handles drawing, validation, and visualization. A Python FastAPI backend provides numeric solve endpoints, project persistence, curve data, and PDF report generation. Heavy or long-running tasks (PDF rendering, large solves) run in background workers (Celery / Dramatiq). PostgreSQL stores projects and metadata; object storage (S3 or MinIO) stores generated PDFs and assets.

Components:
- Frontend (React SPA)
- API server (FastAPI + Uvicorn/Gunicorn)
- Worker queue (Celery or Dramatiq) + Redis
- Database (PostgreSQL)
- Object storage (S3 / MinIO)
- Reverse proxy / TLS (NGINX or cloud-managed)
- CI/CD (GitHub Actions)
- Monitoring & Logging (Prometheus/Grafana, Sentry)

## Component responsibilities

### Frontend (React)
- Interactive drawing canvas (SVG or react-konva) limited to: outer boundary rectangle + axis-aligned internal lines
- Client-side validation (geometry rules from plan.md)
- Create project object and submit to API
- Visualize solve results (shear flows, normal-load diagrams, K-curve plots)
- Request PDF generation and provide download UI
- Optional local save/export (JSON)

### API server (FastAPI)
- REST endpoints + OpenAPI docs
- Lightweight synchronous solves for quick feedback
- Request validation via pydantic models
- Enqueue heavy solves or PDF generation tasks to worker
- Serve curve datasets, material library
- Authentication & authorization (JWT / OAuth2)
- Input validation enforcing plan.md business rules

### Worker (Celery / Dramatiq)
- Execute numeric solver using numpy / scipy
- Compute A*q=b, residuals, detect singular/underdetermined systems
- Curve interpolation using pre-embedded datasets
- Run PDF generation (WeasyPrint or server-side Puppeteer)
- Persist results (DB + object store)

### Database (Postgres)
- Project metadata, user accounts, runs, curve metadata, material library
- Migrations via Alembic (or use SQLModel migration story)

### Object storage (S3 / MinIO)
- Store generated PDFs, diagrams, and large artifacts

### PDF / report generation
- Preferred: Render HTML templates with Jinja2 and convert to PDF using WeasyPrint or Puppeteer (headless Chromium) for vector SVG fidelity
- Alternative: LaTeX + pdflatex for highest typographic quality (more tooling and build complexity)

### Monitoring & Logging
- Centralized logs (structured, rotate)
- Error tracking with Sentry
- Metrics exported for Prometheus (request timings, queue lengths, residual norms, task success/failure)

### Security
- TLS everywhere (Let’s Encrypt or cloud managed)
- JWT or OAuth2 with refresh tokens; RBAC for user/project access
- Input sanitization to prevent injection (report HTML generation must escape untrusted input)
- Object storage ACLs and signed URLs for downloads

## High-level data flow (Solve request)

1. User draws model and supplies bay dimensions, materials, supports, loads.
2. Frontend runs initial validations; calls POST /api/projects (save) or POST /api/solve.
3. API validates payload server-side and either:
   - Synchronously computes small solve and returns results; OR
   - Creates a Run record and enqueues solve task to worker, returning run_id and status endpoint (/api/runs/{id}).
4. Worker picks job, assembles matrix A and vector b, solves for q using numpy/scipy (sparse solvers if many unknowns).
5. Compute tau_i = q_i / t and tau_cr per K-curve interpolation; compute utilization, RF.
6. Compute residual r = A*q - b and normalized epsilon_r; if epsilon_r > tolerance mark run failed and include diagnostics.
7. Persist results to DB, render diagrams (SVG), and generate PDF; store PDF in object store and update run status.
8. Frontend polls /api/runs/{id} or receives websocket/notifications when complete; user downloads report.

## API surface (summary)

- Auth
  - POST /api/auth/login — returns JWT
  - POST /api/auth/refresh
- Projects & runs
  - POST /api/projects — create project (drawing + properties)
  - GET /api/projects/{id} — get saved project
  - PUT /api/projects/{id} — update
  - DELETE /api/projects/{id}
  - POST /api/projects/{id}/solve — enqueue or run solve synchronously (query param ?sync=true)
  - POST /api/solve — ad-hoc solve without saving project
  - GET /api/runs/{run_id} — get status/results (includes links to PDF, SVGs)
  - GET /api/runs/{run_id}/pdf — signed download
- Reference data
  - GET /api/curves — list embedded K-curves + metadata
  - GET /api/materials — material library
- Admin
  - GET /api/health — basic health
  - GET /api/metrics — Prometheus metrics (if exposed)
- Webhooks / Notifications (optional)
  - POST /api/webhook/solve-complete — notify external services

All endpoints validate business rules from plan.md and return structured errors with codes and human-friendly messages.

## Data model (conceptual)

- User: id, email, hashed_password, role
- Project: id, owner_id, title, created_at, drawing (JSON describing lines/vertices), bays (list with width/height, selected curve id), material_id, thickness, supports, loads, validation_warnings
- Curve: curve_id, display_name, equation_convention, valid_range (min,max), points [(r,K)], verification_note
- Material: id, name, E, nu, metadata
- Run: id, project_id (nullable for ad-hoc), submitted_by, started_at, finished_at, status (queued/running/success/failure), residual, errors, result_blob (JSON), pdf_path
- Result (stored in DB or blob): q values per bay, tau_i, tau_cr_i, U_i, RF_i, normal-load diagrams data, SVG diagram assets

Notes:
- Keep numeric results in JSON blobs for replay and regression testing.
- Store large SVG or PDF as blob in object store and store signed URL in Run.

## Solver architecture & implementation notes

- Use numpy for dense small systems, scipy.sparse for larger problems (construct A as sparse CSR)
- Use robust linear-solver patterns and fallbacks:
  - Try sparse direct solver (scipy.sparse.linalg.spsolve)
  - On failure or singular, attempt least-squares or regularized solve (with small damping) and report diagnostic
- Always compute residual epsilon_r = ||r||2 / max(||b||2, 1) and enforce a validated tolerance
- Detect singular/inconsistent systems early and return clear error messages referencing plan.md rules
- Keep numeric core decoupled from web framework for unit-testability (pure Python module with deterministic outputs)

## Validation & business rules enforcement

- Implement both client-side and server-side validation for:
  - Geometry constraints (connected, orthogonal lines only, closed rectangular bays)
  - Load/support exclusivity per vertex
  - Units and positive numeric checks (t > 0, E > 0, valid nu)
  - Curve selection and a/b range (no extrapolation)
- Return error codes for each validation failure (ex: 422 with structured field errors)

## PDF & diagram generation

- Generate diagrams as SVG (from the calculation module) and embed them into an HTML Jinja template
- Use WeasyPrint or Puppeteer to render HTML->PDF with embedded SVG (vector-quality)
- Include intermediate steps in PDF when detailed mode enabled (equations, interpolation points)
- Provide downloadable vector SVGs for users who want to insert figures into other reports

## Persistence & migrations

- Use SQLAlchemy (or SQLModel) + Alembic for schema migrations
- Keep curve dataset in versioned YAML/JSON file in repo; load into DB on application bootstrap or migrations

## Background tasks & scaling

- Use Celery with Redis as broker (or Dramatiq). Workers scale horizontally.
- For short solves, allow synchronous API solve to return immediate response (limit to safe problem sizes).
- Use queue for heavy PDF generation or long-running tasks; track retries and back-off.

## Dev & deployment

- Containerize components with Docker
  - Services: web, worker, redis, postgres, minio (optional), nginx (optional)
- Docker Compose for local dev
- CI: GitHub Actions to run lint, unit tests, type checks (mypy), build images, and run integration tests
- CD: push images to registry and deploy to cloud (Cloud Run, ECS/Fargate, or Kubernetes). Use managed DB and object storage for production.
- Secrets management via environment variables / vault

## Testing strategy

- Unit tests:
  - Geometry detection & planar graph tests (create canonical cases)
  - Matrix assembly tests vs hand-calculated examples
  - Curve interpolation tests (edge values, interpolation points, no-extrapolation)
- Integration tests:
  - End-to-end solve from POST /api/solve to result comparison
  - PDF generation sanity tests
- Regression tests:
  - A suite of small reference models with stored results; run in CI and flag numeric drift
- Property tests:
  - Random small layouts within allowed geometry, ensure solver stability and residuals under threshold

## Observability & reliability

- Metrics: request latencies, queue lengths, solve durations, residual distributions, error rates
- Alerts: failed tasks, high residuals, long solve durations
- Logs: structured logs with correlation IDs for runs

## Prioritized MVP roadmap (short milestones)

1. Core numeric engine + unit tests
   - Implement geometry detection (axis-aligned rectangles)
   - Build A*q=b assembly and solver using numpy/scipy
   - Implement K-curve interpolation and tau/tau_cr, utilization calculations
   - Unit tests based on hand calculations
2. Minimal FastAPI endpoints + frontend prototypes
   - POST /api/solve synchronous endpoint returning results
   - Basic React canvas for drawing, minimal visualization
3. Background tasks and PDF generation
   - Hook up worker queue and asynchronous run flow
   - Implement HTML->PDF render and storage
4. Persistence, auth, and project UI
   - Add Postgres persistence, user auth flows, project save/load
5. Production hardening
   - CI/CD, monitoring, input validation, security review

## Technology choices (concise)

- Web framework: FastAPI
- ASGI server: Uvicorn + Gunicorn (uvicorn workers) or hypercorn
- Numerical: numpy, scipy, scipy.sparse
- Background tasks: Celery + Redis (or Dramatiq)
- DB: PostgreSQL (SQLAlchemy / SQLModel + Alembic)
- PDF: WeasyPrint or server-side Puppeteer; templates via Jinja2
- Frontend: React, react-konva or plain SVG; Plotly.js or D3 for plots
- Containerization: Docker Compose for dev, Kubernetes / Cloud Run for prod
- CI: GitHub Actions
- Storage: S3-compatible (AWS S3 or MinIO for local dev)
- Auth: OAuth2 with JWT tokens (FastAPI security utilities)

## Security & compliance considerations

- Keep project files private by default; enable ACLs for project sharing
- If accepting uploads, enforce size limits and scan or sanitize content used in reports
- Data retention policy for generated PDFs and stored projects
