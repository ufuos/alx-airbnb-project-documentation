# Airbnb Clone — Backend Requirement Specifications

**Repository:** `alx-airbnb-project-documentation`
**File:** `requirements.md`
**Date:** 2025-08-28

---

## Overview

This document specifies technical and functional requirements for three key backend features:

1. **User Authentication & Authorization**
2. **Property (Listing) Management**
3. **Booking System**

For each feature we include: purpose, REST API endpoints, request/response schemas, validation rules, error cases, performance & scalability criteria, security considerations, and testing notes.

---

# 1. User Authentication & Authorization

**Purpose:** Securely register and authenticate users (Guests and Hosts), issue JWTs, support OAuth login, and provide role-based access control (RBAC).

## Entities / DB table (example - PostgreSQL)

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash TEXT, -- nullable for OAuth users
  name VARCHAR(255),
  role VARCHAR(16) NOT NULL DEFAULT 'guest', -- 'guest' | 'host' | 'admin'
  avatar_url TEXT,
  phone VARCHAR(32),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  is_verified BOOLEAN DEFAULT FALSE
);
CREATE INDEX idx_users_email ON users(email);
```

## API Endpoints

### 1.1 Register (email/password)

- `POST /api/v1/auth/register`
- **Request**

```json
{
  "name": "string (optional)",
  "email": "string (required, email)",
  "password": "string (required, min 8)",
  "role": "string (optional, 'guest'|'host')"
}
```

- **Response (201 Created)**

```json
{
  "id": "uuid",
  "email": "string",
  "role": "guest|host",
  "created_at": "ISO8601"
}
```

- **Validation rules**

  - Email must be RFC2822-compliant.
  - Password: minimum 8 characters, at least one letter and one number (or stronger policy).
  - Role can only be `guest` or `host` (admin assigned manually).

- **Error codes**

  - `400` Bad Request (validation failed)
  - `409` Conflict (email already registered)

- **Security**

  - Store password with bcrypt (cost/work factor = 12 or env-configured).
  - Rate-limit endpoint to 10 req/min per IP.
  - Send email verification token (expiring JWT or signed token).

- **Performance**

  - Typical latency: <200ms for auth DB operations under normal load (scale targets subject to infra).

### 1.2 Login (email/password)

- `POST /api/v1/auth/login`
- **Request**

```json
{ "email": "string", "password": "string" }
```

- **Response (200 OK)**

```json
{
  "accessToken": "jwt (short-lived, e.g., 15m)",
  "refreshToken": "opaque or jwt (long-lived, e.g., 30d)",
  "user": { "id": "uuid", "email": "string", "role": "guest|host|admin" }
}
```

- **Validation**

  - Check email exists and bcrypt-compare password.

- **Security**

  - Use asymmetric JWT signing (RS256) or HS256 with strong secret stored in secrets manager.
  - Store refresh tokens in DB (or use rotating refresh tokens) with secure revocation support.
  - Implement refresh token rotation and revocation on logout.

- **Error codes**

  - `401` Unauthorized (invalid credentials)
  - `429` Too Many Requests (brute force protection)

### 1.3 OAuth Login (Google/Facebook)

- `POST /api/v1/auth/oauth`
- **Request**

```json
{ "provider": "google|facebook", "idToken": "string" }
```

- **Behavior**

  - Validate token with provider.
  - Create or update user record; return tokens like standard login.

- **Security**

  - Link social accounts by email only after verifying provider token.

### 1.4 Token Refresh

- `POST /api/v1/auth/refresh`
- **Request**

```json
{ "refreshToken": "string" }
```

- **Response**

```json
{ "accessToken": "jwt", "refreshToken": "new-refresh-or-same" }
```

- **Security**

  - Rotate refresh tokens and store the current valid token/hash server-side.

### 1.5 Logout / Revoke

- `POST /api/v1/auth/logout`
- **Request**

```json
{ "refreshToken": "string" }
```

- **Behavior**

  - Invalidate refresh token in DB; optionally add short-lived entry in revocation list for access tokens.

## Middleware & RBAC

- `authMiddleware` verifies access token and attaches `req.user`.
- `requireRole(...roles)` ensures user has one of the allowed roles.

## Testing Notes

- Unit tests for registration, login, token refresh, and errors.
- Integration tests to ensure token rotation and revocation.

---

# 2. Property (Listing) Management

**Purpose:** CRUD for property listings, image uploads, metadata (amenities, coordinates), and search-friendly fields.

## Entities / DB tables

```sql
CREATE TABLE properties (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id UUID REFERENCES users(id) NOT NULL,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  price NUMERIC(10,2) NOT NULL,
  currency CHAR(3) NOT NULL DEFAULT 'USD',
  address JSONB, -- street, city, state, country, postal_code
  location GEOGRAPHY(Point, 4326), -- lat/lon for geospatial queries
  capacity INTEGER NOT NULL DEFAULT 1,
  amenities JSONB, -- array of amenity ids or objects
  status VARCHAR(16) DEFAULT 'active', -- active | paused | removed
  rating_avg NUMERIC(3,2) DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);
