# Backend Environment Setup on Your Machine (POC)

> File: `docs/phase1/08-backend-setup-on-poc.md`
> Companion to [05-implementation-steps.md](05-implementation-steps.md). Records the exact commands used to bring the backend environment up on the user's machine (`azureuser@POC`, `~/backend`).

## 1. The Mistakes Caught (learn from these)

The first attempt showed two common setup errors:

1. **`npm install` interrupted with Ctrl+C** — installs are atomic; the interrupted run installed nothing. Always re-run to completion.
2. **Env vars typed as shell assignments instead of a `.env` file.**

```bash
# ✗ WRONG — creates temporary shell variables, never read by the app
DATABASE_URL="postgresql://..."
ADMIN_JWT_SECRET="change-me"
PORT=5000
```

dotenv reads the `.env` file from disk at server startup. Shell variables set at the prompt exist only in that shell session and are invisible to the app.

```bash
# ✓ CORRECT — write the values into .env (this is what dotenv reads)
cat > .env <<'EOF'
DATABASE_URL="postgresql://diya_user:diya_pass@localhost:5432/diyashop"
ADMIN_JWT_SECRET="change-me-long-random-string"
PORT=5000
EOF
```

## 2. Exact Commands (in order, from `~/backend`)

### Install dependencies
```bash
npm install express cors dotenv helmet express-validator jsonwebtoken bcryptjs multer @prisma/client
npm install -D prisma nodemon
```
- `@prisma/client` → runtime dependency (used by the app)
- `prisma` → dev dependency (CLI: migrations, seeding)
- `nodemon` → dev dependency (auto-restart on file changes)

### Create the `.env` file
```bash
cat > .env <<'EOF'
DATABASE_URL="postgresql://diya_user:diya_pass@localhost:5432/diyashop"
ADMIN_JWT_SECRET="change-me-long-random-string"
PORT=5000
EOF
```

### Create the folder structure
```bash
mkdir -p src/config src/controllers src/middlewares src/routes src/utils src/uploads
```

### Create `.gitignore` (Phase 2 ready — never commit these)
```bash
cat > .gitignore <<'EOF'
node_modules/
.env
src/uploads/*
!src/uploads/.gitkeep
EOF
touch src/uploads/.gitkeep
```

## 3. Verify PostgreSQL Before Prisma Migrate

```bash
pg_isready
# expected: /var/run/postgresql:5432 - accepting connections
```

If not installed/running:
```bash
sudo apt update && sudo apt install -y postgresql postgresql-contrib
sudo systemctl enable --now postgresql
sudo -u postgres psql -c "CREATE USER diya_user WITH PASSWORD 'diya_pass';"
sudo -u postgres psql -c "CREATE DATABASE diyashop OWNER diya_user;"
```

## 4. What's Next

After install completes and Postgres accepts connections:

1. `npx prisma init` → creates `prisma/schema.prisma` + `.env`
2. Paste the schema from [03-database-design.md](03-database-design.md)
3. `npx prisma migrate dev --name init`
4. `npx prisma db seed` (seed file to be added — default admin + sample diyas)
5. Build `src/` code (config → middlewares → controllers → routes → `index.js`)

## 5. Production Mapping (why this matters later)

| Local (now) | Production (Phase 4/5) |
|-------------|------------------------|
| `.env` file, gitignored | Kubernetes **Secrets** (Phase 4) → AWS parameter store/Secrets Manager (Phase 5) |
| `DATABASE_URL` → localhost | RDS PostgreSQL endpoint (Phase 5) |
| `src/uploads/` local disk | S3 bucket (Phase 5) |
| `PORT=5000` hardcoded in dev | Injected env in the container spec |

Keeping config out of code now is exactly what makes those swaps one-line changes later.