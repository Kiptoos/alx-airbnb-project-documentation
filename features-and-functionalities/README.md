# Features & Functionalities (Backend)

This document enumerates the **backend** capabilities the Airbnb Clone must support. It is organized by **Core Functionalities**, **Technical Requirements**, and **Non‑Functional Requirements**.

## Core Functionalities

### 1) User Management
- **Registration**: Sign up as guest or host; email verification (optional), password hashing.
- **Authentication**: Email+password; JWT sessions; optional OAuth (Google, Facebook).
- **Profiles**: Update profile (photo, phone/email, preferences), KYC flags (optional), role upgrades (guest→host).

### 2) Property Listings
- **Create/Read/Update/Delete (CRUD)** for listings with: title, description, location (geo), price (per night), photos, amenities, house rules, availability calendar.
- **Availability Management**: Blocked dates, seasonal pricing (optional), min/max nights.
- **Media**: Upload & manage images (local FS for learning scenario; pluggable to S3/Cloudinary).

### 3) Search & Filtering
- **Search** by location, price range, guests, date window, amenities.
- **Pagination & Sorting** (price asc/desc, rating, recency).
- **Caching**: Popular queries cached (e.g., Redis).

### 4) Booking Management
- **Create booking** with date validation; prevent overlap/double‑booking.
- **Cancel booking** (guest/host) per policy; compute refunds (if payments integrated).
- **Status lifecycle**: pending → confirmed → completed / canceled; audit logs.

### 5) Payments
- **Checkout**: Stripe/PayPal sandbox; handle intents/tokens, currency amounts.
- **Payouts**: Simulate host payout after completion (webhook or cron).
- **Receipts** and idempotency on callbacks.

### 6) Reviews & Ratings
- **Write review**: guest → listing, linked to a completed booking.
- **Respond**: host response to a review.
- **Aggregation**: average rating per listing; anti‑abuse checks.

### 7) Notifications
- **Email** (SendGrid/Mailgun or console stub) for confirmations, cancellations, payout notices.
- **In‑app** events feed (optional).

### 8) Admin
- **Moderation**: manage users, listings, bookings, reviews.
- **Financials**: view payments/payouts summary.
- **Metrics**: basic KPIs (optional).

## Technical Requirements

- **Database**: PostgreSQL/MySQL with tables: `users`, `properties`, `bookings`, `reviews`, `payments`, plus join/lookup tables.
- **API**: REST (GET/POST/PUT/PATCH/DELETE) with proper status codes; optional GraphQL.
- **Auth**: JWT; Role‑Based Access Control (guest/host/admin).
- **File Storage**: Use local filesystem for images (learning); design pluggable adapter for S3/Cloudinary.
- **Email**: Provider adapter (SendGrid/Mailgun); sandbox support.
- **Error Handling & Logging**: Global error middleware, structured logs, correlation IDs.

## Non‑Functional Requirements

- **Scalability**: Modular services (auth, listings, bookings, payments); stateless API; horizontal scaling ready.
- **Security**: Hash passwords (bcrypt/argon2); HTTPS; input validation; rate limiting; audit logs.
- **Performance**: Caching (Redis), N+1 query avoidance, indexes on search columns, async jobs for webhooks.
- **Testing**: Unit & integration tests (e.g., pytest/Jest); mock external services; minimal load tests.

---

> **Output**: This README accompanies the exported PNGs/diagrams and serves as the authoritative feature list for the project documentation.