CREATE INDEX idx_properties_location ON properties USING GIST (location);
CREATE INDEX idx_properties_owner ON properties(owner_id);
```

### Images

- `property_images` table storing cloud URLs and order/alt text.

## API Endpoints

### 2.1 Create Listing

- `POST /api/v1/properties`
- Auth: `requireRole('host')` or `requireRole('admin')`.
- **Request (multipart/form-data or JSON with image URLs)**

```json
{
  "title": "string (required)",
  "description": "string",
  "price": "number (required, >=0)",
  "currency": "string (3-letter)",
  "address": {"street":"","city":"","country":"","postal_code":""},
  "latitude": number,
  "longitude": number,
  "capacity": integer,
  "amenities": ["wifi","air-conditioning"]
}
```

- **Response (201)**

```json
{ "id": "uuid", "owner_id": "uuid", "title": "...", "created_at": "ISO8601" }
```

- **Validation rules**

  - `price` must be >= 0.
  - `latitude` in \[-90,90], `longitude` in \[-180,180].
  - `title` length: 5-255 chars.

- **Images**

  - Prefer direct upload to Cloudinary/S3 with signed upload URLs.
  - Enforce max file size (e.g., 10 MB) and max images per listing (e.g., 20).

### 2.2 Get Listing / List Listings

- `GET /api/v1/properties/:id` — returns full property details including images and average rating.
- `GET /api/v1/properties` — query params: `?q=city&minPrice=&maxPrice=&guests=&amenities=wifi,pool&page=1&limit=20&sort=price_asc|rating_desc&lat=&lng=&radius=10km`
- **Response (200)**

```json
{
  "items": [ { "id":"","title":"","price":0.0, "thumbnail_url":"", "location":{...} } ],
  "page": 1, "limit": 20, "total": 123
}
```

- **Performance**

  - Use DB pagination with `LIMIT/OFFSET` or keyset pagination for large result sets.
  - Cache popular queries in Redis for 30–300 seconds.

### 2.3 Update Listing

- `PATCH /api/v1/properties/:id` — Only owner or admin.
- Partial updates allowed; update `updated_at`.

### 2.4 Delete / Suspend Listing

- `DELETE /api/v1/properties/:id` — soft delete: set `status='removed'`.

## Validation & Business Rules

- Only Hosts can create listings; Admin can create/edit any.
- Owner cannot set price to zero for public listings unless explicitly allowed (promotional flow).
- Enforce coordinate presence if address provided for mapping features.

## Security

- Validate image MIME types and scan for malware via file scanning pipeline (if available).

## Indexing & Query Optimization

- Index price, rating, capacity, and location (GIST) for geospatial.

## Testing

- Unit tests for create/update/delete and authorization checks.
- Integration tests for search endpoints, pagination, and geo queries.

---

# 3. Booking System

**Purpose:** Allow guests to book available properties for date ranges, prevent double-bookings, maintain booking lifecycle, and handle cancellations/refunds.

## Entities / DB tables

```sql
CREATE TABLE bookings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  property_id UUID REFERENCES properties(id) NOT NULL,
  guest_id UUID REFERENCES users(id) NOT NULL,
  host_id UUID REFERENCES users(id) NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  nights INTEGER NOT NULL,
  total_amount NUMERIC(12,2) NOT NULL,
  currency CHAR(3) NOT NULL DEFAULT 'USD',
  status VARCHAR(16) NOT NULL DEFAULT 'pending', -- pending|confirmed|cancelled|completed|refunded
  payment_id UUID NULL,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(property_id, start_date, end_date) -- simplified; real conflict detection is interval-based
);
-- Additional table: booking_availability rules, cancellations
```

> **Note:** Use interval-overlap checks for conflict detection rather than a naive UNIQUE. Example Postgres exclusion constraint using `tsrange` or custom logic.

## API Endpoints

### 3.1 Create Booking

- `POST /api/v1/bookings`
- Auth: `requireRole('guest')`.
- **Request**

```json
{
  "propertyId": "uuid",
  "startDate": "YYYY-MM-DD",
  "endDate": "YYYY-MM-DD",
  "guests": integer,
  "paymentMethodId": "string (optional)"
}
```

- **Server-side steps**

  1. Validate dates: `startDate < endDate`, minimum advance booking (e.g., 1 day), and max stay length (e.g., 30 days).
  2. Load property and check `status='active'`.
  3. Check capacity >= requested guests.
  4. Check availability — use transactional check + row-level locking (SELECT ... FOR UPDATE) or create an exclusion constraint with `daterange`.
  5. Calculate total price (price per night \* nights + cleaning fees + taxes + host fees).
  6. Create booking record with status `pending`.
  7. Initiate payment authorization (Stripe: PaymentIntent or SetupIntent). If payment succeeds, set status `confirmed`, else `pending_payment`.

- **Response (201)**

```json
{ "id": "uuid", "status": "pending|confirmed", "total_amount": 123.45 }
```

- **Error codes**

  - `400` Validation error (bad dates)
  - `409` Conflict (dates not available)
  - `402` Payment Required (payment failed)

### 3.2 Get Booking

- `GET /api/v1/bookings/:id` — only guest (owner of booking), host (owner of property), or admin.

### 3.3 List Bookings

- `GET /api/v1/bookings?role=guest|host&page=&limit=&status=` — returns bookings for user.

### 3.4 Cancel Booking

- `POST /api/v1/bookings/:id/cancel`
- Business rules:

  - If cancellation is within allowed window, process refund according to property policy.
  - Mark booking `status='cancelled'` and create a `refund` record referencing `payment_id`.
  - Use Stripe refund API for refunds; track refund id and status.

### 3.5 Complete Booking

- `POST /api/v1/bookings/:id/complete` — host or system cron marks completed after `endDate` and release payouts after hold period.

## Concurrency & Data Integrity

- Use database-level exclusion constraints or transaction + advisory locks to prevent double-bookings.
- Example Postgres exclusion using `tstzrange` or `daterange`:

```sql
-- pseudo-example
ALTER TABLE bookings ADD EXCLUDE USING GIST (property_id WITH =, daterange(start_date, end_date, '[]') WITH &&);
```

## Payments & Payouts

- Integrate with Stripe Connect for host payouts.
- Implement platform fee splitting: hold platform fee and transfer rest to host after check-out and hold window.
- Support partial & full refunds.

## Notifications

- On booking creation/confirmation/cancellation -> send email and in-app notification to guest and host.

## Performance Criteria

- Booking creation latency target: <500ms for reads; write path (with payment) may be 1–2s depending on payment provider.
- System must handle burst booking attempts (e.g., flash demand) via queueing and backpressure; protect via rate-limits.

## Testing

- Integration tests to simulate concurrent booking attempts for same property/time window.
- Unit tests for pricing calculation, date validation, and refund logic.

---

# Cross-Cutting Requirements

## API Design & Standards

- All endpoints prefixed with `/api/v1/`.
- Use consistent pagination: `page`, `limit`, and `nextCursor` for cursor-based pagination when needed.
- Use proper HTTP status codes and structured error responses, example:

```json
{
  "error": {
    "code": "BOOKING_CONFLICT",
    "message": "Selected dates are not available",
    "fields": ["startDate", "endDate"]
  }
}
```

## Validation & Error Handling

- Centralized request validation layer (e.g., Joi, Zod, or express-validator).
- Global error handler returning JSON and logging errors.

## Security

- TLS for all traffic.
- Secrets in secret manager (AWS Secrets Manager / HashiCorp Vault).
- Rate limiting per IP and per user (auth endpoints stricter).
- Input sanitization and parameterized DB queries to prevent injection.
- CSP, CORS policy configured in API gateway.

## Observability

- Structured logs (Winston or similar) with request IDs.
- Metrics (Prometheus) for request latencies, error rates, and queue lengths.
- Distributed tracing for performance debugging (OpenTelemetry).

## Caching

- Redis for session caches, short-lived query caches (listings), rate-limiting store, and pub/sub for notifications.

## Testing & CI

- Unit tests: Mocha/Jest/AVA with 80%+ coverage on core modules.
- Integration tests against test DB and mocked payment/email providers.
- CI pipeline (GitHub Actions) running lint, tests, and type checks; deploy on passing builds.

---

# Appendix: Sample JSON Schemas

### User (response)

```json
{
  "type": "object",
  "properties": {
    "id": { "type": "string", "format": "uuid" },
    "email": { "type": "string", "format": "email" },
    "name": { "type": "string" },
    "role": { "type": "string" }
  },
  "required": ["id", "email", "role"]
}
```

### Booking create (request)

```json
{
  "type": "object",
  "properties": {
    "propertyId": { "type": "string", "format": "uuid" },
    "startDate": { "type": "string", "format": "date" },
    "endDate": { "type": "string", "format": "date" },
    "guests": { "type": "integer", "minimum": 1 }
  },
  "required": ["propertyId", "startDate", "endDate", "guests"]
}
```

---

# How to add this file to your repository

From repository root (local clone):

```bash
# create directory and save file
mkdir -p docs && cat > requirements.md <<'EOF'
# (paste the content of this file here)
EOF

git add requirements.md
git commit -m "Add backend requirement specifications for User Auth, Property Management, Booking System"
git push origin main
```

> **Note:** I can't push to your GitHub from here. If you'd like, provide a GitHub personal access token with repo scope (or invite me as a collaborator via a service account) and I can create a PR or push for you — otherwise run the commands above in your local environment.

---

If you want, I can now:

- Generate a Draw\.io XML representing the entities and flows described above.
- Produce OpenAPI (Swagger) spec for the endpoints (YAML/JSON).
- Create example Postman collection or cURL examples for each endpoint.

Which would you like next?
