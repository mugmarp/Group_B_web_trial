# API Specification — Transcript Request System

**Base URL:** `http://localhost:3000/api` (dev) / `https://api.must-transcript.example.com` (prod)  
**Auth:** JWT Bearer token (access + refresh)  
**Content-Type:** `application/json`

---

## Authentication

### POST `/auth/login`
**Public** — Get access + refresh tokens

**Request:**
```json
{
  "email": "alice.nakato@student.must.ac.ug",
  "password": "password123"
}
```

**Response (200):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "dGhpcyBpcyBhIHJlZnJl...",
  "user": {
    "id": "cuid...",
    "regNo": "2025/BSE/001/PS",
    "name": "Alice Nakato",
    "role": "STUDENT"  // or "ADMIN"
  }
}
```

### POST `/auth/refresh`
**Public** — Exchange refresh token for new access token

**Request:**
```json
{ "refreshToken": "dGhpcyBpcyBhIHJlZnJl..." }
```

**Response (200):** Same as login (new tokens).

---

## Student Endpoints

### GET `/student/requests`
**Auth:** Student JWT  
**Query:** `?page=1&limit=10&status=PENDING`

**Response (200):**
```json
{
  "data": [
    {
      "id": "cuid...",
      "type": "FULL",
      "status": "PENDING",
      "purpose": "Masters application",
      "createdAt": "2026-11-15T10:30:00Z",
      "updatedAt": "2026-11-15T10:30:00Z"
    }
  ],
  "meta": { "page": 1, "limit": 10, "total": 1 }
}
```

### POST `/student/requests`
**Auth:** Student JWT

**Request:**
```json
{
  "type": "FULL",
  "purpose": "Masters application at Makerere University"
}
```

**Response (201):**
```json
{
  "id": "cuid...",
  "type": "FULL",
  "status": "PENDING",
  "purpose": "Masters application at Makerere University",
  "createdAt": "2026-11-15T10:30:00Z"
}
```

---

## Admin Endpoints

### GET `/admin/requests`
**Auth:** Admin JWT  
**Query:** `?page=1&limit=20&status=PENDING&program=BSE&search=alice`

**Response (200):**
```json
{
  "data": [
    {
      "id": "cuid...",
      "student": {
        "regNo": "2025/BSE/001/PS",
        "name": "Alice Nakato",
        "email": "alice.nakato@student.must.ac.ug",
        "program": "BSE",
        "year": 2
      },
      "type": "FULL",
      "status": "PENDING",
      "purpose": "Masters application",
      "createdAt": "2026-11-15T10:30:00Z"
    }
  ],
  "meta": { "page": 1, "limit": 20, "total": 42 }
}
```

### PATCH `/admin/requests/:id`
**Auth:** Admin JWT

**Request:**
```json
{
  "status": "APPROVED",
  "adminNotes": "Verified against SIMS records"
}
```

**Response (200):**
```json
{
  "id": "cuid...",
  "status": "APPROVED",
  "adminNotes": "Verified against SIMS records",
  "processedAt": "2026-11-15T14:22:00Z",
  "updatedAt": "2026-11-15T14:22:00Z"
}
```

### GET `/admin/requests/:id/pdf`
**Auth:** Admin JWT  
**Response:** `application/pdf` — Transcript PDF download

---

## Health Check

### GET `/health`
**Public**

**Response (200):**
```json
{
  "status": "ok",
  "timestamp": "2026-11-15T10:30:00Z",
  "uptime": 3600,
  "version": "1.0.0"
}
```

---

## Error Responses

### 400 Bad Request
```json
{ "error": "VALIDATION_ERROR", "message": "Invalid request body", "details": [...] }
```

### 401 Unauthorized
```json
{ "error": "UNAUTHORIZED", "message": "Invalid or expired token" }
```

### 403 Forbidden
```json
{ "error": "FORBIDDEN", "message": "Insufficient permissions" }
```

### 404 Not Found
```json
{ "error": "NOT_FOUND", "message": "Request not found" }
```

### 409 Conflict
```json
{ "error": "CONFLICT", "message": "Duplicate request for this period" }
```

### 500 Internal Server Error
```json
{ "error": "INTERNAL_ERROR", "message": "Something went wrong" }
```

---

## Validation Rules

| Endpoint | Field | Rules |
|----------|-------|-------|
| `POST /student/requests` | `type` | Required, enum: FULL/PARTIAL/CERTIFIED |
| | `purpose` | Required, string, 10-500 chars |
| `PATCH /admin/requests/:id` | `status` | Required, enum: APPROVED/REJECTED/READY_FOR_PICKUP |
| | `adminNotes` | Optional, string, max 1000 chars |

---

## Rate Limiting (Future)

- Auth endpoints: 10 req/min per IP
- Student endpoints: 60 req/min per user
- Admin endpoints: 120 req/min per user

---

*Version 1.0 | 2026-11-10*