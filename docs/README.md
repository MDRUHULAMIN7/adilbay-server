# Furnixo / Adilbay — Backend Documentation Index

Welcome to the backend documentation for the **Furnixo / Adilbay** E-Commerce platform. This directory contains all technical specifications, architectural blueprints, database schemas, and API documentation for the backend services.

---

## 📚 Documentation Directory

| Document | Description | Direct Link |
| :--- | :--- | :--- |
| **Backend Architecture & SRS** | Master 2,000+ line Software Requirement Specification (SRS) detailing system design, Prisma schema, API endpoints, Zod schemas, caching, and error handling | [backend-architecture-srs.md](file:///d:/Restart/CodeClub/adilbay/adilbay-server/docs/backend-architecture-srs.md) |

---

## 🏛️ System Architecture Summary

The backend is built with **Node.js, Express.js, TypeScript, PostgreSQL, Prisma ORM, and Redis**. It adheres to a strict layered clean architecture:

```
[ Client Request ]
       │
       ▼
[ Security & Middleware Layer ]
   ├── Helmet (HTTP security headers)
   ├── CORS (Strict origin check)
   ├── Rate Limiting (express-rate-limit + Redis store)
   ├── Auth Middleware (JWT access verification)
   └── Zod Validation (Request body, params, query)
       │
       ▼
[ Routing Layer ] (`src/routes/`)
       │
       ▼
[ Controllers ] (`src/controllers/`) ─── HTTP req/res handling, status codes
       │
       ▼
[ Services ] (`src/services/`) ─────── Pure business logic, transactions
       │
       ▼
[ Data Access Layer ]
   ├── Prisma ORM (`prisma/schema.prisma`) ───> PostgreSQL Database
   └── Redis Client (`ioredis`) ──────────────> In-Memory Cache (Cart, OTP, Sessions)
```

---

## 🗄️ Database & Schema Highlights

- **ORM**: Prisma Client with PostgreSQL
- **Key Models**:
  - `User`, `Account`, `RefreshToken`, `Address`
  - `Category`, `Brand`, `Product`, `ProductVariant`, `ProductImage`, `Inventory`
  - `Cart`, `CartItem` (with Redis caching layer)
  - `Order`, `OrderItem`, `Payment`, `Shipment`
  - `Review`, `Wishlist`, `Coupon`
- **Schema Location**: Detailed in [Section 4 of backend-architecture-srs.md](file:///d:/Restart/CodeClub/adilbay/adilbay-server/docs/backend-architecture-srs.md#4-complete-database-schema-prismaschemaprisma)

---

## 🔒 Security Architecture

- **Authentication**: Dual-token strategy with short-lived JWT Access Tokens (Header `Bearer`) and rotating Refresh Tokens (Secure HttpOnly cookies).
- **Phone Verification**: OTP generation via SMS Gateway (BulkSMSBD) with strict Redis-based rate limiting (prevents SMS bombing).
- **Role-Based Access Control (RBAC)**: Fine-grained permissions for `CUSTOMER`, `VENDOR`, and `ADMIN`.
- **Validation**: Strict schema validation with Zod on all incoming requests before controllers execute.

---

## 🚀 Quick Links for Developers

- **API Specification**: [Section 6 of backend-architecture-srs.md](file:///d:/Restart/CodeClub/adilbay/adilbay-server/docs/backend-architecture-srs.md#6-complete-api-design--endpoint-specification)
- **Reusable QueryBuilder**: [Section 10 of backend-architecture-srs.md](file:///d:/Restart/CodeClub/adilbay/adilbay-server/docs/backend-architecture-srs.md#10-global-reusable-querybuilder-with-metadata-pagination-srchelpersquerybuilderts)
- **Error Handling Architecture**: [Section 14 of backend-architecture-srs.md](file:///d:/Restart/CodeClub/adilbay/adilbay-server/docs/backend-architecture-srs.md#14-global-enterprise-error-handling--dual-persona-diagnostic-engine)
- **Root Overview**: [Root Project README](file:///d:/Restart/CodeClub/adilbay/README.md)
