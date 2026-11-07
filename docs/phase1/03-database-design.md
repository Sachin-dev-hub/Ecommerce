# Phase 1 — Database Design (PostgreSQL + Prisma)

> File: `docs/phase1/03-database-design.md`
> **Status:** proposed — awaiting confirmation before code is generated.

## 1. Overview

Six models: **Admin**, **Product**, **ProductImage**, **Customer**, **Order**, **OrderItem**.

## 2. Entity-Relationship Diagram (ERD)

```
Admin  ──────────── (independent; authorizes product writes)

Product  1────∞  ProductImage      (cascade delete)

Customer 1────∞  Order
Order    1────∞  OrderItem         (cascade delete)
OrderItem ∞────1  Product          (price/name snapshotted for history)
```

**Relationships:**
- A **Product** has many **ProductImages** (one-to-many). Deleting a product deletes its images (`onDelete: Cascade`).
- A **Customer** has many **Orders** (one-to-many).
- An **Order** has many **OrderItems** (one-to-many, cascade delete).
- Each **OrderItem** references one **Product** (many-to-one), but stores a snapshot of the product name and price.

## 3. Prisma Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Admin {
  id           Int      @id @default(autoincrement())
  name         String
  email        String   @unique
  passwordHash String
  role         String   @default("ADMIN")   // future: SUPER_ADMIN
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
}

model Product {
  id          Int            @id @default(autoincrement())
  name        String
  slug        String         @unique      // SEO-friendly URL
  description String
  features    String[]                     // bullet points (Postgres text[])
  price       Decimal        @db.Decimal(10,2)
  stock       Int            @default(0)
  isActive    Boolean        @default(true)
  createdAt   DateTime       @default(now())
  updatedAt   DateTime       @updatedAt
  images      ProductImage[]
}

model ProductImage {
  id         Int     @id @default(autoincrement())
  product    Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  productId  Int
  url        String                 // "/uploads/xxx.jpg" — S3 URL in Phase 5
  altText    String?
  isPrimary  Boolean @default(false)
  sortOrder  Int     @default(0)
}

model Customer {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  phone     String?
  address   String?
  createdAt DateTime @default(now())
  orders    Order[]
}

model Order {
  id           Int         @id @default(autoincrement())
  orderNumber  String      @unique        // e.g. "DYA-000123"
  customer     Customer    @relation(fields: [customerId], references: [id])
  customerId   Int
  status       OrderStatus @default(PENDING)
  totalAmount  Decimal     @db.Decimal(10,2)
  createdAt    DateTime    @default(now())
  updatedAt    DateTime    @updatedAt
  items        OrderItem[]
}

model OrderItem {
  id          Int      @id @default(autoincrement())
  order       Order    @relation(fields: [orderId], references: [id], onDelete: Cascade)
  orderId     Int
  product     Product  @relation(fields: [productId], references: [id])
  productId   Int
  productName String   // snapshot — survives product deletion
  unitPrice   Decimal  @db.Decimal(10,2) // snapshot of price at purchase time
  quantity    Int
  subtotal    Decimal  @db.Decimal(10,2)
}

enum OrderStatus {
  PENDING
  CONFIRMED
  SHIPPED
  DELIVERED
  CANCELLED
}
```

## 4. Design Decisions & Why

| Decision | Why |
|----------|-----|
| **OrderItem snapshots** `productName`/`unitPrice` | Order history stays accurate even if the product is later renamed, repriced, or deleted. This is standard e-commerce practice. |
| **Cascade delete** (Product→Image, Order→Item) | No orphaned rows; DB stays clean automatically. |
| **`Decimal(10,2)` for all money** | Never use `float` for money — floating-point rounding causes real pricing bugs. |
| **Unique `slug` on Product** | Clean, shareable URLs (`/products/diya-marigold`) and SEO-friendly. |
| **`String[]` for features** | Postgres native `text[]`; renders as bullet points on the details page. |
| **`orderNumber` unique** | Human-friendly order reference for customers and support. |
| **Customer + Order separate from Admin** | Admin manages the catalog; customers are order-level entities. Different lifetimes and auth. |
| **No payment fields yet** | Payment status/gateway reference is added in Phase 7 — the schema is structured to accept it without rework. |
| **`isActive` on Product** | Soft-hide products without deleting them (e.g., sold out of season). |

## 5. Real-World Production Notes

- **Indexes:** Prisma auto-creates indexes for `@unique` and `@relation` fields. If the catalog grows large, we'd add explicit indexes on `Order.customerId`, `Order.createdAt`, `Product.isActive` for query performance.
- **Migrations:** every change to this schema produces a versioned migration under `prisma/migrations/` — this is how production DB upgrades stay reversible and reviewed.
- **Money type note:** Decimal maps to Postgres `numeric(10,2)`, exact for currency.
- **Backups** of this DB become a Phase 7 concern; the migration history makes restore/testing easy.

See also: [01-architecture.md](01-architecture.md) · [04-api-design.md](04-api-design.md).