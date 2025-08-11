# bsgx-hiring
## Price Optimization Tool (FastAPI + React)

Full‑stack application for managing products and running simple price optimization and demand forecasting with role‑based access control (RBAC), email verification, and an admin console for roles/permissions.

### Tech Stack
- Backend: FastAPI, SQLAlchemy, Pydantic, JWT (python‑jose), Passlib (bcrypt)
- DB: PostgreSQL (psycopg2)
- Frontend: React (CRA), React Router, Redux Toolkit, MUI
- Emails: Mailtrap SMTP (for email verification)

### Monorepo Structure
```
satvik/
  app/                # FastAPI backend (API, core, DB, schemas)
  client/             # React frontend (CRA)
  scripts/            # Utilities for DB seeding, migrations, role ops
  product_data.csv    # Seed data for products
  requirements.txt
```

## Prerequisites
- Python 3.11+
- Node.js 18+ and npm
- PostgreSQL 13+ running locally (or a managed instance)

## Backend Setup
1) Create and activate a virtualenv
```bash
cd /Users/hbhatia1/Desktop/satvik
python -m venv .venv
source .venv/bin/activate
```

2) Install dependencies
```bash
pip install -r requirements.txt
```

3) Configure environment variables (create `.env` in the repo root)
```env
# Database
DATABASE_URL=postgresql+psycopg2://postgres:postgres@localhost:5432/satvik

# JWT
SECRET_KEY=change_me_to_a_long_random_string
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Frontend URL used in email links
FRONTEND_URL=http://localhost:3000

# Mailtrap SMTP (https://mailtrap.io/)
MAILTRAP_SMTP_HOST=sandbox.smtp.mailtrap.io
MAILTRAP_SMTP_PORT=2525
MAILTRAP_API_KEY=your_mailtrap_api_key
MAIL_FROM_EMAIL=no-reply@example.com
MAIL_FROM_NAME=Price Optimization Tool
```

4) Initialize database and seed data
```bash
# Option A: one-shot reset + create tables + seed
python scripts/seed.py --reset-db

# Option B: stepwise
python -c "from app.db.session import Base, engine; Base.metadata.create_all(bind=engine)"
python scripts/seed.py --migrate            # adds columns like roles.default_role if missing
python scripts/seed.py --set-defaults       # marks admin/supplier/buyer as default roles
python scripts/seed.py                      # run with no args to see help
```

5) Run the backend API
```bash
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

API will be available at `http://127.0.0.1:8000`. Interactive docs at `http://127.0.0.1:8000/docs`.

## Frontend Setup
1) Install dependencies and set API base URL
```bash
cd /Users/hbhatia1/Desktop/satvik/client
npm install

# Optional: create client/.env if you want a custom API URL
echo "REACT_APP_API_BASE_URL=http://127.0.0.1:8000/api/v1" > .env
```

2) Run the frontend
```bash
npm start
```

The app will open at `http://localhost:3000` (CORS for 3000/3001 is preconfigured in the backend).

## Features Overview
### Auth & Email Verification
- Register: `POST /api/v1/auth/register` → sends a verification email via Mailtrap
- Verify: `GET /api/v1/auth/verify-email?token=...`
- Login: `POST /api/v1/auth/token` (OAuth2 password grant, returns JWT)
- Current user: `GET /api/v1/auth/me`
- Dev-only manual verify: `GET /api/v1/auth/verify-manually/{email}`

On registration, users are created with the default role `buyer` and must verify email before login.

### RBAC (Roles & Permissions)
Default permissions include (non-exhaustive):
- product:create, product:read, product:update, product:delete
- forecast:read, optimization:read
- admin:read:users, admin:update:users, admin:assign:roles
- admin:read:roles, admin:create:role, admin:update:role, admin:delete:role

Default roles (seeded):
- admin: all permissions
- supplier: product create/read/update + forecast/optimization read
- buyer: product read only

Admin APIs:
- `GET /api/v1/admin/users` — list users (with roles)
- `POST /api/v1/admin/users/{user_id}/roles` — assign a role
- `DELETE /api/v1/admin/users/{user_id}/roles/{role_id}` — revoke a role
- `GET /api/v1/admin/roles` — list roles (with permissions)
- `POST /api/v1/admin/roles` — create role
- `PUT /api/v1/admin/roles/{role_id}` — update role
- `DELETE /api/v1/admin/roles/{role_id}` — delete role
- `GET /api/v1/admin/permissions` — list permissions

Helper script actions:
```bash
# Verify a user by email (dev convenience)
python scripts/seed.py --verify-email someone@example.com

# Assign a role to a user
python scripts/seed.py --assign-role someone@example.com admin
```

### Products, Optimization, Forecasting
Products API (permission-guarded):
- `GET /api/v1/products/` — list with optional filters `category`, `search`
- `GET /api/v1/products/categories` — unique categories
- `GET /api/v1/products/optimize` — list with computed `optimized_price`
- `GET /api/v1/products/{product_id}` — get one
- `POST /api/v1/products/` — create
- `PUT /api/v1/products/{product_id}` — update
- `DELETE /api/v1/products/{product_id}` — delete
- `GET /api/v1/products/{product_id}/forecast` — 12‑month simulated forecast

Optimization is a simple supply/demand heuristic ensuring price stays above cost. Forecast is simulated for demo purposes.

## Frontend Routes
- Public: `/login`, `/register`, `/verify-email`
- Protected dashboard: `/dashboard`
  - `/dashboard/products`
  - `/dashboard/price-optimization`
  - `/dashboard/settings` (admin‑only via route guard)

## Development Tips
- If login returns 400 “Account not verified”, use the email link from Mailtrap or the dev endpoint/script to verify.
- Ensure `.env` values are correct; backend reads from repo‑root `.env`.
- Update CORS `origins` in `app/main.py` if serving the frontend from a different host/port.
- The backend auto-creates tables on startup; seeding roles/permissions/products is handled via `scripts/seed.py`.

## Troubleshooting
- DB connection errors: verify `DATABASE_URL`, database exists, user has rights.
- 401/403 errors: missing/expired JWT, or account lacks required permission.
- CORS errors: ensure backend `origins` includes the frontend URL.
- Emails not received: verify Mailtrap credentials and inbox; tokens expire in 1 hour.

## License
Proprietary or as defined by the repository owner.


