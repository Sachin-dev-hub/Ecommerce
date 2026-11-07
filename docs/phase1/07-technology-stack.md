# Phase 1 — Technology Stack & What We Use Each For

> File: `docs/phase1/07-technology-stack.md`
> **The single reference file: every technology in this project and its purpose.**
> Updated as new phases add technologies.

## 1. Quick Reference Table

| # | Technology | Category | What we use it for |
|---|-----------|----------|--------------------|
| 1 | **React.js** | Frontend | Building the user interface: product listing, product details, image gallery, shopping cart, admin login + dashboard |
| 2 | **Vite** | Frontend build tool | Fast dev server + production build for the React app |
| 3 | **React Router** | Frontend routing | Navigating between pages (Home, Product, Cart, Admin, …) |
| 4 | **Axios** | HTTP client | Frontend ↔ backend API calls |
| 5 | **React Context** | Frontend state | Cart + admin auth state shared across pages |
| 6 | **Node.js** | Backend runtime | Runs the Express API server (JavaScript on the server) |
| 7 | **Express.js** | Backend framework | REST API routes, middleware, request handling |
| 8 | **Multer** | File upload | Product image upload handling |
| 9 | **bcryptjs** | Security | Hashing admin passwords |
| 10 | **jsonwebtoken** | Security | Admin authentication (JWT) |
| 11 | **Helmet** | Security | HTTP security headers |
| 12 | **CORS** | Security | Controlling which origins can call the API |
| 13 | **express-validator** | Validation | Validating API request inputs |
| 14 | **PostgreSQL** | Database | Storing products, images, customers, orders |
| 15 | **Prisma ORM** | ORM | Type-safe DB access, schema, migrations, seeding |
| 16 | **dotenv** | Config | Loading `.env` environment variables |
| 17 | **nodemon** | Dev tool | Auto-restart backend on code changes during development |

## 2. Detailed Explanations

### Frontend
**React.js** — Component-based UI library. Each page and reusable element (product card, gallery, navbar) is a component; state changes re-render only what's needed. This is what makes the shop responsive and interactive.

**Vite** — Build tool that gives an instant dev server (hot reload) and a minified, optimized production bundle. Modern replacement for Create React App.

**React Router** — Declarative client-side routing. `/` → home, `/products/:slug` → product details, `/cart`, `/admin/login`, `/admin` → dashboard. The app never does a full page reload.

**Axios** — Promise-based HTTP client with clean error handling and interceptors (used to attach the JWT to admin requests automatically).

**React Context** — Built-in React state sharing. `CartContext` holds cart items (persisted to localStorage); `AuthContext` holds the admin JWT + login state. No external state library needed at this scale.

### Backend
**Node.js** — JavaScript runtime on the server. Same language as the frontend → one language for the whole app. Event-driven, non-blocking — good for I/O-heavy API workloads.

**Express.js** — Minimal web framework on top of Node. Maps URLs to handler functions, runs middleware (auth, validation, errors) in a chain. The most widely used Node API framework.

**Multer** — Express middleware for `multipart/form-data` (file uploads). Handles storage, file-type filtering, size limits.

**bcryptjs** — One-way password hashing. Passwords are stored only as salted bcrypt hashes; even if the DB leaks, passwords stay protected.

**jsonwebtoken** — Signs and verifies JWTs. After admin login, the token proves identity on every subsequent request — stateless, works across multiple servers.

**Helmet** — Sets secure HTTP headers (CSP, X-Frame-Options, X-Content-Type-Options, HSTS) to harden the API against common web attacks.

**CORS** — HTTP header control. By default browsers block cross-origin calls; CORS config lets our React app (port 5173) call the API (port 5000) while blocking others.

**express-validator** — Declarative validation for request bodies/params. Returns 400 with clear messages before bad data reaches the database.

### Data Layer
**PostgreSQL** — Full-featured relational database. Enforces relationships (FKs), constraints, and transactions — exactly what orders and inventory need. Mature, open source, industry standard.

**Prisma ORM** — Declarative schema → generates a type-safe client, auto-generates migrations, provides seeding. Replaces hand-written SQL with safe, typed queries.

**dotenv** — Loads `DATABASE_URL`, `ADMIN_JWT_SECRET`, `PORT` from `.env` into `process.env` at startup.

## 3. How They Fit Together (data flow)

```
Browser
  │ React renders pages
  ▼
Axios ──(REST JSON)──▶ Express (routes → middleware → controllers)
  │                         │                    │
  │                         │ multer             │ bcrypt / jwt
  │                         ▼                    ▼
  │                  uploads/ (images)     authentication
  ▼                         │
Prisma ──▶ PostgreSQL (products, images, customers, orders)
```

- **dotenv** feeds config to Express + Prisma.
- **nodemon** keeps the dev server fresh during development.
- **Vite proxy** forwards `/api` calls from the frontend to the backend during development.

## 4. What's Coming in Later Phases (so this list stays honest)

| Phase | Technologies added |
|-------|-------------------|
| 2 | Git, GitHub, branching strategy |
| 3 | Docker, Docker Compose, Kubernetes (Deployments, Services, ConfigMaps, Secrets, Ingress) |
| 4 | Jenkins, Trivy, container registry |
| 5 | Terraform, AWS (VPC, EKS, ECR, RDS, S3, CloudWatch, IAM) |
| 6 | DNS, ACM, ALB, HTTPS/TLS |
| 7 | Prometheus, Grafana, payment gateway, backup/DR tooling |

*(This section will be expanded into its own file when each phase begins.)*

## 5. Why These Choices (interview-prep summary)

- **One language everywhere** (JS/TS) — smaller team, easier hiring, shared utilities.
- **React + Vite** — fastest modern DX and production performance.
- **PostgreSQL + Prisma** — correctness-first data layer with zero SQL-injection surface.
- **JWT + bcrypt** — stateless, horizontally scalable auth.
- **Express + ecosystem** — battle-tested, huge community, easy to find answers.

See also: [01-architecture.md](01-architecture.md) · [02-folder-structure.md](02-folder-structure.md).