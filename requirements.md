# Backend Requirement Specifications

This specification details API endpoints, validation rules, and performance criteria for three core services: **Authentication**, **Property Listings**, and **Booking**. Endpoints are RESTful; all payloads are JSON; responses include `success`, `data`, and `error` fields.

---

## 1) Authentication Service

### Endpoints
- `POST /api/v1/auth/register`
  - **Body**: `{ "email": string, "password": string (>=8), "role": "guest"|"host" }`
  - **Validations**: unique email; strong password; role in enum.
  - **Response**: `201 Created` with `{ "user": {id, email, role}, "token": JWT }`
- `POST /api/v1/auth/login`
  - **Body**: `{ "email": string, "password": string }`
  - **Response**: `200 OK` with `{ "token": JWT, "user": {id, email, role} }`
- `GET /api/v1/auth/me` (auth required)
  - **Response**: `200 OK` with user profile.
- `PATCH /api/v1/users/:id` (owner or admin)
  - **Body**: profile fields `{ "name"?, "avatarUrl"?, "phone"?, "preferences"? }`
  - **Response**: `200 OK`.

### Security & Performance
- **JWT** in `Authorization: Bearer <token>`; expiry (e.g., 15m access + refresh).
- **Password hashing** (bcrypt/argon2), email normalization.
- **Rate limit**: login and register endpoints (e.g., 5/min/IP).
- **Audit logs** for role changes, password resets.

---

## 2) Property Listings Service

### Endpoints
- `POST /api/v1/properties` (host)
  - **Body**: `{ title, description, pricePerNight, maxGuests, address, lat, lng, amenities[], photos[], rules?, availability? }`
  - **Validations**: required fields; positive price; coordinates valid; photos size/type.
  - **Response**: `201 Created` with property object.
- `GET /api/v1/properties/:id`
  - **Response**: `200 OK` with property, aggregated rating, availability.
- `GET /api/v1/properties`
  - **Query**: `q, lat, lng, radiusKm, minPrice, maxPrice, guests, amenities[], startDate, endDate, sort, page, limit`
  - **Response**: `200 OK` paginated list.
- `PATCH /api/v1/properties/:id` (owner or admin)
  - Update mutable fields; preserve ID/owner.
- `DELETE /api/v1/properties/:id` (owner or admin)

### Performance & Indexing
- DB indexes: `(lat,lng)`, price, createdAt; GIN index on amenities (JSONB) if PostgreSQL.
- Cache: hot queries and listing-by-id (TTL ~60s).
- Media: file storage on disk with unique paths; background thumbnailing (optional).

---

## 3) Booking Service

### Endpoints
- `POST /api/v1/bookings`
  - **Body**: `{ propertyId, checkIn (ISO), checkOut (ISO), guests }`
  - **Validations**: date order; availability check; maxGuests; minNights.
  - **Response**: `201 Created` with booking `{ id, status: "pending"|"confirmed" }`.
- `GET /api/v1/bookings/:id` (owner of booking, host of property, or admin)
- `GET /api/v1/bookings` (auth) — filter by role, status, date range.
- `PATCH /api/v1/bookings/:id/cancel` — cancellation policy rules applied.

### Payments (integration point)
- `POST /api/v1/payments/intent` — create payment intent with provider; amount derived from property nightly rate, fees, taxes.
- Webhooks: `/api/v1/payments/webhook` — idempotent; on success → mark booking confirmed; on failure → expire/cancel.

### Consistency & Concurrency
- Prevent double‑booking: `SELECT ... FOR UPDATE` on date ranges or exclusion constraints.
- Idempotency keys for booking creation and payment callbacks.

---

## Cross‑Cutting Concerns

- **Validation**: Central schema validation (e.g., JSON Schema / pydantic / zod).
- **Errors**: Problem+JSON style; do not leak internals.
- **Observability**: request IDs, structured logs, basic metrics (p95 latency).
- **Docs**: OpenAPI/Swagger JSON at `/api-docs` with examples and auth flows.
- **Testing**: Unit tests for services; integration tests with test DB; payment webhooks mocked.
