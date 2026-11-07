# Phase 1 — Application Architecture

> File: `docs/phase1/01-architecture.md` · Part of the Phase 1 design documentation.
> **Status:** proposed — awaiting user confirmation before any application code is generated.

## 1. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Browser (User / Admin)                      │
└─────────────────────────────────────────────────────────────────────┘
                                │  HTTP
                                ▼
                        ┌─────────────────┐
                        │  React SPA (Vite)│   Frontend  :5173
                        │  Product pages,  │
                        │  Cart, Admin UI  │
                        └─────────────────┘
                                │  REST / JSON
                                ▼
                        ┌─────────────────┐
                        │ Express API      │   Backend  :5000
                        │ Auth, Products,  │
                        │ Images, Orders   │
                        └─────────────────┘
                          │          │
               Prisma ORM │          │ serve /uploads (static)
                          ▼          ▼
                 ┌──────────────┐  ┌──────────────────┐
                 │ PostgreSQL   │  │ uploads/ folder  │
                 │ (products,   │  │ (product images) │
                 │  orders,     │  └──────────────────┘
                 │  customers)  │
                 └──────────────┘
```

## 2. Components

| Component | Technology | Responsibility |
|-----------|-----------|----------------|
| Frontend | React (SPA) + Vite | Renders product listing, product details, image gallery, shopping cart, admin login + dashboard. Talks to the backend only via REST JSON. |
| Backend | Node.js + Express | REST API server: admin auth (JWT), product CRUD, image upload, order management. |
| Database | PostgreSQL | Source of truth for products, images, customers, orders. |
| ORM | Prisma | Type-safe bridge between Express and PostgreSQL; schema-as-code; migrations. |
| Static files | Express static / `uploads/` | Serves uploaded product images during development. Replaced by S3 in Phase 5. |

## 3. Data Flow (request lifecycle)

**Customer browsing:**
1. Browser loads React SPA.
2. React fetches `GET /api/products` → Express → Prisma → PostgreSQL → JSON back to React → rendered as product grid.
3. User clicks a product → `GET /api/products/:slug` returns details + image gallery.
4. User adds to cart → handled in React state (localStorage persisted); cart contents become an order only at checkout.

**Placing an order:**
1. Checkout sends `POST /api/orders` with customer details + cart items.
2. Backend creates/links a `Customer`, creates the `Order` + `OrderItem` rows, computes totals in a transaction.
3. Backend returns the `orderNumber` → confirmation page.

**Admin:**
1. `POST /api/admin/login` → validates credentials → returns JWT.
2. Admin stores JWT; every subsequent admin request sends it in the `Authorization` header.
3. Express `auth` middleware verifies the JWT before letting the request reach product/order controllers.
4. Image upload: `POST /api/products/:id/images` with `multipart/form-data` → Multer validates + saves file → URL stored in `ProductImage` row.

## 4. Why This Stack (interview-prep)

- **React + Vite** — component-based UI with fast dev server and optimized production builds; Vite is the modern replacement for Create React App.
- **Express** — minimal, mature REST framework; huge ecosystem; standard for Node APIs.
- **PostgreSQL** — relational integrity (foreign keys, constraints, transactions) that an e-commerce domain (orders, inventory) genuinely needs.
- **Prisma** — removes raw SQL, gives type-safe queries, auto migrations, easy seeding.
- **JWT + bcrypt** — stateless, scalable admin auth; passwords hashed, never stored in plain text.
- **Multer** — standard, battle-tested file-upload middleware.

## 5. How It Maps to Production (later phases)

- The exact same layout gets containerized in **Phase 3** (Docker + Kubernetes).
- **Phase 5** moves DB to RDS, images to S3, API behind a load balancer.
- **Phase 7** adds a payment gateway — orders already have the structure needed to attach payment status.
- Separation of concerns here (frontend ↔ API ↔ DB) is what makes each of those moves a small, isolated change.

See also: [07-technology-stack.md](07-technology-stack.md) for the full technology → purpose reference.