<h1 align="center">Organization Management API</h1>

<p align="center">
  Hardened NestJS REST API for hierarchical organization management — multi-level RBAC, JWT with refresh-token rotation, and OpenAPI docs.
</p>

<p align="center">
  <img alt="NestJS" src="https://img.shields.io/badge/NestJS-E0234E?style=flat-square&logo=nestjs&logoColor=white">
  <img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white">
  <img alt="MongoDB" src="https://img.shields.io/badge/MongoDB-Mongoose-47A248?style=flat-square&logo=mongodb&logoColor=white">
  <img alt="JWT" src="https://img.shields.io/badge/Auth-JWT%20%2B%20Refresh-000000?style=flat-square&logo=jsonwebtokens&logoColor=white">
  <img alt="Swagger" src="https://img.shields.io/badge/OpenAPI-Swagger-85EA2D?style=flat-square&logo=swagger&logoColor=black">
</p>

---

## Overview

**Organization Management API** is a backend service that models a company's internal structure as a hierarchy — **Company → Branch → User** — and governs every operation through a five-level role-based access-control scheme. It is built on NestJS with a modular, dependency-injected architecture and ships with production-grade request hardening and interactive OpenAPI documentation.

The same business domain is also implemented in Java/Quarkus in [`finance-api-quarkus`](https://github.com/renanbambam/finance-api-quarkus); this repository is the **Node/NestJS** take on it, useful for comparing two backend ecosystems against one specification.

---

## Key capabilities

- **Hierarchical RBAC** — five roles (`SUPER_ADMIN`, `ADMIN`, `MANAGER`, `USER`, `CUSTOMER`) enforced declaratively via a `@Roles` decorator + global `RolesGuard`.
- **JWT auth with refresh-token rotation** — separate access and refresh strategies (Passport), dedicated refresh guard, and a login-validation middleware.
- **Globally secured by default** — `JwtAuthGuard` is applied app-wide; endpoints opt out explicitly with an `@Public()` decorator (deny-by-default posture).
- **Request hardening** — `helmet`, `cookie-parser`, rate limiting (`@nestjs/throttler`), and a strict global `ValidationPipe` (`whitelist` + `forbidNonWhitelisted` + transform).
- **Self-documenting** — Swagger/OpenAPI UI with bearer auth and custom body/file decorators.
- **Modular design** — isolated feature modules (`auth`, `user`, `company`, `branch`) over MongoDB via Mongoose.

---

## API surface

Interactive docs are served at **`/api`** (Swagger UI). All routes are protected by default; `auth` routes are public.

### Auth — `/auth`
| Method | Path | Access | Description |
|--------|------|--------|-------------|
| `POST` | `/login` | public | Authenticate, issue access + refresh tokens |
| `POST` | `/refresh` | refresh token | Rotate tokens |

### Users — `/user`
| Method | Path | Roles |
|--------|------|-------|
| `GET` | `/`, `/all`, `/:id` | `MANAGER`+ / `ADMIN`+ |
| `POST` | `/`, `/role`, `/promote-demote` | `MANAGER`, `ADMIN`, `SUPER_ADMIN` |
| `PATCH` / `DELETE` | `/` | `MANAGER`, `ADMIN`, `SUPER_ADMIN` |

### Companies — `/company`
| Method | Path | Roles |
|--------|------|-------|
| `GET` | `/`, `/all`, `/name`, `/:id` | `SUPER_ADMIN` |
| `POST` | `/` | `SUPER_ADMIN`, `ADMIN` |
| `PATCH` / `DELETE` | `/` | `SUPER_ADMIN` |

### Branches — `/branch`
| Method | Path | Roles |
|--------|------|-------|
| `GET` / `POST` / `PATCH` / `DELETE` | `/`, `/all`, `/name`, `/:id` | `SUPER_ADMIN`, `ADMIN` |

---

## Architecture

See [`ARCHITECTURE.md`](./ARCHITECTURE.md) for diagrams (module graph, request pipeline, auth flow, role hierarchy).

```
src/
├── auth/        Authentication & authorization
│   ├── strategies/   JWT + refresh Passport strategies
│   ├── guards/       JwtAuthGuard · RefreshAuthGuard · RolesGuard
│   ├── decorators/   @Roles · @Public · @CurrentUser · Swagger helpers
│   └── middlewares/  login validation
├── user/        User domain (roles, promotion/demotion)
├── company/     Company domain
└── branch/      Branch domain
```

Every feature module follows NestJS layering: `controller → service → schema/entity`, with DTOs validated by `class-validator`.

---

## Getting started

### Prerequisites
- Node.js 18+
- MongoDB instance
- npm

### Configuration
No secrets are committed. Copy the template and fill it in:
```bash
cp .env.example .env
```
| Variable | Description |
|----------|-------------|
| `DB_URI` | MongoDB connection string |
| `JWT_SECRET` / `JWT_EXPIRE` | Access-token signing secret & TTL |
| `RT_SECRET` / `RT_EXPIRE` | Refresh-token signing secret & TTL |
| `PORT` | HTTP port (default `3000`) |

### Run
```bash
npm install
npm run start:dev      # watch mode
```
Swagger UI: <http://localhost:3000/api>

### Quality
```bash
npm run lint
npm test               # unit (Jest)
npm run test:e2e       # end-to-end
```

---

## Tech stack

`NestJS` · `TypeScript` · `MongoDB + Mongoose` · `Passport JWT (access + refresh)` · `class-validator` · `@nestjs/throttler` · `helmet` · `Swagger / OpenAPI`
