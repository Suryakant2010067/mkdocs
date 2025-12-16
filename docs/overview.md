# System Overview

This section describes the overall design of the **NBS Farmer Registration API**.

---

## High-level responsibilities

The backend service is responsible for:

- **Survey form (data capture)**
  - Create/update farmer master records.
  - Upload documents (photo, Aadhaar, agreement) via multipart form.
  - Accept GPS coordinates and build PostGIS polygons for land parcels.
  - Track timestamps and basic audit information.
- **Data visualization (insights)**
  - Provide clean, queryable data to dashboards/BI for farmer counts, parcel areas, and upload completeness.
  - Surface health/status (e.g., missing documents, invalid polygons) for QA.
- **Integration & security**
  - Restrict access to trusted frontends via CORS.
  - Expose a REST API suitable for PWA/web/other services.

---

## Logical components

- **API Layer (Express)**
  - Defines routes for farmers, documents, and land parcels.
  - Handles validation, error handling, and response shaping.

- **Storage Layer (PostgreSQL + PostGIS)**
  - Stores farmer master data, document metadata, and polygon geometries.
  - Exposes spatial querying capabilities through PostGIS.

- **File Storage (Google Cloud Storage)**
  - Stores binary file content for documents.
  - Buckets are organised per environment (e.g. `nbs-farmer-dev`, `nbs-farmer-prod`).  

- **Connectivity & Infrastructure**
  - Cloud SQL instance hosts the PostgreSQL + PostGIS database.
  - Cloud SQL Connector is used (or recommended) for secure database access.

---

## Typical request flow

1. **Client sends request**
   - Survey/PWA submits data (e.g. `POST /farmers`, `/documents`, `/land-parcels`).
2. **Validation & parsing**
   - JSON or multipart/form-data parsed and validated (fields, file sizes/types).
3. **Business logic**
   - Farmer record created/updated; files uploaded to GCS; GPS converted to PostGIS polygon.
4. **Persistence**
   - PostgreSQL tables updated (`farmerdetails`, `farmer_documents`, `land_parcels`).
5. **Response**
   - Structured JSON returned (IDs, URLs, status). Dashboards/BI later read from the same database/views for visualization.

This API is intended to be used primarily by **PWA or web applications**, but can also be consumed by other backend services.
