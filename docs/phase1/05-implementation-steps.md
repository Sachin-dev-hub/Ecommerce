# Phase 1 — Implementation Steps

> File: `docs/phase1/05-implementation-steps.md`
> **Status:** proposed — awaiting confirmation before code is generated.

## 0. Environment Check (this machine)

- ✅ Node v18.19.1, npm 9.2.0 installed
- ❌ PostgreSQL **not installed** → Step 1 below installs it

## 1. Install PostgreSQL Locally

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install -y postgresql postgresql-contrib
sudo systemctl enable --now postgresql

# create app user + database
sudo -u postgres psql -c "CREATE USER diya_user WITH PASSWORD 'diya_pass';"
sudo -u postgres psql -c "CREATE DATABASE diyashop OWNER diya_user;"
```

Verify:
```bash
psql "postgresql://diya_user:diya_pass@localhost:5432/diyashop" -c "SELECT 1;"
```
→ prints `1` when the connection works.

> **Alternative (optional):** run PostgreSQL in Docker instead:
> `docker run -d --name diya-db -e POSTGRES_PASSWORD=diya_pass -p 5432:5432 postgres:16`
> Local install is the default for Phase 1 to keep it self-contained; Docker is a Phase 3 topic.

## 2. Backend Scaffold

```bash
mkdir -p backend/src/{config,controllers,middlewares,routes,utils,uploads}
cd backend
npm init -y
npm install express cors dotenv helmet express-validator jsonwebtoken bcryptjs multer
npm install -D prisma @prisma/client nodemon
```

- `helmet` → security headers
- `multer` → image uploads
- `nodemon` → dev auto-restart

## 3. Prisma Setup

```bash
npx prisma init
# paste schema from docs/phase1/03-database-design.md into prisma/schema.prisma
# set DATABASE_URL in .env  →  postgresql://diya_user:diya_pass@localhost:5432/diyashop

npx prisma migrate dev --name init     # creates tables + migration
npx prisma db seed                     # default admin + sample diya products
```

`.env` (gitignored):
```
DATABASE_URL="postgresql://diya_user:diya_pass@localhost:5432/diyashop"
ADMIN_JWT_SECRET="change-me-long-random-string"
PORT=5000
```

`.env.example` (committed, same keys with placeholder values).

## 4. Backend Code (in order)

1. `src/config/db.js` — PrismaClient instance
2. `src/utils/slugify.js`, `src/utils/asyncHandler.js`
3. `src/middlewares/auth.js` — JWT verify → `req.admin`
4. `src/middlewares/upload.js` — Multer (storage, image-only filter, 5 MB limit)
5. `src/middlewares/validate.js` — express-validator rules per route
6. `src/middlewares/errorHandler.js` — centralized error responses
7. Controllers: `auth` → `product` → `image` → `order`
8. Routes wiring each controller to the endpoints in [04-api-design.md](04-api-design.md)
9. `src/index.js` — Express app: `helmet`, `cors`, `express.json()`, routes, static `/uploads`, error handler, listen on `PORT`

## 5. API Validation Checklist

Start the server: `npm run dev` (nodemon) → API on `http://localhost:5000`.

| # | Test | Expected |
|---|------|----------|
| 1 | `POST /api/admin/login` (seeded credentials) | `200` + JWT token |
| 2 | `POST /api/admin/login` (wrong password) | `401` |
| 3 | `POST /api/products` without token | `401` |
| 4 | `POST /api/products` with token | `201` + product JSON |
| 5 | `GET /api/products` | `200` + array (includes seeded products) |
| 6 | `GET /api/products/:slug` | `200` + product + images; `404` for unknown slug |
| 7 | `PUT /api/products/:id` (admin) | `200` + updated product |
| 8 | `POST /api/products/:id/images` (admin, multipart) | `201` + image URL |
| 9 | `DELETE /api/images/:id` (admin) | `200` + file + DB row removed |
| 10 | `POST /api/orders` (valid items) | `201` + orderNumber, stock decreased |
| 11 | `POST /api/orders` (insufficient stock) | `409` |
| 12 | `GET /api/orders` (admin) | `200` + list |
| 13 | `PATCH /api/orders/:id/status` (admin) | `200` + new status |

(Full curl/Postman commands provided alongside each endpoint when we build it.)

## 6. Frontend Scaffold

```bash
npm create vite@latest frontend -- --template react
cd frontend
npm install react-router-dom axios
```

- `vite.config.js`: proxy `/api` → `http://localhost:5000` (no CORS friction in dev)

## 7. Frontend Pages (in order)

1. `api/client.js` (axios instance) → `products.js`, `orders.js`
2. `context/CartContext.jsx` (cart state, localStorage persistence)
3. `context/AuthContext.jsx` (admin JWT + login/logout)
4. Components: `Navbar`, `ProductCard`, `ImageGallery`, `CartItem`
5. Pages: `Home` (listing) → `ProductDetails` (+gallery) → `Cart` → `Checkout` → `AdminLogin` → `AdminDashboard` (product CRUD, image upload, order status)
6. `App.jsx` routing + protected admin route (redirects to login if no token)

## 8. End-to-End Validation

Run both:
- Backend: `npm run dev` (port 5000)
- Frontend: `npm run dev` (port 5173) → open `http://localhost:5173`

**Customer journey:** browse products → open details + gallery → add to cart → checkout → order placed → confirmation with order number.
**Admin journey:** login → create product → upload images → edit product → place test order → update order status.

## 9. Documentation

- Write `README.md` with: prerequisites, install, DB setup, migration + seed, run commands, API summary, test checklist.

## 10. Definition of Done (Phase 1)

- [ ] PostgreSQL running with `diyashop` DB, migrated, seeded
- [ ] All API endpoints work and pass the validation checklist
- [ ] Frontend renders product listing, details, gallery, cart, checkout
- [ ] Admin login + dashboard CRUD + order management work
- [ ] Customer purchase journey works end-to-end locally
- [ ] README documents setup/run/test

Then, and only after **your confirmation**, Phase 1 is marked done and we proceed to Phase 2.