# Database & Storage

This section describes how data is stored in **PostgreSQL + PostGIS** and how uploaded documents are managed in **Google Cloud Storage (GCS)**.

---

## Database Engine

- **Database**: PostgreSQL  
- **Extension**: PostGIS (for spatial/geometry support)

PostGIS enables storage and querying of GPS polygons for farmer land parcels.

---

## Table overview

### 1. `farmerdetails`

Stores the core master data for each farmer.

| Column       | Type           | Description                          |
|-------------|----------------|--------------------------------------|
| `id`        | SERIAL (PK)    | Unique farmer ID                     |
| `farmer_name` | TEXT         | Full name of the farmer              |
| `mobile`    | TEXT           | Mobile number                        |
| `address`   | TEXT           | Address / village / district info    |
| `created_at` | TIMESTAMP     | Record creation time                 |

---

### 2. `farmer_documents`

Stores metadata and URLs for uploaded documents. Binary file content is stored in GCS, not in the database.

| Column          | Type           | Description                            |
|-----------------|----------------|----------------------------------------|
| `id`            | SERIAL (PK)    | Unique document record ID              |
| `farmer_id`     | INTEGER (FK)   | References `farmerdetails.id`          |
| `photo_url`     | TEXT           | Public or signed URL to farmer photo   |
| `aadhar_url`    | TEXT           | URL to Aadhaar document                |
| `agreement_url` | TEXT           | URL to agreement document              |
| `uploaded_at`   | TIMESTAMP      | Timestamp of upload                    |

> **Note:** Foreign keys should be indexed for efficient joins.

---

### 3. `land_parcels`

Stores geospatial information for each land parcel linked to a farmer.

| Column      | Type                           | Description                                      |
|------------|---------------------------------|--------------------------------------------------|
| `id`       | SERIAL (PK)                    | Unique land parcel ID                            |
| `farmer_id`| INTEGER (FK)                   | References `farmerdetails.id`                    |
| `geom`     | `GEOMETRY(POLYGON, 4326)`      | Land parcel polygon in WGS84 (lat/lng)          |
| `created_at` | TIMESTAMP                    | When the parcel was recorded                    |

Spatial indexes (e.g. GIST index on `geom`) are recommended for fast spatial queries.

---

## GPS polygon creation

Client applications typically send an array of GPS points:

```json
{
  "coordinates": [
    { "lat": 18.5204, "lng": 73.8567 },
    { "lat": 18.5210, "lng": 73.8575 },
    { "lat": 18.5198, "lng": 73.8580 },
    { "lat": 18.5204, "lng": 73.8567 }
  ]
}
```

The backend converts this into a **PostGIS POLYGON**, for example:

```text
POLYGON((
  73.8567 18.5204,
  73.8575 18.5210,
  73.8580 18.5198,
  73.8567 18.5204
))
```

Key points:

- The first and last coordinates must match to form a closed polygon.
- SRID `4326` is used for latitude/longitude in WGS84.

---

## Google Cloud Storage (GCS)

Uploaded documents are stored in a **GCS bucket** (per environment, e.g. `nbs-farmer-dev`).

Recommended structure:

```text
gs://nbs-farmer-bucket/
  farmers/
    {farmer_id}/
      photo_{timestamp}.jpg
      aadhar_{timestamp}.pdf
      agreement_{timestamp}.pdf
```

- The application uploads files using the service account defined by `GCS_KEY_FILE`.
- After upload, the resulting GCS URL (public or signed) is stored in `farmer_documents`.

---

## Backup & retention (high level)

- **Database**: enable automated backups and point-in-time recovery on the Cloud SQL instance.
- **GCS**: configure lifecycle policies if needed (e.g. archive old versions, delete after N days).

Adjust these strategies as per your organisational compliance and retention policies.


