# API Reference

This section documents the main REST endpoints exposed by the **NBS Farmer Registration API**.

All examples assume the API is available at:

```text
BASE_URL = https://backend-survey-13977221722.asia-south2.run.app
```

---

## Authentication

> **Note:** Adapt this section to your real auth mechanism if it differs.

- In a simple setup, the API may be protected only with **network-level** restrictions (e.g. VPN, private VPC).
- For production, you should consider:
  - API keys or JWT-based authentication.
  - Role-based access control (e.g. field user, admin, backoffice).

---

## Common conventions

- **Content-Type**
  - JSON requests: `application/json`
  - File uploads: `multipart/form-data`
- **Date/time fields** are returned in ISO 8601 format, e.g. `2025-01-01T10:30:00Z`.
- **IDs** are numeric auto-increment primary keys unless otherwise stated.

---

## 1. Farmers

### 1.1 Create farmer

- **Endpoint**: `POST /farmers`
- **Description**: Create a new farmer master record.
- **Request body** (`application/json`):

```json
{
  "farmer_name": "Ravi Kumar",
  "mobile": "9876543210",
  "address": "Village X, District Y, State Z"
}
```

- **Response** `201 Created`:

```json
{
  "id": 1,
  "farmer_name": "Ravi Kumar",
  "mobile": "9876543210",
  "address": "Village X, District Y, State Z",
  "created_at": "2025-01-01T10:30:00Z"
}
```

---

### 1.2 Get farmer by ID

- **Endpoint**: `GET /farmers/{id}`
- **Description**: Fetch a single farmer record.
- **Path parameters**:
  - `id` – numeric farmer ID

- **Response** `200 OK`:

```json
{
  "id": 1,
  "farmer_name": "Ravi Kumar",
  "mobile": "9876543210",
  "address": "Village X, District Y, State Z",
  "created_at": "2025-01-01T10:30:00Z"
}
```

If the farmer is not found, return `404 Not Found`:

```json
{
  "error": "Farmer not found"
}
```

---

### 1.3 List farmers (optional)

- **Endpoint**: `GET /farmers`
- **Query parameters** (optional):
  - `page` – page number (default: `1`)
  - `limit` – page size (default: `20`)

- **Response** `200 OK`:

```json
{
  "items": [
    {
      "id": 1,
      "farmer_name": "Ravi Kumar",
      "mobile": "9876543210"
    }
  ],
  "page": 1,
  "limit": 20,
  "total": 1
}
```

---

## 2. Document Uploads

### 2.1 Upload farmer documents

- **Endpoint**: `POST /farmers/{id}/documents`
- **Description**: Upload one or more documents for a farmer.
- **Content-Type**: `multipart/form-data`
- **Path parameters**:
  - `id` – farmer ID

- **Form fields**:
  - `photo` – image file (optional)
  - `aadhar` – image/PDF file (optional)
  - `agreement` – image/PDF file (optional)

- **Response** `201 Created`:

```json
{
  "farmer_id": 1,
  "photo_url": "https://storage.googleapis.com/nbs-farmer-bucket/photo_1.jpg",
  "aadhar_url": "https://storage.googleapis.com/nbs-farmer-bucket/aadhar_1.pdf",
  "agreement_url": "https://storage.googleapis.com/nbs-farmer-bucket/agreement_1.pdf"
}
```

If the farmer does not exist, return `404 Not Found`.

---

## 3. Land Parcels

### 3.1 Save land parcel polygon

- **Endpoint**: `POST /farmers/{id}/land-parcels`
- **Description**: Persist a land parcel geometry for a farmer.
- **Content-Type**: `application/json`

- **Request body**:

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

> **Note:** The first and last point should be the same to form a closed polygon.

- **Response** `201 Created`:

```json
{
  "id": 10,
  "farmer_id": 1,
  "area_sq_m": 12450.32,
  "srid": 4326
}
```

Internally, the API converts the coordinate array into a **PostGIS POLYGON** with SRID `4326`.

---

### 3.2 List land parcels for a farmer

- **Endpoint**: `GET /farmers/{id}/land-parcels`
- **Description**: Retrieve all land parcels linked to a farmer.

- **Response** `200 OK` (example):

```json
[
  {
    "id": 10,
    "farmer_id": 1,
    "area_sq_m": 12450.32,
    "centroid": {
      "lat": 18.5208,
      "lng": 73.8574
    }
  }
]
```

---

## 4. Health & Utility

### 4.1 Health check

- **Endpoint**: `GET /health`
- **Response** `200 OK`:

```json
{
  "status": "ok",
  "uptime": 123.45
}
```

This endpoint can be used by load balancers and monitoring systems to verify that the service is running.

---

## Error handling

The API should follow a consistent error response structure. A recommended format is:

```json
{
  "error": "ValidationError",
  "message": "farmer_name is required",
  "details": {
    "field": "farmer_name"
  }
}
```

Common HTTP status codes:

- `400 Bad Request` – invalid payload or missing required fields.
- `401 Unauthorized` / `403 Forbidden` – authentication/authorization failures (if implemented).
- `404 Not Found` – resource does not exist.
- `500 Internal Server Error` – unexpected server error.


