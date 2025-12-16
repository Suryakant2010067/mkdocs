
# NBS Farmer Registration API

Welcome to the **official documentation** for the **NBS Farmer Registration API**.

This backend service powers farmer onboarding, document management, and geospatial land parcel storage for field and web applications.

---

## What this API does

- **Farmer registration** – capture and store core farmer details.
- **Document upload** – securely upload and store Photo, Aadhaar, and Agreement files.
- **Land parcel management** – save land boundaries as GPS polygons using PostGIS.
- **Cloud-native storage** – persist files in **Google Cloud Storage (GCS)**.
- **Secure database access** – connect to **Cloud SQL (PostgreSQL + PostGIS)** using secure connection patterns.

---

## Technology stack

- **Runtime**: Node.js with Express
- **Database**: PostgreSQL + PostGIS (for spatial data)
- **Object Storage**: Google Cloud Storage
- **Connectivity**: Cloud SQL Connector (recommended)
- **Middleware**: Multer (file uploads), CORS (controlled frontend access)

---

## How to use this documentation

- **Overview** – high-level architecture and core components of the system.
- **Setup & Configuration** – environment variables, local setup, and deployment notes.
- **API Reference** – endpoints, request/response formats, status codes, and examples.
- **Database & Storage** – table design, spatial columns, and file storage layout.

If you are starting from scratch, begin with **Setup & Configuration**, then move to the **API Reference**.

---

## Solution at a glance

The product is used in **two parts**:

1) **Survey Form (Data Capture)** – field or web app users enter farmer details, upload documents, and draw/submit land parcel polygons. This feeds the API endpoints for farmers, documents, and land parcels.  
2) **Data Visualization (Insights & QA)** – dashboards/BI consume the API/database to display coverage, parcel areas, data completeness, and upload status. Typical outputs include farmer counts by region, parcel area summaries, and document upload health.

