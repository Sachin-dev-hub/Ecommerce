# Phase 1 — REST API Design

> File: `docs/phase1/04-api-design.md`
> **Status:** proposed — awaiting confirmation before code is generated.

## 1. Endpoint Overview

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| POST | `/api/admin/login` | — | Admin login → returns JWT |
| GET | `/api/products` | — | List products (search + stock filter) |
| GET | `/api/products/:slug` | — | Single product + image gallery |
| POST | `/api/products` | Admin | Create product |
| PUT | `/api/products/:id` | Admin | Update product |
| DELETE | `/api/products/:id` | Admin | Delete product |
| POST | `/api/products/:id/images` | Admin | Upload product image (multipart) |
| DELETE | `/api/images/:id` | Admin | Delete an image |
| POST | `/api/orders` | — | Place order (creates Customer + Order) |
| GET | `/api/orders/:id` | — | Order lookup (customer confirmation) |
| GET | `/api/orders` | Admin | List all orders |
| PATCH | `/api/orders/:id/status` | Admin | Update order status |

## 2. Request / Response Conventions

- Base URL (dev): `http://localhost:5000`
- All responses JSON; errors use a consistent shape:

```json
{ "success": false, "message": "Product not found" }
```

- Successful responses use a consistent shape:

```json
{ "success": true, "data": { ... } }
```

- **Auth:** admin endpoints require header `Authorization: Bearer <JWT>`. The `auth` middleware rejects missing/invalid/expired tokens with `401`.

## 3. Key Endpoint Details

### POST `/api/admin/login`
```json
// request
{ "email": "admin@diyashop.com", "password": "admin123" }
```
```json
// response 200
{
  "success": true,
  "data": {
    "token": "eyJhbGciOi...",
    "admin": { "id": 1, "name": "Admin", "email": "admin@diyashop.com", "role": "ADMIN" }
  }
}
```
- Password verified with `bcrypt.compare`; JWT signed with `ADMIN_JWT_SECRET`, expires in 7 days.
- Never return `passwordHash`.

### POST `/api/products`  (Admin)
```json
// request body
{
  "name": "Marigold Diya",
  "description": "Handmade clay diya with marigold motif.",
  "features": ["Handcrafted", "Eco-friendly", "Battery lamp optional"],
  "price": 249.0,
  "stock": 50,
  "isActive": true
}
```
- `slug` auto-generated from `name` (slugify utility). 409 if slug already exists.
- Validated with express-validator: name/description/price required, price > 0, stock ≥ 0.

### POST `/api/products/:id/images`  (Admin)
- `multipart/form-data`, field name `image`.
- Multer validates: image MIME type only (`jpeg/png/webp`), max 5 MB.
- File saved to `backend/src/uploads/`, DB row created with URL `/uploads/<filename>`.
- First image for a product automatically becomes `isPrimary`.

### POST `/api/orders`
```json
// request body
{
  "customer": { "name": "Asha", "email": "asha@example.com", "phone": "9876543210", "address": "…" },
  "items": [ { "productId": 1, "quantity": 2 } ]
}
```
```json
// response 201
{ "success": true, "data": { "orderNumber": "DYA-000001", "status": "PENDING", "totalAmount": 498.0, "items": [...] } }
```
- Runs inside a DB **transaction**: upsert Customer → create Order + OrderItems → compute total server-side (never trust client totals) → **decrement stock** with an in-transaction stock check.
- Rejects orders when stock is insufficient (`409`).

### GET `/api/products`
- Optional query params: `?search=diya&inStockOnly=true`.
- Returns only `isActive: true` products by default (admin sees all via a separate flag or endpoint).

## 4. Validation & Error Handling

- **express-validator** on every body/param; first validation failure → `400` with field message.
- Central **errorHandler middleware**: known errors → proper status codes; unexpected errors → `500` with generic message (no stack trace leaked to client).
- `asyncHandler` wraps every async controller so thrown errors are forwarded to the error handler (no unhandled promise rejections).

## 5. Interview Notes

- **Why JWT?** Stateless — the API can scale horizontally (K8s, Phase 5) without shared session storage.
- **Why server-side totals?** Never trust client-sent totals; a forged cart could undercharge.
- **Why transaction for orders?** Stock decrement + order creation must be atomic; a crash mid-way must not oversell or create half-orders.

See also: [03-database-design.md](03-database-design.md) · [05-implementation-steps.md](05-implementation-steps.md).