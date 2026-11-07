# Phase 1 — Proposed Folder Structure

> File: `docs/phase1/02-folder-structure.md`
> **Status:** proposed — awaiting confirmation before code is generated.

## 1. Repo Layout

```
Ecommerce/
├── backend/
├── frontend/
├── docs/
│   └── phase1/           # this documentation
├── CLAUDE.md
├── README.md
└── requirement           # single source of truth (spec)
```

Frontend and backend live in separate top-level folders so each can be built, tested, and later containerized independently.

## 2. Backend Structure

```
backend/
├── src/
│   ├── config/
│   │   └── db.js              # Prisma client instance + app config
│   ├── controllers/           # request handlers (business logic)
│   │   ├── auth.controller.js
│   │   ├── product.controller.js
│   │   ├── image.controller.js
│   │   └── order.controller.js
│   ├── middlewares/
│   │   ├── auth.js            # JWT verification (protects admin routes)
│   │   ├── upload.js          # Multer config (storage, file filter, limits)
│   │   ├── validate.js        # express-validator request validation
│   │   └── errorHandler.js    # centralized error responses
│   ├── routes/                # thin route files → controller functions
│   │   ├── auth.routes.js
│   │   ├── product.routes.js
│   │   ├── image.routes.js
│   │   └── order.routes.js
│   ├── utils/
│   │   ├── asyncHandler.js    # wraps async controllers, forwards errors
│   │   └── slugify.js         # name → URL slug helper
│   ├── uploads/               # product images (local dev; gitignored)
│   └── index.js               # server entry point (Express app + static)
├── prisma/
│   ├── schema.prisma          # DB schema (source of truth)
│   ├── seed.js                # default admin + sample products
│   └── migrations/            # auto-generated SQL migrations
├── .env                       # secrets (gitignored)
├── .env.example               # template committed to git
├── .gitignore
└── package.json
```

### Why each part matters
- **`middlewares/`** — auth and validation are cross-cutting concerns; keeping them as middleware means a single `auth` check protects every admin route instead of duplicating logic in each controller.
- **`controllers/` vs `routes/`** — routes only map URL → handler; controllers hold logic. Keeps files small and unit-testable.
- **`prisma/migrations/`** — every schema change produces a versioned migration; this is what makes production DB upgrades predictable later.
- **`uploads/`** — local-only image storage for Phase 1. Gitignored because images are data, not code; swapped for S3 in Phase 5.
- **`.env` / `.env.example`** — secrets never committed; the example file documents exactly which env vars the app needs.

## 3. Frontend Structure

```
frontend/
├── src/
│   ├── api/
│   │   ├── client.js           # axios instance (base URL + auth header)
│   │   ├── products.js         # product API calls
│   │   └── orders.js           # order API calls
│   ├── components/
│   │   ├── Navbar.jsx
│   │   ├── ProductCard.jsx
│   │   ├── ImageGallery.jsx
│   │   └── CartItem.jsx
│   ├── context/
│   │   ├── CartContext.jsx     # cart state (add/remove/clear, persisted)
│   │   └── AuthContext.jsx     # admin JWT state + login/logout
│   ├── pages/
│   │   ├── Home.jsx            # product listing
│   │   ├── ProductDetails.jsx  # details + image gallery
│   │   ├── Cart.jsx
│   │   ├── Checkout.jsx        # order placement (payment later, Phase 7)
│   │   ├── AdminLogin.jsx
│   │   └── AdminDashboard.jsx  # product CRUD + image upload + order status
│   ├── App.jsx                 # React Router setup
│   ├── main.jsx                # React entry point
│   └── index.css
├── index.html
├── vite.config.js              # dev proxy /api → backend :5000
└── package.json
```

### Why this shape
- **`context/`** — cart and auth state is shared across many pages; React Context avoids prop-drilling and external state libraries for a project this size.
- **`api/`** — all HTTP calls isolated in one layer; if the backend URL changes or auth headers are needed, it's a one-file change.
- **`components/` vs `pages/`** — reusable UI pieces (cards, gallery, navbar) vs. full routed views.
- **`vite.config.js` proxy** — during development, the frontend calls `/api/...` and Vite forwards to the backend, avoiding CORS and hardcoded URLs.

See also: [01-architecture.md](01-architecture.md) and [07-technology-stack.md](07-technology-stack.md).