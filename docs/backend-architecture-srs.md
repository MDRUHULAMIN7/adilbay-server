# Furnixo - Comprehensive Backend Architecture, SRS & Implementation Blueprint

**Project Name:** Furnixo E-Commerce Backend  
**Target Platform:** Furniture E-Commerce (Bangladesh Market Focus)  
**Frontend Stack:** Next.js (App Router), TypeScript, Tailwind CSS, TanStack Query/Form  
**Live Frontend:** [furnixo.vercel.app](https://furnixo.vercel.app/) | [Dashboard](https://furnixo.vercel.app/dashboard)  
**Backend Tech Stack:** Node.js, Express.js, TypeScript, PostgreSQL, Prisma ORM, Redis, Zod, JWT, BulkSMSBD  

---

## 1. System Architecture & High-Level Overview

```
                                    +-------------------------------------------------+
                                    |         Furnixo Next.js Frontend App            |
                                    |     (Client Browser / Next.js Server Components)|
                                    +-------------------------------------------------+
                                                             |
                                                             | HTTPS (CORS restricted)
                                                             v
+------------------------------------------------------------------------------------------------------------------+
|                                             Furnixo Express Backend API                                          |
|                                                                                                                  |
|  [Security Layer]                                                                                                |
|   ├── Helmet (HTTP Security Headers)                                                                             |
|   ├── CORS (Strict Origin: furnixo.vercel.app)                                                                    |
|   └── Rate Limiter (express-rate-limit + Redis store) -> Blocks OTP SMS Bombing & Brute Force                      |
|                                                                                                                  |
|  [Routing & Middlewares]                                                                                         |
|   ├── Auth Middleware (JWT Access Token in Bearer Header + Refresh Token in HttpOnly Cookie)                     |
|   ├── Role Guard Middleware (ADMIN vs USER permission matrix)                                                    |
|   └── Zod Validation Middleware (Request body, params & query validation)                                        |
|                                                                                                                  |
|  [Layered Application Pattern]                                                                                    |
|   Routes ───> Controllers (HTTP req/res) ───> Services (Business Logic) ───> Repositories / Prisma ORM            |
+------------------------------------------------------------------------------------------------------------------+
          |                                      |                                      |
          v                                      v                                      v
+-----------------------+              +-----------------------+              +-----------------------+
|  PostgreSQL Database  |              |      Redis Cache      |              |   BulkSMSBD Gateway   |
| (Prisma Client Layer) |              | (Cart, OTP, Hot Data) |              |  (Phone OTP & Alerts) |
+-----------------------+              +-----------------------+              +-----------------------+
```

---

## 2. Directory & Layer Structure (Where & Why Each File Exists)

```
furnixo-backend/
├── prisma/
│   ├── schema.prisma                  # Central Prisma Database Schema definition
│   └── seed.ts                        # Initial database seeder (Admin account, base attributes, categories)
├── src/
│   ├── @types/                        # Express namespace expansions (e.g., req.user typed with User payload)
│   ├── config/                        # Environment variables, database clients, Redis & SMS configs
│   │   ├── env.ts                     # Zod-validated environment configuration (fails fast on startup)
│   │   ├── prisma.ts                  # Singleton Prisma Client instance
│   │   └── redis.ts                   # Singleton ioredis client instance
│   ├── constants/                     # Shared constants, enums, status codes, regexes
│   │   ├── regex.ts                   # Bangladesh phone number regex, password rules
│   │   └── messages.ts                # Unified error and success messages
│   ├── controllers/                   # HTTP handling layer: extracts req, calls services, sends res
│   │   ├── auth.controller.ts         # Send OTP, Verify OTP, Register, Login, Refresh, Logout
│   │   ├── user.controller.ts         # User profile, saved addresses, wishlist
│   │   ├── product.controller.ts      # Product listing, filtering, search, single product
│   │   ├── category.controller.ts     # Category and subcategory public & admin endpoints
│   │   ├── cart.controller.ts         # Redis-backed shopping cart for authenticated users
│   │   ├── order.controller.ts        # Order checkout, order tracking, invoice data
│   │   ├── customOrder.controller.ts  # Furniture configurator requests, quote acceptance
│   │   ├── review.controller.ts       # Verified purchase product reviews
│   │   ├── coupon.controller.ts       # Coupon verification & application
│   │   ├── setting.controller.ts      # Site settings, branding, delivery charges
│   │   └── admin.controller.ts        # Admin dashboard analytics, order status updates, quotations
│   ├── middlewares/                   # Reusable request filters and interceptors
│   │   ├── auth.middleware.ts         # Verifies JWT access token from Authorization header
│   │   ├── role.middleware.ts         # Restricts access by user role (ADMIN vs USER)
│   │   ├── validate.middleware.ts     # Generic Zod validation middleware for body, query, and params
│   │   ├── rateLimiter.middleware.ts  # IP & phone rate limiters to prevent abuse
│   │   └── error.middleware.ts        # Global centralized error handler (Zod, Prisma, HTTP errors)
│   ├── routes/                        # Express Routers mapping endpoints to controllers
│   │   ├── index.ts                   # Master router combining all sub-routers under /api
│   │   ├── auth.routes.ts
│   │   ├── user.routes.ts
│   │   ├── product.routes.ts
│   │   ├── category.routes.ts
│   │   ├── cart.routes.ts
│   │   ├── order.routes.ts
│   │   ├── customOrder.routes.ts
│   │   ├── review.routes.ts
│   │   ├── coupon.routes.ts
│   │   ├── setting.routes.ts
│   │   └── admin.routes.ts
│   ├── services/                      # Pure business logic & database queries (Prisma/Redis)
│   │   ├── auth.service.ts
│   │   ├── user.service.ts
│   │   ├── product.service.ts
│   │   ├── cart.service.ts
│   │   ├── order.service.ts
│   │   ├── customOrder.service.ts
│   │   ├── review.service.ts
│   │   ├── coupon.service.ts
│   │   └── setting.service.ts
│   ├── types/                         # Dedicated TypeScript interfaces & DTOs (Strict separation)
│   │   ├── common.types.ts            # Generic API response, Express extensions, status types
│   │   ├── query.types.ts             # Pagination, Sorting, Filter, and Metadata interfaces
│   │   ├── auth.types.ts              # Auth payloads, token payloads, registration DTOs
│   │   ├── product.types.ts           # Product filter criteria, create/update DTOs
│   │   ├── order.types.ts             # Order items, checkout payload, order timeline
│   │   └── customOrder.types.ts       # Configurator request DTO, quotation payload
│   ├── helpers/                       # Reusable logic, query builders, and business helpers
│   │   ├── queryBuilder.ts            # Global Generic QueryBuilder (Search, Filter, Sort, Pagination, Meta)
│   │   ├── paginationHelper.ts        # Page/Limit/Skip calculator and metadata builder
│   │   └── orderNumberGenerator.ts    # Sequential readable order ID generator (e.g., FXO-2026-0001)
│   ├── utils/                         # Standalone utilities & 3rd party wrappers
│   │   ├── apiResponse.ts             # Standardized API response format { success, message, data, meta, error }
│   │   ├── jwt.ts                     # Generate and verify Access (7d) & Refresh (30d) tokens
│   │   ├── hash.ts                    # bcrypt password hashing and verification
│   │   ├── sms.ts                     # BulkSMSBD API integration
│   │   └── slugify.ts                 # String slugifier for SEO-friendly URLs
│   ├── validations/                   # Zod request validation schemas
│   │   ├── auth.validation.ts
│   │   ├── user.validation.ts
│   │   ├── product.validation.ts
│   │   ├── order.validation.ts
│   │   ├── customOrder.validation.ts
│   │   ├── review.validation.ts
│   │   └── coupon.validation.ts
│   ├── app.ts                         # Express application setup (middlewares, routes)
│   └── server.ts                      # Server entrypoint (listens on PORT, graceful shutdown)
├── .env.example
├── package.json
└── tsconfig.json
```

---

## 3. Security & Performance Deep Dive (Problem vs. Solution)

| Security / Performance Feature | Why It Is Needed (Technical Benefit) | Attack / Bug / Problem It Solves |
| :--- | :--- | :--- |
| **Refresh Token in HttpOnly Cookie** | Access Token (7 days) is sent in Authorization header. Refresh Token (30 days) is stored in a cookie with `httpOnly: true`, `secure: true`, `sameSite: 'strict'`. | **XSS (Cross-Site Scripting) Token Theft:** JavaScript running in the browser cannot read `HttpOnly` cookies. If an attacker injects a malicious script, they cannot steal the refresh token. |
| **Strict Rate Limiting (`express-rate-limit` + Redis)** | Limits the number of OTP requests (max 3 requests / hour per phone / IP) and login attempts (max 5 requests / 15 min per IP). | **SMS Bombing & BulkSMSBD Balance Drain:** Attackers cannot script repeated OTP calls to drain SMS balance. It also blocks **Brute-Force Password Guessing**. |
| **Dual Validation (Client + Server with Zod)** | Frontend validates instantly for high UX. Backend strictly validates every field using identical Zod schemas before touching business logic. | **API Tampering & Invalid Data:** Malicious users bypassing frontend controls using Postman/curl are stopped dead at the Express boundary. Prevents database type mismatches. |
| **Soft Delete Pattern (`isDeleted`, `deletedAt`)** | When Admin removes a product, user, or category, `isDeleted = true` and `deletedAt = now()` is set. Admin still has an optional "Hard Delete" API for permanent cleanup. | **Relational Integrity Violations:** Prevents foreign key crashes in historical orders (`OrderItem` referencing deleted product). Users can still inspect old invoices accurately. |
| **bcrypt Password Hashing (Salt Rounds = 10)** | One-way salted hashing of 6-digit numeric passwords before persisting to PostgreSQL. | **Database Breach Password Exposure:** Even if database dumps are leaked, attacker cannot reverse-engineer user passwords. |
| **Helmet & CORS Security Headers** | Sets CSP, X-Frame-Options, HSTS, X-Content-Type-Options. Restricts API access exclusively to `https://furnixo.vercel.app` (and dev localhost). | **Clickjacking, MIME Sniffing, CSRF, Cross-Domain Data Theft:** Unauthorized 3rd-party websites cannot embed your app or make cross-origin fetch requests. |
| **Redis Caching for Carts & OTPs** | Active shopping carts and 4-digit OTPs are held in Redis with automatic TTL (Time To Live). | **Database Bottleneck & Slow Checkout:** Database isn't hammered every time a user increments an item in their cart. OTPs auto-expire cleanly after 5 minutes without cron jobs. |
| **Database Indexing (`@@index`)** | Indexes created on `phone`, `email`, `categoryId`, `orderStatus`, `userId`, `slug`. | **Slow Query Scans:** Prevents full table scans on PostgreSQL when millions of products or orders accumulate. |

---

## 4. Complete Database Schema (`prisma/schema.prisma`)

```prisma
// ==========================================
// Furnixo Database Schema (PostgreSQL + Prisma)
// ==========================================

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ================= Enums =================

enum Role {
  USER
  ADMIN
}

enum OrderStatus {
  PENDING
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELED
  RETURNED
}

enum PaymentMethod {
  COD
  ONLINE
}

enum PaymentStatus {
  PENDING
  PAID
  FAILED
  REFUNDED
}

enum CustomOrderStatus {
  PENDING             // Submitted by user, awaiting admin review
  QUOTATION_PROVIDED  // Admin set quoted price & estimated days
  ACCEPTED            // User approved price; placed as COD order
  REJECTED            // User declined quotation
  PROCESSING          // Under production in workshop
  DELIVERED           // Delivered to customer
  CANCELED            // Canceled by admin or user
}

enum AttributeType {
  FURNITURE_TYPE  // E.g., Table, Chair, Sofa, Bed, Wardrobe
  MATERIAL        // E.g., Shegun Wood, Chittagong Teak, MDF, Metal, Velvet
  COLOR           // E.g., Walnut Brown, Matte Black, Natural Wood, Off-White
  DESIGN_STYLE    // E.g., Modern Minimalist, Classic Royal, Industrial, Scandinavian
}

enum AddressType {
  HOME
  OFFICE
}

// ================= Models =================

model User {
  id            String         @id @default(uuid())
  phone         String         @unique // Valid BD format: 01XXXXXXXXX
  name          String?
  email         String?        @unique
  passwordHash  String
  role          Role           @default(USER)
  photoUrl      String?
  isVerified    Boolean        @default(false)
  isActive      Boolean        @default(true) // For admin block/unblock
  isDeleted     Boolean        @default(false)
  deletedAt     DateTime?

  addresses     Address[]
  orders        Order[]
  customOrders  CustomOrder[]
  reviews       Review[]
  wishlists     Wishlist[]

  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt

  @@index([phone])
  @@index([role])
}

model Address {
  id             String      @id @default(uuid())
  userId         String
  user           User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  type           AddressType @default(HOME)
  recipientName  String
  recipientPhone String
  city           String      // E.g., Dhaka, Chittagong
  thana          String      // E.g., Mirpur, Dhanmondi, Uttara
  localStreet    String      // House, Road, Block details
  postalCode     String?
  isDefault      Boolean     @default(false)

  orders         Order[]

  createdAt      DateTime    @default(now())
  updatedAt      DateTime    @updatedAt

  @@index([userId])
}

model Category {
  id            String         @id @default(uuid())
  name          String         @unique
  slug          String         @unique
  image         String?
  tag           String?        // E.g., "Trending", "Summer Collection"
  isActive      Boolean        @default(true)
  isFeatured    Boolean        @default(false)
  isDeleted     Boolean        @default(false)
  deletedAt     DateTime?

  subCategories SubCategory[]
  products      Product[]

  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt

  @@index([slug])
}

model SubCategory {
  id          String    @id @default(uuid())
  categoryId  String
  category    Category  @relation(fields: [categoryId], references: [id], onDelete: Cascade)
  name        String
  slug        String    @unique
  image       String?
  isActive    Boolean   @default(true)
  isDeleted   Boolean   @default(false)
  deletedAt   DateTime?

  products    Product[]

  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  @@index([categoryId])
  @@index([slug])
}

model Product {
  id            String         @id @default(uuid())
  categoryId    String
  category      Category       @relation(fields: [categoryId], references: [id])
  subCategoryId String?
  subCategory   SubCategory?   @relation(fields: [subCategoryId], references: [id])

  title         String
  slug          String         @unique
  subTitle      String?
  description   String         @db.Text
  price         Float
  discountPrice Float?
  brand         String?
  quantity      Int            @default(0) // Inventory stock count
  warranty      String?        // E.g., "5 Years Official Warranty"
  isFeatured    Boolean        @default(false)
  isActive      Boolean        @default(true)
  isDeleted     Boolean        @default(false)
  deletedAt     DateTime?

  images        ProductImage[]
  specs         ProductSpec[]
  orderItems    OrderItem[]
  reviews       Review[]
  wishlists     Wishlist[]

  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt

  @@index([categoryId])
  @@index([subCategoryId])
  @@index([slug])
  @@index([isFeatured, isActive, isDeleted])
}

model ProductImage {
  id        String   @id @default(uuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  url       String
  isPrimary Boolean  @default(false)
  sortOrder Int      @default(0)

  @@index([productId])
}

model ProductSpec {
  id        String   @id @default(uuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  key       String   // E.g., "Material", "Dimensions", "Finish", "Seating Capacity"
  value     String   // E.g., "Segun Wood", "6.5ft x 3ft", "High-Gloss Lacquer"

  @@index([productId])
}

model Order {
  id                String         @id @default(uuid())
  orderNumber       String         @unique // Formatted: FXO-2026-XXXX
  userId            String
  user              User           @relation(fields: [userId], references: [id])
  shippingAddressId String
  shippingAddress   Address        @relation(fields: [shippingAddressId], references: [id])

  orderStatus       OrderStatus    @default(PENDING)
  paymentMethod     PaymentMethod  @default(COD)
  paymentStatus     PaymentStatus  @default(PENDING)

  totalAmount       Float          // Subtotal of products
  discountAmount    Float          @default(0)
  deliveryCharge    Float          @default(0)
  finalAmount       Float          // totalAmount - discountAmount + deliveryCharge
  couponCode        String?

  adminNotes        String?        @db.Text
  customerNotes     String?        @db.Text

  orderItems        OrderItem[]

  createdAt         DateTime       @default(now())
  updatedAt         DateTime       @updatedAt

  @@index([userId])
  @@index([orderStatus])
  @@index([orderNumber])
}

model OrderItem {
  id           String   @id @default(uuid())
  orderId      String
  order        Order    @relation(fields: [orderId], references: [id], onDelete: Cascade)
  productId    String
  product      Product  @relation(fields: [productId], references: [id])
  productTitle String   // Stored historically in case product title changes
  unitPrice    Float    // Stored historically at time of purchase
  quantity     Int
  subtotal     Float

  @@index([orderId])
  @@index([productId])
}

// ==== Approach 2: Interactive Custom Furniture Quotation Builder ====
model CustomOrder {
  id                    String            @id @default(uuid())
  customOrderNumber     String            @unique // Formatted: FXC-2026-XXXX
  userId                String
  user                  User              @relation(fields: [userId], references: [id])

  furnitureType         String            // E.g., "Dining Table", "Executive Chair"
  material              String            // E.g., "Chittagong Teak", "Commercial Ply"
  color                 String            // E.g., "Walnut Polish", "Matte Black"
  designStyle           String            // E.g., "Scandinavian Minimalist"
  budgetRange           String            // E.g., "15,000 - 25,000 BDT"
  customSuggestion      String?           @db.Text
  referenceImages       String[]          // URLs of user-provided inspiration pictures

  status                CustomOrderStatus @default(PENDING)
  quotedPrice           Float?            // Provided by Admin during review
  adminNote             String?           @db.Text
  estimatedDeliveryDays Int?              // E.g., 14 days
  acceptedAt            DateTime?

  createdAt             DateTime          @default(now())
  updatedAt             DateTime          @updatedAt

  @@index([userId])
  @@index([status])
}

// Admin-manageable dynamic options for Custom Product Builder
model CustomProductAttribute {
  id            String        @id @default(uuid())
  type          AttributeType
  name          String        // Display name: e.g., "Teak Wood (Shegun)"
  value         String        // Value key: e.g., "shegun_wood"
  extraPriceEst Float?        @default(0) // Optional reference price indicator
  isActive      Boolean       @default(true)
  sortOrder     Int           @default(0)

  createdAt     DateTime      @default(now())
  updatedAt     DateTime      @updatedAt

  @@unique([type, value])
  @@index([type, isActive])
}

model Review {
  id         String   @id @default(uuid())
  userId     String
  user       User     @relation(fields: [userId], references: [id])
  productId  String
  product    Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  rating     Int      // 1 to 5
  comment    String   @db.Text
  images     String[] // User uploaded review images
  isApproved Boolean  @default(true) // Admin spam filtering flag

  createdAt  DateTime @default(now())
  updatedAt  DateTime @updatedAt

  @@index([productId, isApproved])
  @@index([userId])
}

model Wishlist {
  id        String   @id @default(uuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  productId String
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)

  createdAt DateTime @default(now())

  @@unique([userId, productId]) // One user can wishlist a product once
  @@index([userId])
}

model Coupon {
  id                String    @id @default(uuid())
  code              String    @unique // E.g., "EID2026", "FURNIXO10"
  discountPercent   Float     // E.g., 10.0 for 10%
  maxDiscountAmount Float?    // E.g., max 2000 BDT
  minSpend          Float     @default(0)
  validUntil        DateTime
  usageLimit        Int?      // Max total usages allowed
  usageCount        Int       @default(0)
  isActive          Boolean   @default(true)

  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt

  @@index([code, isActive])
}

model SiteSetting {
  id                   String   @id @default(uuid())
  siteName             String   @default("Furnixo")
  tagline              String?  @default("Modern Wooden Living")
  logoUrl              String?
  contactPhone         String?
  contactEmail         String?
  address              String?
  facebookLink         String?
  instagramLink        String?
  whatsappNumber       String?
  insideDhakaShipping  Float    @default(100.0)
  outsideDhakaShipping Float    @default(200.0)

  updatedAt            DateTime @updatedAt
}
```

---

## 5. Complete Zod Validation Schemas (`src/validations/`)

### 5.1 Auth Validation (`src/validations/auth.validation.ts`)
```typescript
import { z } from 'zod';

// Validates Bangladesh Mobile Numbers (e.g., 01712345678 or +8801712345678)
export const bdPhoneRegex = /^(?:\+8801|01)[3-9]\d{8}$/;

export const sendOtpSchema = z.object({
  body: z.object({
    phone: z.string().regex(bdPhoneRegex, "Invalid Bangladesh phone number. Format: 01XXXXXXXXX"),
  }),
});

export const verifyOtpSchema = z.object({
  body: z.object({
    phone: z.string().regex(bdPhoneRegex, "Invalid Bangladesh phone number"),
    otp: z.string().length(4, "OTP must be exactly 4 digits"),
  }),
});

export const registerSchema = z.object({
  body: z.object({
    phone: z.string().regex(bdPhoneRegex, "Invalid Bangladesh phone number"),
    otp: z.string().length(4, "OTP must be exactly 4 digits"),
    password: z.string().length(6, "Password must be exactly 6 digits"),
    name: z.string().min(2, "Name must be at least 2 characters").optional(),
  }),
});

export const loginSchema = z.object({
  body: z.object({
    phone: z.string().regex(bdPhoneRegex, "Invalid Bangladesh phone number"),
    password: z.string().length(6, "Password must be exactly 6 digits"),
  }),
});

export const changePasswordSchema = z.object({
  body: z.object({
    oldPassword: z.string().length(6, "Old password must be 6 digits"),
    newPassword: z.string().length(6, "New password must be 6 digits"),
  }),
});
```

### 5.2 Custom Order Validation (`src/validations/customOrder.validation.ts`)
```typescript
import { z } from 'zod';

export const createCustomOrderSchema = z.object({
  body: z.object({
    furnitureType: z.string().min(2, "Furniture type is required"),
    material: z.string().min(2, "Material selection is required"),
    color: z.string().min(2, "Color choice is required"),
    designStyle: z.string().min(2, "Design style is required"),
    budgetRange: z.string().min(2, "Budget range is required (e.g. 10000-20000 BDT)"),
    customSuggestion: z.string().max(1000, "Suggestion cannot exceed 1000 characters").optional(),
    referenceImages: z.array(z.string().url("Invalid image URL")).max(5, "Maximum 5 reference images allowed").optional().default([]),
  }),
});

export const adminQuoteCustomOrderSchema = z.object({
  params: z.object({
    id: z.string().uuid("Invalid custom order ID"),
  }),
  body: z.object({
    quotedPrice: z.number().positive("Quoted price must be greater than 0"),
    adminNote: z.string().min(2, "Admin note/clarification is required"),
    estimatedDeliveryDays: z.number().int().positive().optional().default(14),
  }),
});
```

### 5.3 Order & Checkout Validation (`src/validations/order.validation.ts`)
```typescript
import { z } from 'zod';

export const createOrderSchema = z.object({
  body: z.object({
    shippingAddressId: z.string().uuid("Valid shipping address ID is required"),
    couponCode: z.string().optional(),
    customerNotes: z.string().max(500).optional(),
    items: z.array(z.object({
      productId: z.string().uuid("Invalid product ID"),
      quantity: z.number().int().min(1, "Quantity must be at least 1"),
    })).min(1, "Order must contain at least one item"),
  }),
});

export const updateOrderStatusSchema = z.object({
  params: z.object({
    id: z.string().uuid("Invalid order ID"),
  }),
  body: z.object({
    orderStatus: z.enum(["PENDING", "PROCESSING", "SHIPPED", "DELIVERED", "CANCELED", "RETURNED"]).optional(),
    paymentStatus: z.enum(["PENDING", "PAID", "FAILED", "REFUNDED"]).optional(),
    adminNotes: z.string().optional(),
  }),
});
```

---

## 6. Complete API Design & Endpoint Specification

### 6.1 Authentication & Authorization Endpoints (`/api/auth`)

| Method | Endpoint | Access | Purpose & Flow |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/auth/send-otp` | Public (Rate-Limited) | Checks if phone exists. If new, generates 4-digit OTP, stores in Redis (5 min TTL), and triggers BulkSMSBD API. |
| `POST` | `/api/auth/verify-otp` | Public (Rate-Limited) | Verifies user's 4-digit OTP against Redis before completing registration form. |
| `POST` | `/api/auth/register` | Public (Rate-Limited) | Verifies OTP + creates User with hashed 6-digit password. Generates Access Token (7d) & Refresh Token (30d cookie). Auto-logs in. |
| `POST` | `/api/auth/login` | Public (Rate-Limited) | Validates phone & 6-digit password with bcrypt. Returns Access Token in JSON and sets Refresh Token in HttpOnly cookie. |
| `POST` | `/api/auth/refresh` | Public (Reads Cookie) | Reads HttpOnly `refreshToken`, verifies validity, and issues a fresh 7-day Access Token. |
| `POST` | `/api/auth/logout` | Authenticated | Clears the `refreshToken` HttpOnly cookie. |
| `POST` | `/api/auth/forgot-password/otp` | Public | Sends OTP for password reset to existing phone number. |
| `POST` | `/api/auth/forgot-password/reset` | Public | Verifies OTP and resets user password to a new 6-digit PIN. |

### 6.2 User Profile & Saved Addresses (`/api/users`)

| Method | Endpoint | Access | Purpose |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/users/profile` | USER / ADMIN | Returns currently authenticated user's details, role, phone, and addresses. |
| `PATCH`| `/api/users/profile` | USER / ADMIN | Updates name, email, profile photo URL. |
| `PATCH`| `/api/users/change-password` | USER / ADMIN | Validates old 6-digit password and sets new 6-digit password. |
| `GET` | `/api/users/addresses` | USER | Fetches all saved shipping addresses for user. |
| `POST` | `/api/users/addresses` | USER | Adds a new delivery address (recipientName, phone, city, thana, localStreet). |
| `PATCH`| `/api/users/addresses/:id/default`| USER | Sets an address as the default delivery location. |
| `DELETE`|`/api/users/addresses/:id`| USER | Deletes a saved address. |
| `GET` | `/api/users/wishlist` | USER | Retrieves user's wishlisted products. |
| `POST` | `/api/users/wishlist/:productId` | USER | Toggles product in wishlist (adds if absent, removes if present). |

### 6.3 Public Products & Catalog (`/api/products`)

| Method | Endpoint | Access | Purpose |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/products` | Public | Paginated product list with search, category filtering, min/max price, in-stock filter, and sorting (`newest`, `price-asc`, `price-desc`). Excludes `isDeleted = true`. |
| `GET` | `/api/products/featured` | Public | Fetches featured products for homepage carousel & visualizer. |
| `GET` | `/api/products/:slug` | Public | Returns single product with specifications, multiple images, and approved reviews. |
| `GET` | `/api/categories` | Public | Returns all active categories with their nested subcategories. |
| `GET` | `/api/settings` | Public | Returns site branding, logo URL, contact info, and delivery fees for checkout. |

### 6.4 Shopping Cart & Checkout (`/api/cart` & `/api/orders`)

| Method | Endpoint | Access | Purpose |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/cart` | USER | Retrieves active cart from Redis. |
| `POST` | `/api/cart/sync` | USER | Synchronizes local storage guest cart into Redis immediately upon user login. |
| `POST` | `/api/cart/items` | USER | Adds item or updates quantity in Redis cart. |
| `DELETE`| `/api/cart/items/:productId` | USER | Removes item from Redis cart. |
| `POST` | `/api/checkout/apply-coupon` | USER | Validates coupon code against cart amount; returns discount calculation. |
| `POST` | `/api/orders` | USER | Places Cash on Delivery (COD) order. Deducts stock inventory and clears Redis cart. |
| `GET` | `/api/orders/my-orders` | USER | Returns logged-in customer's order history with status timeline. |
| `GET` | `/api/orders/:id` | USER / ADMIN | Retrieves detailed order invoice and shipping address. |
| `POST` | `/api/orders/:id/review` | USER | Submits a review. Verifies that user actually purchased this product and order status is `DELIVERED`. |

### 6.5 Interactive Custom Design Builder (`/api/custom-orders`)

| Method | Endpoint | Access | Purpose & Approach 2 Flow |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/custom-orders/attributes` | Public | Returns dynamic configurator options (furniture types, wood materials, colors, styles) configured by Admin. |
| `POST` | `/api/custom-orders` | USER | Customer submits customized furniture request (status = `PENDING`). |
| `GET` | `/api/custom-orders/my-requests`| USER | Customer views their custom design requests, admin quotations, and notes. |
| `PATCH`| `/api/custom-orders/:id/accept` | USER | Customer accepts Admin's quoted price -> status becomes `ACCEPTED` -> triggers COD workshop order. |
| `PATCH`| `/api/custom-orders/:id/reject` | USER | Customer declines quoted price -> status becomes `REJECTED`. |

### 6.6 Admin Control Panel (`/api/admin`)

| Method | Endpoint | Access | Purpose |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/admin/dashboard` | ADMIN | Dashboard stats: Total revenue, total orders, pending orders, custom orders, active users. |
| `GET` | `/api/admin/users` | ADMIN | User list with filter by active/blocked. |
| `PATCH`| `/api/admin/users/:id/status` | ADMIN | Block/unblock a user account. |
| `POST` | `/api/admin/products` | ADMIN | Creates new product with images and specs. |
| `PATCH`| `/api/admin/products/:id` | ADMIN | Updates product details, prices, and stock inventory. |
| `DELETE`| `/api/admin/products/:id` | ADMIN | Soft delete (`isDeleted = true`) by default; hard delete query param supported (`?force=true`). |
| `GET` | `/api/admin/orders` | ADMIN | View all customer orders with filters (`status`, `dateRange`, `search`). |
| `PATCH`| `/api/admin/orders/:id/status` | ADMIN | Updates order status (`PROCESSING`, `SHIPPED`, `DELIVERED`, `CANCELED`). |
| `GET` | `/api/admin/custom-orders` | ADMIN | View all submitted custom furniture requests. |
| `PATCH`| `/api/admin/custom-orders/:id/quote`| ADMIN | Sets `quotedPrice`, `estimatedDeliveryDays`, and `adminNote`. Triggers BulkSMSBD SMS alert to customer! |
| `GET` | `/api/admin/reviews` | ADMIN | View pending reviews for spam moderation. |
| `PATCH`| `/api/admin/reviews/:id/approve` | ADMIN | Approve or reject customer review. |
| `POST` | `/api/admin/coupons` | ADMIN | Create new promotional discount coupon. |
| `PATCH`| `/api/admin/settings` | ADMIN | Update website logo, banner info, social links, and shipping costs. |
| `POST` | `/api/admin/custom-attributes` | ADMIN | Add new materials, wood types, colors to custom furniture configurator. |

---

## 7. Core Implementation Snippets (Copy-Paste Ready)

### 7.1 BulkSMSBD Integration Utility (`src/utils/sms.ts`)
```typescript
import axios from 'axios';

interface SendSmsParams {
  to: string;       // E.g., "017XXXXXXXX"
  message: string;  // Text message content
}

export const sendBulkSMS = async ({ to, message }: SendSmsParams): Promise<boolean> => {
  try {
    const apiKey = process.env.BULKSMSBD_API_KEY;
    const senderId = process.env.BULKSMSBD_SENDER_ID;
    const url = process.env.BULKSMSBD_URL || "http://bulksmsbd.net/api/smsapi";

    // Format phone number to standard 11-digit BD number
    let formattedPhone = to.replace(/^\+88/, "");

    const response = await axios.post(url, null, {
      params: {
        api_key: apiKey,
        type: "text",
        number: formattedPhone,
        senderid: senderId,
        message: message,
      },
      timeout: 8000,
    });

    if (response.data && response.data.response_code === 202) {
      return true;
    }
    console.error("BulkSMSBD Error Response:", response.data);
    return false;
  } catch (error) {
    console.error("Failed to dispatch SMS via BulkSMSBD:", error);
    return false;
  }
};
```

### 7.2 Standard API Response Wrapper (`src/utils/apiResponse.ts`)
```typescript
import { Response } from 'express';

export class ApiResponse {
  static success<T>(res: Response, message: string, data?: T, statusCode: number = 200) {
    return res.status(statusCode).json({
      success: true,
      message,
      data: data ?? null,
    });
  }

  static error(res: Response, message: string, error?: any, statusCode: number = 400) {
    return res.status(statusCode).json({
      success: false,
      message,
      error: error ?? null,
    });
  }
}
```

### 7.3 JWT Token Utility with HttpOnly Cookie Helper (`src/utils/jwt.ts`)
```typescript
import jwt from 'jsonwebtoken';
import { Response } from 'express';

interface TokenPayload {
  userId: string;
  role: 'ADMIN' | 'USER';
}

const ACCESS_SECRET = process.env.JWT_ACCESS_SECRET || 'furnixo_super_access_secret_2026';
const REFRESH_SECRET = process.env.JWT_REFRESH_SECRET || 'furnixo_super_refresh_secret_2026';

export const generateTokens = (payload: TokenPayload) => {
  const accessToken = jwt.sign(payload, ACCESS_SECRET, { expiresIn: '7d' });
  const refreshToken = jwt.sign(payload, REFRESH_SECRET, { expiresIn: '30d' });
  return { accessToken, refreshToken };
};

export const setRefreshTokenCookie = (res: Response, refreshToken: string) => {
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,                                  // Inaccessible to browser JS (Protects against XSS)
    secure: process.env.NODE_ENV === 'production',   // HTTPS only in production
    sameSite: 'strict',                              // Protects against CSRF
    maxAge: 30 * 24 * 60 * 60 * 1000,                // 30 Days
  });
};
```

### 7.4 Authentication & Role Guard Middlewares (`src/middlewares/auth.middleware.ts`)
```typescript
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { ApiResponse } from '../utils/apiResponse';

export interface AuthenticatedUser {
  userId: string;
  role: 'ADMIN' | 'USER';
}

declare global {
  namespace Express {
    interface Request {
      user?: AuthenticatedUser;
    }
  }
}

export const authenticateJWT = (req: Request, res: Response, next: NextFunction) => {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return ApiResponse.error(res, "Access denied. Bearer token missing.", null, 401);
  }

  const token = authHeader.split(' ')[1];

  try {
    const decoded = jwt.verify(token, process.env.JWT_ACCESS_SECRET || 'furnixo_super_access_secret_2026') as AuthenticatedUser;
    req.user = decoded;
    next();
  } catch (err) {
    return ApiResponse.error(res, "Invalid or expired access token.", null, 403);
  }
};

export const requireAdmin = (req: Request, res: Response, next: NextFunction) => {
  if (!req.user || req.user.role !== 'ADMIN') {
    return ApiResponse.error(res, "Forbidden: Admin privileges required.", null, 403);
  }
  next();
};
```

### 7.5 Generic Zod Validation Middleware (`src/middlewares/validate.middleware.ts`)
```typescript
import { Request, Response, NextFunction } from 'express';
import { AnyZodObject, ZodError } from 'zod';
import { ApiResponse } from '../utils/apiResponse';

export const validateRequest = (schema: AnyZodObject) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params,
      });
      next();
    } catch (error) {
      if (error instanceof ZodError) {
        const formattedErrors = error.errors.map((err) => ({
          field: err.path.join('.').replace(/^(body|query|params)\./, ''),
          message: err.message,
        }));
        return ApiResponse.error(res, "Validation failed", formattedErrors, 422);
      }
      return ApiResponse.error(res, "Internal validation error", null, 500);
    }
  };
};
```

---

## 8. Perspective: "Will AI Take Everyone's Jobs, and How Will AI Companies Survive?"

You raised a profound economic and philosophical question:
> *"amar kotha holo ai jodi sobar job nia nei tahole to tara ar ai use kore kaj korebena ahole ai company gula income korbe kivabe ar cholbe ki kore"*

Here is the objective economic and engineering reality:

1. **The Fallacy of Total Displacement (Economic Feedback Loop)**:
   In macroeconomic theory, if AI were to displace all workers and leave the population without income, purchasing power would collapse to zero. No consumer would have money to buy furniture from Furnixo, buy mobile phones, or purchase $20/month AI subscriptions. Consequently, OpenAI, Google, Anthropic, and tech giants would instantly bankrupt themselves. Market economies do not permit a one-sided producer surplus without consumer demand.

2. **The Shift in Abstraction, Not Elimination**:
   - 60 years ago, computer programming required punch cards and assembly code. When high-level languages like C and Java arrived, people said: *"Compilers will eliminate all programmer jobs!"* Instead, the software industry exploded from thousands of programmers to over 30 million engineers worldwide.
   - AI is doing the exact same thing: it is **raising the abstraction layer**.
   - Instead of writing boilerplate CRUD and regex by hand for 3 weeks, you build the entire Furnixo platform in 2 days. 

3. **From "Code Typist" to "Software Architect & Business Owner"**:
   - In the past, building a custom e-commerce system with custom quotation builders, OTP SMS verification, Redis caches, and inventory tracking required a dedicated team of 8 engineers costing 3-5 lakh BDT per month.
   - Today, **you** act as the Lead Architect, Product Director, and Founder. AI works as your hyper-efficient junior and mid-level developer ("Copilot"), allowing you to compete directly with multimillion-dollar corporations like Otobi, Regal, or Hatil.
   - The value is shifting from *“Who can remember syntax?”* to *“Who understands business logic, market needs, customer psychology, and system architecture?”*

---

## 9. Dedicated TypeScript Types & Interfaces (`src/types/`)

To guarantee strict separation of concerns, no type or interface is defined inline inside controllers or services. All contracts live in dedicated domain files under `src/types/`.

### 9.1 Common & Query Types (`src/types/query.types.ts`)
```typescript
export interface PaginationMeta {
  page: number;
  limit: number;
  total: number;
  totalPages: number;
  hasNextPage: boolean;
  hasPrevPage: boolean;
}

export interface PaginatedResult<T> {
  meta: PaginationMeta;
  data: T[];
}

export interface QueryParams {
  page?: string | number;
  limit?: string | number;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
  search?: string;
  minPrice?: string | number;
  maxPrice?: string | number;
  [key: string]: any;
}

export interface QueryBuilderOptions<TModel, TWhereInput, TOrderByInput> {
  searchableFields?: string[];
  filterableFields?: string[];
  rangeFields?: {
    field: string;
    minKey?: string;
    maxKey?: string;
  }[];
  defaultSortBy?: string;
  defaultSortOrder?: 'asc' | 'desc';
  include?: any;
  select?: any;
  baseWhere?: TWhereInput;
}
```

### 9.2 API Response Types (`src/types/common.types.ts`)
```typescript
import { PaginationMeta } from './query.types';

export type UserRole = 'ADMIN' | 'USER';

export interface AuthenticatedUser {
  userId: string;
  role: UserRole;
  phone?: string;
}

export interface ApiResponseFormat<T = any> {
  success: boolean;
  message: string;
  meta?: PaginationMeta;
  data?: T | null;
  error?: any;
}
```

### 9.3 Product Domain Types (`src/types/product.types.ts`)
```typescript
import { QueryParams } from './query.types';

export interface ProductFilterParams extends QueryParams {
  categoryId?: string;
  subCategoryId?: string;
  brand?: string;
  isFeatured?: boolean | string;
  inStock?: boolean | string;
}

export interface CreateProductDTO {
  categoryId: string;
  subCategoryId?: string;
  title: string;
  subTitle?: string;
  description: string;
  price: number;
  discountPrice?: number;
  brand?: string;
  quantity: number;
  warranty?: string;
  isFeatured?: boolean;
  images: { url: string; isPrimary?: boolean; sortOrder?: number }[];
  specs?: { key: string; value: string }[];
}

export interface UpdateProductDTO extends Partial<CreateProductDTO> {
  isActive?: boolean;
}
```

### 9.4 Order Domain Types (`src/types/order.types.ts`)
```typescript
import { OrderStatus, PaymentMethod, PaymentStatus } from '@prisma/client';
import { QueryParams } from './query.types';

export interface CreateOrderItemDTO {
  productId: string;
  quantity: number;
}

export interface CreateOrderDTO {
  shippingAddressId: string;
  couponCode?: string;
  customerNotes?: string;
  items: CreateOrderItemDTO[];
}

export interface UpdateOrderStatusDTO {
  orderStatus?: OrderStatus;
  paymentStatus?: PaymentStatus;
  adminNotes?: string;
}

export interface OrderFilterParams extends QueryParams {
  orderStatus?: OrderStatus;
  paymentStatus?: PaymentStatus;
  userId?: string;
  startDate?: string;
  endDate?: string;
}
```

### 9.5 Custom Design Order Types (`src/types/customOrder.types.ts`)
```typescript
import { CustomOrderStatus } from '@prisma/client';
import { QueryParams } from './query.types';

export interface CreateCustomOrderDTO {
  furnitureType: string;
  material: string;
  color: string;
  designStyle: string;
  budgetRange: string;
  customSuggestion?: string;
  referenceImages?: string[];
}

export interface AdminQuoteDTO {
  quotedPrice: number;
  adminNote: string;
  estimatedDeliveryDays?: number;
}

export interface CustomOrderFilterParams extends QueryParams {
  status?: CustomOrderStatus;
  userId?: string;
}
```

---

## 10. Global Reusable QueryBuilder with Metadata Pagination (`src/helpers/queryBuilder.ts`)

This is the unified database query engine. It dynamically converts Express URL query parameters (`?search=sofa&minPrice=5000&maxPrice=25000&sortBy=price&sortOrder=asc&page=1&limit=12`) into type-safe Prisma `where`, `orderBy`, `skip`, and `take` clauses, then computes and returns complete pagination metadata.

```typescript
import { PaginationMeta, PaginatedResult, QueryParams, QueryBuilderOptions } from '../types/query.types';

export class QueryBuilder<TModel, TWhereInput = any, TOrderByInput = any> {
  private modelDelegate: any;
  private query: QueryParams;
  private options: QueryBuilderOptions<TModel, TWhereInput, TOrderByInput>;

  constructor(
    modelDelegate: any,
    query: QueryParams,
    options: QueryBuilderOptions<TModel, TWhereInput, TOrderByInput> = {}
  ) {
    this.modelDelegate = modelDelegate;
    this.query = query;
    this.options = options;
  }

  // 1. Build where clause: Search + Exact Filters + Numeric Ranges + Base Filters
  private buildWhereClause(): any {
    const where: any = { ...(this.options.baseWhere || {}) };

    // Soft delete guard (if model supports it)
    if (where.isDeleted === undefined) {
      where.isDeleted = false;
    }

    // A. Multi-field Keyword Search
    const search = this.query.search?.toString().trim();
    if (search && this.options.searchableFields && this.options.searchableFields.length > 0) {
      where.OR = this.options.searchableFields.map((field) => ({
        [field]: {
          contains: search,
          mode: 'insensitive',
        },
      }));
    }

    // B. Explicit Filterable Fields (Category, Brand, Status, etc.)
    if (this.options.filterableFields) {
      for (const field of this.options.filterableFields) {
        if (this.query[field] !== undefined && this.query[field] !== '') {
          const value = this.query[field];

          // Boolean string parsing
          if (value === 'true' || value === true) {
            where[field] = true;
          } else if (value === 'false' || value === false) {
            where[field] = false;
          } else {
            where[field] = value;
          }
        }
      }
    }

    // C. Range Filters (e.g., minPrice / maxPrice, startDate / endDate)
    if (this.options.rangeFields) {
      for (const range of this.options.rangeFields) {
        const minVal = this.query[range.minKey || `min${range.field.charAt(0).toUpperCase() + range.field.slice(1)}`];
        const maxVal = this.query[range.maxKey || `max${range.field.charAt(0).toUpperCase() + range.field.slice(1)}`];

        if (minVal !== undefined || maxVal !== undefined) {
          where[range.field] = {};
          if (minVal !== undefined && minVal !== '') {
            where[range.field].gte = Number(minVal);
          }
          if (maxVal !== undefined && maxVal !== '') {
            where[range.field].lte = Number(maxVal);
          }
        }
      }
    }

    return where;
  }

  // 2. Build order by clause
  private buildOrderByClause(): any {
    const sortBy = (this.query.sortBy as string) || this.options.defaultSortBy || 'createdAt';
    const sortOrder = (this.query.sortOrder as 'asc' | 'desc') || this.options.defaultSortOrder || 'desc';

    return {
      [sortBy]: sortOrder,
    };
  }

  // 3. Build pagination (skip / take)
  private buildPagination(): { page: number; limit: number; skip: number; take: number } {
    const page = Math.max(1, Number(this.query.page) || 1);
    const limit = Math.min(100, Math.max(1, Number(this.query.limit) || 10)); // Default 10, max 100
    const skip = (page - 1) * limit;

    return { page, limit, skip, take: limit };
  }

  // 4. Execute query with parallel total count
  public async execute(): Promise<PaginatedResult<TModel>> {
    const where = this.buildWhereClause();
    const orderBy = this.buildOrderByClause();
    const { page, limit, skip, take } = this.buildPagination();

    const findManyArgs: any = {
      where,
      orderBy,
      skip,
      take,
    };

    if (this.options.include) {
      findManyArgs.include = this.options.include;
    } else if (this.options.select) {
      findManyArgs.select = this.options.select;
    }

    // Execute data fetch and count in parallel for maximum PostgreSQL throughput
    const [data, total] = await Promise.all([
      this.modelDelegate.findMany(findManyArgs),
      this.modelDelegate.count({ where }),
    ]);

    const totalPages = Math.ceil(total / limit);

    const meta: PaginationMeta = {
      page,
      limit,
      total,
      totalPages,
      hasNextPage: page < totalPages,
      hasPrevPage: page > 1,
    };

    return { meta, data };
  }
}
```

---

## 11. Strict Separation in Practice: Controller vs. Service

### Rule of Architecture
- **Controller:** ONLY parses HTTP `req` (`body`, `query`, `params`, `user`), triggers services, and calls `ApiResponse`. Never writes SQL/Prisma queries. Never touches database directly.
- **Service:** ONLY executes business logic, database transactions, calculations, and external APIs. Never touches `req` or `res`. Never sends HTTP status codes.

### 11.1 Product Service (`src/services/product.service.ts`)
```typescript
import { prisma } from '../config/prisma';
import { QueryBuilder } from '../helpers/queryBuilder';
import { ProductFilterParams, CreateProductDTO } from '../types/product.types';
import { PaginatedResult } from '../types/query.types';
import { slugify } from '../utils/slugify';

export class ProductService {
  // Pure business logic: fetches paginated products using generic QueryBuilder
  static async getAllProducts(query: ProductFilterParams): Promise<PaginatedResult<any>> {
    const queryBuilder = new QueryBuilder(prisma.product, query, {
      searchableFields: ['title', 'description', 'brand', 'subTitle'],
      filterableFields: ['categoryId', 'subCategoryId', 'brand', 'isFeatured', 'isActive'],
      rangeFields: [{ field: 'price', minKey: 'minPrice', maxKey: 'maxPrice' }],
      defaultSortBy: 'createdAt',
      defaultSortOrder: 'desc',
      include: {
        category: { select: { id: true, name: true, slug: true } },
        images: { orderBy: { sortOrder: 'asc' } },
      },
    });

    return await queryBuilder.execute();
  }

  // Pure business logic: creates product with images and specs in a transaction
  static async createProduct(payload: CreateProductDTO) {
    const baseSlug = slugify(payload.title);
    let uniqueSlug = baseSlug;
    let counter = 1;

    // Ensure unique slug
    while (await prisma.product.findUnique({ where: { slug: uniqueSlug } })) {
      uniqueSlug = `${baseSlug}-${counter}`;
      counter++;
    }

    return await prisma.product.create({
      data: {
        title: payload.title,
        slug: uniqueSlug,
        subTitle: payload.subTitle,
        description: payload.description,
        price: payload.price,
        discountPrice: payload.discountPrice,
        brand: payload.brand,
        quantity: payload.quantity,
        warranty: payload.warranty,
        isFeatured: payload.isFeatured ?? false,
        categoryId: payload.categoryId,
        subCategoryId: payload.subCategoryId,
        images: {
          create: payload.images.map((img, idx) => ({
            url: img.url,
            isPrimary: img.isPrimary ?? idx === 0,
            sortOrder: img.sortOrder ?? idx,
          })),
        },
        specs: payload.specs
          ? {
              create: payload.specs.map((s) => ({
                key: s.key,
                value: s.value,
              })),
            }
          : undefined,
      },
      include: {
        images: true,
        specs: true,
      },
    });
  }
}
```

### 11.2 Product Controller (`src/controllers/product.controller.ts`)
```typescript
import { Request, Response } from 'express';
import { ProductService } from '../services/product.service';
import { ApiResponse } from '../utils/apiResponse';

export class ProductController {
  // Controller ONLY receives req, delegates to Service, and formats JSON
  static async getProducts(req: Request, res: Response) {
    try {
      const result = await ProductService.getAllProducts(req.query);
      return ApiResponse.success(res, "Products fetched successfully", result.data, 200, result.meta);
    } catch (error: any) {
      return ApiResponse.error(res, error.message || "Failed to fetch products", error, 500);
    }
  }

  static async createProduct(req: Request, res: Response) {
    try {
      const newProduct = await ProductService.createProduct(req.body);
      return ApiResponse.success(res, "Product created successfully", newProduct, 201);
    } catch (error: any) {
      return ApiResponse.error(res, error.message || "Failed to create product", error, 400);
    }
  }
}
```

### 11.3 Order Checkout with Inventory Transaction (`src/services/order.service.ts`)
```typescript
import { prisma } from '../config/prisma';
import { redis } from '../config/redis';
import { CreateOrderDTO } from '../types/order.types';
import { generateOrderNumber } from '../helpers/orderNumberGenerator';

export class OrderService {
  static async createOrder(userId: string, payload: CreateOrderDTO) {
    // Execute database transaction to guarantee ACID inventory management
    return await prisma.$transaction(async (tx) => {
      let totalAmount = 0;
      const orderItemsData: any[] = [];

      // 1. Validate stock availability and calculate price
      for (const item of payload.items) {
        const product = await tx.product.findUnique({
          where: { id: item.productId },
        });

        if (!product || product.isDeleted || !product.isActive) {
          throw new Error(`Product not found or unavailable: ${item.productId}`);
        }

        if (product.quantity < item.quantity) {
          throw new Error(`Insufficient stock for "${product.title}". In stock: ${product.quantity}`);
        }

        const unitPrice = product.discountPrice ?? product.price;
        const subtotal = unitPrice * item.quantity;
        totalAmount += subtotal;

        orderItemsData.push({
          productId: product.id,
          productTitle: product.title,
          unitPrice,
          quantity: item.quantity,
          subtotal,
        });

        // Deduct inventory stock
        await tx.product.update({
          where: { id: product.id },
          data: { quantity: { decrement: item.quantity } },
        });
      }

      // 2. Fetch shipping settings for delivery charges
      const settings = await tx.siteSetting.findFirst();
      const shippingAddress = await tx.address.findUnique({
        where: { id: payload.shippingAddressId },
      });

      if (!shippingAddress) {
        throw new Error("Shipping address not found");
      }

      // Inside Dhaka vs Outside Dhaka
      const isDhaka = shippingAddress.city.toLowerCase().includes('dhaka');
      const deliveryCharge = isDhaka
        ? (settings?.insideDhakaShipping ?? 100)
        : (settings?.outsideDhakaShipping ?? 200);

      // 3. Apply Coupon if provided
      let discountAmount = 0;
      if (payload.couponCode) {
        const coupon = await tx.coupon.findUnique({
          where: { code: payload.couponCode.toUpperCase() },
        });

        if (coupon && coupon.isActive && coupon.validUntil > new Date() && totalAmount >= coupon.minSpend) {
          discountAmount = (totalAmount * coupon.discountPercent) / 100;
          if (coupon.maxDiscountAmount && discountAmount > coupon.maxDiscountAmount) {
            discountAmount = coupon.maxDiscountAmount;
          }

          // Increment coupon usage
          await tx.coupon.update({
            where: { id: coupon.id },
            data: { usageCount: { increment: 1 } },
          });
        }
      }

      const finalAmount = totalAmount - discountAmount + deliveryCharge;
      const orderNumber = await generateOrderNumber(tx);

      // 4. Create Order Record
      const order = await tx.order.create({
        data: {
          orderNumber,
          userId,
          shippingAddressId: payload.shippingAddressId,
          orderStatus: 'PENDING',
          paymentMethod: 'COD',
          paymentStatus: 'PENDING',
          totalAmount,
          discountAmount,
          deliveryCharge,
          finalAmount,
          couponCode: payload.couponCode,
          customerNotes: payload.customerNotes,
          orderItems: {
            create: orderItemsData,
          },
        },
        include: {
          orderItems: true,
          shippingAddress: true,
        },
      });

      // 5. Clear User Redis Cart after successful order placement
      await redis.del(`cart:${userId}`);

      return order;
    });
  }
}
```

### 11.4 Updated API Response Wrapper Supporting Metadata (`src/utils/apiResponse.ts`)
```typescript
import { Response } from 'express';
import { PaginationMeta } from '../types/query.types';

export class ApiResponse {
  static success<T>(
    res: Response,
    message: string,
    data?: T,
    statusCode: number = 200,
    meta?: PaginationMeta
  ) {
    return res.status(statusCode).json({
      success: true,
      message,
      meta: meta ?? undefined,
      data: data ?? null,
    });
  }

  static error(res: Response, message: string, error?: any, statusCode: number = 400) {
    return res.status(statusCode).json({
      success: false,
      message,
      error: error ?? null,
    });
  }
}
```

---

## 12. Environment Variables, PostgreSQL Connection & Frontend-Backend Integration

### 12.1 Environment Variables Configuration

#### Backend `.env` (`furnixo-backend/.env`)
```env
# Server Runtime
PORT=5000
NODE_ENV=development

# PostgreSQL Connection String (Prisma)
# Local: postgresql://postgres:password123@localhost:5432/furnixo_db?schema=public
# Cloud (Neon / Supabase): postgresql://furnixo_owner:secret@ep-cool-project.neon.tech/furnixo_db?sslmode=require
DATABASE_URL="postgresql://postgres:password123@localhost:5432/furnixo_db?schema=public"

# Redis Connection (Local or Upstash Cloud)
REDIS_URL="redis://localhost:6379"

# Security & JWT Tokens (Must be at least 32 characters long in production)
JWT_ACCESS_SECRET="furnixo_super_secure_access_token_secret_key_2026_xyz"
JWT_REFRESH_SECRET="furnixo_super_secure_refresh_token_secret_key_2026_xyz"
JWT_ACCESS_EXPIRY="7d"
JWT_REFRESH_EXPIRY="30d"

# BulkSMSBD SMS Gateway Credentials
BULKSMSBD_API_KEY="your_bulksmsbd_api_key_here"
BULKSMSBD_SENDER_ID="your_approved_sender_id"
BULKSMSBD_URL="http://bulksmsbd.net/api/smsapi"

# CORS & Cookie Domain
FRONTEND_URL="http://localhost:3000"
PROD_FRONTEND_URL="https://furnixo.vercel.app"
COOKIE_SECRET="furnixo_cookie_signing_secret_key_2026"
```

#### Frontend `.env.local` (`Furnixo/.env.local`)
```env
# Backend API Base URL
NEXT_PUBLIC_API_URL="http://localhost:5000/api"

# Public Site Domain
NEXT_PUBLIC_SITE_URL="http://localhost:3000"
```

---

### 12.2 PostgreSQL Database Connection Flow

```
+--------------------------+                 +---------------------------+                 +--------------------------+
|      Prisma Schema       |                 |       Prisma Client       |                 |   PostgreSQL Database    |
| (prisma/schema.prisma)   | ─── db push ──> | (Singleton Connection)    | ─── SQL TCP ──> | (Local:5432 or Cloud:    |
|  - Models, Relations     |                 | (src/config/prisma.ts)    |                 |  Neon / Supabase / AWS)  |
+--------------------------+                 +---------------------------+                 +--------------------------+
```

#### 1. Connection String Structure
`postgresql://[USER]:[PASSWORD]@[HOST]:[PORT]/[DATABASE_NAME]?schema=public&sslmode=prefer`
- **USER**: Your PostgreSQL user (default: `postgres`).
- **PASSWORD**: Database password.
- **HOST**: `localhost` (for local PC) or cloud hostname (e.g., `ep-xyz.neon.tech`).
- **PORT**: Default is `5432`.
- **DATABASE_NAME**: `furnixo_db`.

#### 2. Singleton Prisma Client (`src/config/prisma.ts`)
*Prevents connection pool exhaustion in Node.js development during hot-reloading:*
```typescript
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

#### 3. Database Commands
```bash
# 1. Generate TypeScript Prisma Client
npx prisma generate

# 2. Push schema changes directly to PostgreSQL (Great for rapid prototyping)
npx prisma db push

# 3. Create production migration files (Safe schema tracking)
npx prisma migrate dev --name init

# 4. Open Prisma Studio (Visual GUI in browser at localhost:5555 to view/edit database records)
npx prisma studio
```

---

### 12.3 Express CORS & Cookie Settings for Secure Next.js Connection

For Next.js to communicate with Express and receive the `HttpOnly` refresh token cookie:
1. `credentials: true` must be enabled on Express CORS.
2. `origin` must strictly match the Next.js port (`http://localhost:3000` or `https://furnixo.vercel.app`).

#### `src/app.ts` (Backend CORS Setup)
```typescript
import express from 'express';
import cors from 'cors';
import cookieParser from 'cookie-parser';
import helmet from 'helmet';

const app = express();

const allowedOrigins = [
  process.env.FRONTEND_URL || 'http://localhost:3000',
  process.env.PROD_FRONTEND_URL || 'https://furnixo.vercel.app',
];

app.use(helmet());
app.use(cookieParser(process.env.COOKIE_SECRET));
app.use(
  cors({
    origin: (origin, callback) => {
      // Allow requests with no origin (like mobile apps, curl, server-side fetch)
      if (!origin || allowedOrigins.includes(origin)) {
        callback(null, true);
      } else {
        callback(new Error('Blocked by CORS policy'));
      }
    },
    credentials: true, // CRITICAL: Allows browser to send & receive HttpOnly cookies!
  })
);

app.use(express.json());
```

---

### 12.4 Next.js Frontend Integration & Rendering Strategy

Next.js App Router provides two distinct rendering environments:

```
                                    Next.js App Router
                                            │
               ┌────────────────────────────┴────────────────────────────┐
               ▼                                                         ▼
     [Server Components (RSC)]                                 [Client Components ("use client")]
  • Run on server at request/build time                     • Run in browser with React state
  • Best for: SEO, Initial Page Load, Product PDP/PLP       • Best for: Cart, Auth, Configurator, Checkout
  • Fast First Contentful Paint (FCP)                       • Instant interactivity & Optimistic UI
  • Uses native fetch() with Next.js Cache & ISR            • Uses Axios Client with Interceptors & TanStack Query
```

#### 1. Server Component: SEO-Optimized Product Listing (`src/app/shop/page.tsx`)
```typescript
import { Product } from '@/types/product';

interface ShopPageProps {
  searchParams: Promise<{ [key: string]: string | undefined }>;
}

export default async function ShopPage({ searchParams }: ShopPageProps) {
  const params = await searchParams;
  const queryString = new URLSearchParams(params as Record<string, string>).toString();

  // Fetches directly from backend on Next.js server with 60s Incremental Static Regeneration (ISR)
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/products?${queryString}`, {
    next: { revalidate: 60 },
  });

  const responseData = await res.json();
  const products: Product[] = responseData.data || [];
  const meta = responseData.meta;

  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-6">Furniture Catalog</h1>
      <p className="text-gray-500 mb-4">Showing {meta?.total || 0} products</p>
      <div className="grid grid-cols-1 md:grid-cols-3 lg:grid-cols-4 gap-6">
        {products.map((product) => (
          <div key={product.id} className="border rounded-lg p-4">
            <h2 className="font-semibold">{product.title}</h2>
            <p className="text-amber-600 font-bold">BDT {product.price}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

#### 2. Centralized Frontend API Client with Silent Token Refresh (`src/lib/api-client.ts`)
*Client Components use this Axios instance to make authenticated requests. It automatically handles token expiration and retries requests silently:*

```typescript
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || 'http://localhost:5000/api',
  withCredentials: true, // CRITICAL: Sends HttpOnly refresh token cookie automatically
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request Interceptor: Attach Access Token from memory/localStorage
apiClient.interceptors.request.use((config) => {
  if (typeof window !== 'undefined') {
    const token = localStorage.getItem('furnixo_access_token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
  }
  return config;
});

// Response Interceptor: Silent Token Refresh on 401 Unauthorized
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    // If 401 Unauthorized and request hasn't been retried yet
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        // Call refresh token API (Browser automatically includes HttpOnly refreshToken cookie)
        const refreshResponse = await axios.post(
          `${process.env.NEXT_PUBLIC_API_URL || 'http://localhost:5000/api'}/auth/refresh`,
          {},
          { withCredentials: true }
        );

        const newAccessToken = refreshResponse.data.data.accessToken;

        // Save new access token
        localStorage.setItem('furnixo_access_token', newAccessToken);

        // Retry original request with new token
        originalRequest.headers.Authorization = `Bearer ${newAccessToken}`;
        return apiClient(originalRequest);
      } catch (refreshError) {
        // If refresh fails, user must log in again
        localStorage.removeItem('furnixo_access_token');
        if (typeof window !== 'undefined') {
          window.location.href = '/login';
        }
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);
```

---

## 13. Image Management (Cloudinary + Multer) & High-Value Packages

### 13.1 Image Upload Architecture: Why Cloudinary is Ideal for Furnixo

Furnixo is a furniture e-commerce platform where high-resolution wood grain, fabric textures, and lifestyle room visuals are critical for conversions. Cloudinary provides:
1. **Automatic Format Optimization (`f_auto`):** Automatically converts PNG/JPG to modern lightweight WebP or AVIF based on the customer's browser.
2. **Dynamic Quality Compression (`q_auto`):** Reduces image file size by up to 70% with zero visible quality loss.
3. **Dynamic Transformations (URLs):**
   - Thumbnail: `https://res.cloudinary.com/.../w_300,h_300,c_fill/...`
   - Hero Zoom: `https://res.cloudinary.com/.../w_1200,q_auto,f_auto/...`
4. **Generous Free Tier:** 25 Monthly Credits (~25 GB storage or 25,000 transformations), ideal for launching without cloud infrastructure costs.

#### 1. Multer Memory Storage Configuration (`src/middlewares/upload.middleware.ts`)
```typescript
import multer from 'multer';

// Use memoryStorage so uploaded buffer stays in RAM without touching server disk
const storage = multer.memoryStorage();

const fileFilter = (req: any, file: Express.Multer.File, cb: multer.FileFilterCallback) => {
  if (file.mimetype.startsWith('image/')) {
    cb(null, true);
  } else {
    cb(new Error('Only image files (jpg, jpeg, png, webp) are permitted!'));
  }
};

export const upload = multer({
  storage,
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB maximum file size
  },
  fileFilter,
});
```

#### 2. Cloudinary Upload Utility (`src/utils/cloudinary.ts`)
```typescript
import { v2 as cloudinary, UploadApiResponse } from 'cloudinary';
import streamifier from 'streamifier';

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET,
});

export class CloudinaryService {
  // Uploads image buffer from Multer memory to Cloudinary folder
  static async uploadBuffer(
    fileBuffer: Buffer,
    folder: string = 'furnixo/products'
  ): Promise<UploadApiResponse> {
    return new Promise((resolve, reject) => {
      const uploadStream = cloudinary.uploader.upload_stream(
        {
          folder,
          resource_type: 'image',
          transformation: [{ quality: 'auto', fetch_format: 'auto' }],
        },
        (error, result) => {
          if (error) return reject(error);
          resolve(result as UploadApiResponse);
        }
      );

      streamifier.createReadStream(fileBuffer).pipe(uploadStream);
    });
  }

  // Delete image by public ID if product is permanently purged
  static async deleteImage(publicId: string): Promise<any> {
    return await cloudinary.uploader.destroy(publicId);
  }
}
```

---

### 13.2 Recommended Production & Developer Packages

| Category | Package | What It Does & Why It's Valuable |
| :--- | :--- | :--- |
| **Response Compression** | `compression` | Compresses Express JSON responses using Gzip / Brotli before sending to client. Reduces 100KB product lists to ~15KB (85% bandwidth reduction). |
| **Production Logging** | `winston` + `morgan` | Standardizes HTTP request logs with execution times and stores daily error logs in `logs/error.log` instead of disappearing `console.log` statements. |
| **Date & Time Calculations** | `dayjs` | Ultra-lightweight (2KB) date manipulation. Perfect for coupon validity checking, estimated delivery date calculations, and invoice timestamps. |
| **Automated Invoicing** | `pdfkit` | Generates official PDF invoice receipts on demand when orders transition to `DELIVERED`, allowing customers to download PDF invoices from their dashboard. |
| **Fast Dev Environment** | `tsx` | Executes TypeScript files directly at near-native speed using esbuild without complex build steps or slow nodemon+ts-node restarts. |
| **Database Mocking** | `@faker-js/faker` | Used in `prisma/seed.ts` to generate realistic dummy categories, furniture specs, prices, and customer reviews during initial development. |
| **Parameter Pollution Defense** | `hpp` | Prevents HTTP Parameter Pollution attacks (e.g., passing `?price=10&price=20` to bypass filtering logic). |

---

## 14. Global Enterprise Error Handling & Dual-Persona Diagnostic Engine

### 14.1 Philosophy & Information Security Rules

1. **Zero Information Leakage (Public / User Security):**
   - Regular customers must **NEVER** see raw database schema details, PostgreSQL constraint names, SQL queries, file system paths, or raw stack traces.
   - Leaking stack traces or database errors exposes system internals to attackers looking for SQL/logic vulnerabilities.
   - User messages must be **concise, polite, and actionable** (e.g., *"This phone number is already registered. Please log in."*).

2. **Admin-Tailored Diagnostic Clarity (Admin Productivity):**
   - When an error occurs during an Admin action (or for requests where `req.user?.role === 'ADMIN'`), the response includes an `adminDiagnostics` object.
   - It specifies:
     - The exact **Prisma Error Code** (e.g., `P2002`, `P2025`, `P2003`).
     - The exact model and database column that caused the issue.
     - An actionable resolution tip (e.g., *"Tip: Ensure Category ID exists before attaching a SubCategory"*).
   - This allows Admins in the Furnixo dashboard to immediately understand the issue without logging into server consoles.

---

### 14.2 Custom Application Error Class (`src/utils/AppError.ts`)
```typescript
export class AppError extends Error {
  public statusCode: number;
  public errorCode: string;
  public isOperational: boolean;
  public adminDetails?: any;

  constructor(
    message: string,
    statusCode: number = 400,
    errorCode: string = 'BAD_REQUEST',
    adminDetails?: any
  ) {
    super(message);
    this.statusCode = statusCode;
    this.errorCode = errorCode;
    this.isOperational = true; // True for expected business errors (vs unexpected server crashes)
    this.adminDetails = adminDetails;

    Error.captureStackTrace(this, this.constructor);
  }
}
```

---

### 14.3 Complete Global Error Handler Middleware (`src/middlewares/error.middleware.ts`)

```typescript
import { Request, Response, NextFunction } from 'express';
import { ZodError } from 'zod';
import { Prisma } from '@prisma/client';
import multer from 'multer';
import { AppError } from '../utils/AppError';

export const globalErrorHandler = (
  err: any,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  let statusCode = err.statusCode || 500;
  let userMessage = err.message || 'Something went wrong. Please try again later.';
  let errorCode = err.errorCode || 'INTERNAL_SERVER_ERROR';
  let details: any = null;
  let adminDiagnostics: any = null;

  const isAdmin = req.user?.role === 'ADMIN';
  const isDev = process.env.NODE_ENV === 'development';

  // ==========================================
  // 1. Handle Zod Form / Input Validation Errors
  // ==========================================
  if (err instanceof ZodError) {
    statusCode = 422;
    errorCode = 'VALIDATION_ERROR';
    userMessage = 'Please check the entered form details and try again.';

    // Clean user-facing field errors
    details = err.errors.map((e) => ({
      field: e.path.join('.').replace(/^(body|query|params)\./, ''),
      message: e.message,
    }));

    if (isAdmin || isDev) {
      adminDiagnostics = {
        type: 'ZodValidationError',
        failedFields: err.errors.map((e) => ({
          path: e.path.join('.'),
          code: e.code,
          expected: (e as any).expected,
          received: (e as any).received,
          message: e.message,
        })),
      };
    }
  }

  // ==========================================
  // 2. Handle Prisma Known Database Errors
  // ==========================================
  else if (err instanceof Prisma.PrismaClientKnownRequestError) {
    statusCode = 400;

    switch (err.code) {
      case 'P2002': {
        // Unique constraint violation (e.g. duplicate phone, email, slug)
        const target = (err.meta?.target as string[]) || [];
        const fieldName = target.join(', ');
        errorCode = 'DUPLICATE_ENTRY';
        userMessage = `A record with this ${fieldName || 'information'} already exists.`;

        if (isAdmin || isDev) {
          adminDiagnostics = {
            prismaCode: 'P2002',
            model: (err.meta as any)?.modelName,
            conflictingFields: target,
            tip: 'Unique constraint violated. Use a different value or edit existing record.',
          };
        }
        break;
      }

      case 'P2025': {
        // Record not found to update/delete
        statusCode = 404;
        errorCode = 'RECORD_NOT_FOUND';
        userMessage = 'The requested item could not be found.';

        if (isAdmin || isDev) {
          adminDiagnostics = {
            prismaCode: 'P2025',
            cause: err.meta?.cause || 'Record to update/delete does not exist in database.',
            tip: 'Verify that the ID passed in URL params actually exists.',
          };
        }
        break;
      }

      case 'P2003': {
        // Foreign key constraint failed
        errorCode = 'RELATION_CONSTRAINT_FAILED';
        userMessage = 'Cannot perform this action because a related item is missing or in use.';

        if (isAdmin || isDev) {
          adminDiagnostics = {
            prismaCode: 'P2003',
            field: err.meta?.field_name,
            tip: 'Foreign key failure. Make sure parent record (e.g. Category/Address) exists first.',
          };
        }
        break;
      }

      default: {
        errorCode = 'DATABASE_ERROR';
        userMessage = 'A database error occurred. Please contact support.';
        if (isAdmin || isDev) {
          adminDiagnostics = {
            prismaCode: err.code,
            meta: err.meta,
          };
        }
      }
    }
  }

  // ==========================================
  // 3. Handle Prisma Validation Errors (Type Mismatch)
  // ==========================================
  else if (err instanceof Prisma.PrismaClientValidationError) {
    statusCode = 400;
    errorCode = 'DATABASE_VALIDATION_ERROR';
    userMessage = 'Invalid data provided for database operation.';

    if (isAdmin || isDev) {
      adminDiagnostics = {
        type: 'PrismaClientValidationError',
        details: err.message.split('\n').filter((l) => l.includes('Argument') || l.includes('Unknown')),
      };
    }
  }

  // ==========================================
  // 4. Handle JWT Authentication & Expiry Errors
  // ==========================================
  else if (err.name === 'TokenExpiredError') {
    statusCode = 401;
    errorCode = 'TOKEN_EXPIRED';
    userMessage = 'Your session has expired. Please log in again.';
  } else if (err.name === 'JsonWebTokenError') {
    statusCode = 401;
    errorCode = 'INVALID_TOKEN';
    userMessage = 'Invalid authentication token. Please log in again.';
  }

  // ==========================================
  // 5. Handle Multer File Upload Errors
  // ==========================================
  else if (err instanceof multer.MulterError) {
    statusCode = 400;
    errorCode = 'UPLOAD_ERROR';

    if (err.code === 'LIMIT_FILE_SIZE') {
      userMessage = 'Uploaded file is too large. Maximum allowed size is 5MB.';
    } else if (err.code === 'LIMIT_UNEXPECTED_FILE') {
      userMessage = 'Too many files or unexpected field name in file upload.';
    } else {
      userMessage = `File upload failed: ${err.message}`;
    }

    if (isAdmin || isDev) {
      adminDiagnostics = {
        multerCode: err.code,
        field: err.field,
      };
    }
  }

  // ==========================================
  // 6. Handle Custom Application Errors (AppError)
  // ==========================================
  else if (err instanceof AppError) {
    statusCode = err.statusCode;
    errorCode = err.errorCode;
    userMessage = err.message;
    if ((isAdmin || isDev) && err.adminDetails) {
      adminDiagnostics = err.adminDetails;
    }
  }

  // ==========================================
  // 7. Unhandled Unexpected Server Errors (500)
  // ==========================================
  else {
    statusCode = 500;
    errorCode = 'INTERNAL_SERVER_ERROR';
    // STRICT SECURITY: Never leak raw error message to regular users
    userMessage = 'An unexpected server error occurred. Our team has been notified.';

    if (isAdmin || isDev) {
      adminDiagnostics = {
        rawMessage: err.message,
        stack: err.stack,
      };
    }
  }

  // Construct Final Standardized Error Response
  const responsePayload: any = {
    success: false,
    message: userMessage,
    error: {
      code: errorCode,
      details: details || undefined,
    },
  };

  // Attach diagnostic help ONLY for Admins or in local development
  if (adminDiagnostics) {
    responsePayload.error.adminDiagnostics = adminDiagnostics;
  }

  return res.status(statusCode).json(responsePayload);
};
```

---

### 14.4 Real-World Comparison: User Response vs. Admin Response

#### Scenario 1: Duplicate Phone Number Registration
- **User Response (Clean, Polite, Actionable):**
  ```json
  {
    "success": false,
    "message": "A record with this phone already exists.",
    "error": {
      "code": "DUPLICATE_ENTRY"
    }
  }
  ```

- **Admin Response (Full Diagnostic Context):**
  ```json
  {
    "success": false,
    "message": "A record with this phone already exists.",
    "error": {
      "code": "DUPLICATE_ENTRY",
      "adminDiagnostics": {
        "prismaCode": "P2002",
        "model": "User",
        "conflictingFields": ["phone"],
        "tip": "Unique constraint violated. Use a different value or edit existing record."
      }
    }
  }
  ```

#### Scenario 2: Form Validation Error (Zod)
- **User Response (Targeted Field Message for UI Form Display):**
  ```json
  {
    "success": false,
    "message": "Please check the entered form details and try again.",
    "error": {
      "code": "VALIDATION_ERROR",
      "details": [
        { "field": "phone", "message": "Invalid Bangladesh phone number. Format: 01XXXXXXXXX" },
        { "field": "password", "message": "Password must be exactly 6 digits" }
      ]
    }
  }
  ```

#### Scenario 3: Unexpected Database Failure (500 Server Crash)
- **User Response (100% Secure, Zero Secrets Leaked):**
  ```json
  {
    "success": false,
    "message": "An unexpected server error occurred. Our team has been notified.",
    "error": {
      "code": "INTERNAL_SERVER_ERROR"
    }
  }
  ```

- **Admin Response (Instant Debugging in Admin Dashboard):**
  ```json
  {
    "success": false,
    "message": "An unexpected server error occurred. Our team has been notified.",
    "error": {
      "code": "INTERNAL_SERVER_ERROR",
      "adminDiagnostics": {
        "rawMessage": "connect ECONNREFUSED 127.0.0.1:5432",
        "stack": "Error: connect ECONNREFUSED...\n    at TCPConnectWrap.afterConnect [as oncomplete]..."
      }
    }
  }
  ```



