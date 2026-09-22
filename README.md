# Adilbay / Furnixo — Backend Server

This repository contains the backend RESTful API server for the **Adilbay / Furnixo** E-Commerce platform, built for scalability, enterprise security, and high performance.

---

## 🛠️ Technology Stack

- **Runtime & Language**: Node.js, TypeScript
- **Framework**: Express.js
- **Database**: PostgreSQL
- **ORM**: Prisma ORM
- **In-Memory Cache & Session**: Redis (`ioredis`)
- **Validation**: Zod
- **Authentication**: JWT (Access Tokens) + HttpOnly Refresh Tokens + Phone OTP (BulkSMSBD)
- **Security**: Helmet, CORS, Express-Rate-Limit (Redis Store)
- **Media Storage**: Cloudinary + Multer

---

## 📁 Repository Structure

```
adilbay-server/
├── docs/                               # Comprehensive Server Documentation
│   ├── README.md                       # Backend Docs Index
│   └── backend-architecture-srs.md     # Master SRS & Architectural Blueprint
├── prisma/                             # Prisma Schema & Seeds
│   ├── schema.prisma                   # Database Schema Definition
│   └── seed.ts                         # Initial Seed Data
├── src/                                # Application Source Code
│   ├── @types/                         # TypeScript Type Augmentations
│   ├── config/                         # Environment & Client Configs (Prisma, Redis)
│   ├── constants/                      # Enums, HTTP Statuses, Error Codes
│   ├── controllers/                    # Route Handlers / HTTP Controllers
│   ├── errors/                         # Custom Error Classes & Global Handler
│   ├── helpers/                        # Pagination, QueryBuilder, Crypto Utilities
│   ├── middlewares/                    # Auth, RBAC, Validation, Error Handling
│   ├── routes/                         # Express Route Definitions
│   ├── services/                       # Business Logic Layer
│   ├── types/                          # Shared DTOs and Data Interfaces
│   ├── validations/                    # Zod Request Validation Schemas
│   ├── app.ts                          # Express Application Setup
│   └── server.ts                       # Server Entry Point & Process Listeners
├── .env.example                        # Template Environment Variables
├── package.json
└── tsconfig.json
```

---

## ⚡ Quick Start

### 1. Prerequisites
- **Node.js** (v18.x or v20.x+)
- **PostgreSQL** (v14+ running locally or in cloud)
- **Redis** (v6+ running locally or in cloud)

### 2. Install Dependencies
```bash
npm install
```

### 3. Setup Environment Variables
Create a `.env` file in the `adilbay-server` directory based on `.env.example`:

```env
# Server
NODE_ENV=development
PORT=5000
API_PREFIX=/api/v1
CLIENT_URL=http://localhost:3000

# Database (PostgreSQL)
DATABASE_URL="postgresql://postgres:password@localhost:5432/adilbay_db?schema=public"

# Redis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=

# JWT Secrets
JWT_ACCESS_SECRET=your_super_secret_access_key
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_SECRET=your_super_secret_refresh_key
JWT_REFRESH_EXPIRES_IN=7d

# SMS Gateway (BulkSMSBD)
SMS_API_KEY=your_bulksmsbd_api_key
SMS_SENDER_ID=your_sender_id

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

### 4. Database Setup & Seed
```bash
# Generate Prisma Client
npx prisma generate

# Apply Migrations
npx prisma migrate dev --name init

# Seed Database (Default admin, categories, demo products)
npx prisma db seed
```

### 5. Start Development Server
```bash
npm run dev
```
The API server will run at: `http://localhost:5000/api/v1`

---

## 📖 In-Depth Documentation

For the complete architectural design, database entity-relationship diagram, Zod validation rules, endpoint inventory, and security policies, refer to:
- [Backend Documentation Index](file:///d:/Restart/CodeClub/adilbay/adilbay-server/docs/README.md)
- [Backend Architecture & SRS Blueprint](file:///d:/Restart/CodeClub/adilbay/adilbay-server/docs/backend-architecture-srs.md)
- [Root Workspace README](file:///d:/Restart/CodeClub/adilbay/README.md)
