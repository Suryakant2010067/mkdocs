# Setup & Configuration

This guide explains how to run the **NBS Farmer Registration API** locally and configure it for cloud deployment.

---

## Prerequisites

- **Node.js** LTS (e.g. 18.x or newer)
- **npm** or **yarn**
- Access to a **PostgreSQL** instance with **PostGIS** extension enabled
- A **Google Cloud project** with:
  - Cloud Storage bucket
  - (Recommended) Cloud SQL instance for PostgreSQL
  - Service account with required permissions

---

## Installation

1. Clone or download the project source code.
2. Install dependencies:

```bash
npm install
```

3. Create an `.env` file in the project root.

---

## Required environment variables

```env
PORT=3000

DB_USER=postgres
DB_PASS=your_password
DB_NAME=farmers_db
DB_HOST=localhost
DB_PORT=5432

# If using Cloud SQL Connector, this is typically: project:region:instance
INSTANCE_CONNECTION_NAME=project:region:instance

# Google Cloud Storage
GCS_BUCKET_NAME=nbs-farmer-bucket
GCS_KEY_FILE=service-account.json
```

> **Note:** Adjust database host/port according to your environment (local Docker, Cloud SQL, etc.).

---

## Running locally

```bash
npm run dev        # or: node index.js / npm start
```

The API will start on `http://localhost:3000` (or the port you set via `PORT`).

---

## Basic health check

Expose a simple health endpoint (for example):

- `GET /health` → returns `{ "status": "ok" }`  

Use this endpoint for uptime checks, load balancer health probes, or basic troubleshooting.

---

## Deployment notes (high level)

- **Application hosting**: Deploy on a platform like Cloud Run, App Engine, or a VM.
- **Database**: Use **Cloud SQL for PostgreSQL** with **PostGIS** enabled.
- **Storage**: Use a dedicated **GCS bucket** per environment (dev/test/prod).
- **Secrets**: Prefer managed secret storage (e.g. Secret Manager) rather than committing secrets in `.env` to version control.
