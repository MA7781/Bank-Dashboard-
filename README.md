# Banking Products Dashboard

A full-stack demo that shows a customer's banking products (savings accounts, credit cards, loans and investments) on one screen.

**Mock data only.** No real bank accounts are connected and no real payments are processed.

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite, React Router, Axios, Recharts |
| Backend | Python, FastAPI, Pydantic, JWT (PyJWT), bcrypt password hashing |
| Database | SQL via SQLAlchemy: SQLite for quick local runs, PostgreSQL in Docker |
| DevOps | Git/GitHub, GitHub Actions CI, Docker Compose, Nginx |

## Features

- Register / login with JWT authentication; role-based access (`customer`, `admin`)
- Dashboard overview: total products, active products, pending applications, total balance, chart by category
- Product cards for Savings, Credit Card, Loan and Investment, with masked account/card numbers
- Search by name, filter by category and status (Active / Pending / Closed), sort by balance or date
- View Details modal with recent transactions
- Apply for a product (creates a **Pending** product); an admin can approve it from **Settings**
- Transactions page with paging
- Responsive layout: desktop, tablet and mobile (sidebar becomes a slide-out menu)
- Swagger API docs at `/docs`

## Project structure

```
banking-dashboard/
├── frontend/            React app (src/components, pages, services, hooks, utils)
├── backend/
│   ├── app/
│   │   ├── api/         routes (auth, banking) and auth dependencies
│   │   ├── core/        settings, password hashing, JWT
│   │   ├── database/    engine and session
│   │   ├── models/      SQLAlchemy tables
│   │   ├── schemas/     Pydantic request/response models
│   │   └── services/    business logic and demo seed data
│   ├── db/schema.sql    PostgreSQL schema (same tables as the models)
│   └── tests/           pytest integration tests
├── nginx/default.conf   serves the React build and proxies /api to the backend
├── .github/workflows/   CI pipeline
├── docker-compose.yml
└── .env.example
```

## Demo logins

| Role | Email | Password |
|---|---|---|
| Customer | demo@bank.com | Demo@1234 |
| Admin | admin@bank.com | Admin@1234 |

## Run locally (Windows PowerShell)

You need Python 3.11+ and Node.js 20+.

**1. Backend** (first terminal)

```powershell
cd banking-dashboard\backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
pytest -q                                   # optional: run the tests
uvicorn app.main:app --reload --port 8000
```

If PowerShell blocks the activate script, run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` once.
The API is now at http://127.0.0.1:8000 and Swagger docs at http://127.0.0.1:8000/docs.
A `banking.db` SQLite file with demo data is created automatically on first start.

**2. Frontend** (second terminal)

```powershell
cd banking-dashboard\frontend
npm install
npm run dev
```

Open http://localhost:5173 and sign in with a demo login.

## Run with Docker (PostgreSQL + Nginx)

```powershell
cd banking-dashboard
Copy-Item .env.example .env        # then edit .env: set a real SECRET_KEY and password
docker compose up --build
```

Open http://localhost:8080. Stop with `Ctrl+C`, and remove the database volume with `docker compose down -v`.

## API overview

| Method | Path | Notes |
|---|---|---|
| POST | `/api/auth/register` | Creates a customer, returns a JWT |
| POST | `/api/auth/login` | Returns a JWT |
| GET | `/api/auth/me` | Current user |
| GET | `/api/products` | Query: `category`, `status`, `q`, `sort` |
| GET | `/api/products/{id}` | Product details |
| POST | `/api/applications` | Apply for a product (creates it as Pending) |
| GET | `/api/applications` | Your applications |
| GET | `/api/dashboard/stats` | Dashboard figures and per-category totals |
| GET | `/api/transactions` | Query: `product_id`, `limit`, `offset` |
| GET | `/api/admin/applications` | Admin only |
| POST | `/api/admin/applications/{id}/approve` | Admin only |

Send the token as `Authorization: Bearer <token>`.

## Security notes

- Passwords are hashed with bcrypt; they are never stored or returned in plain text.
- JWTs are signed with `SECRET_KEY` (set your own in `.env`) and expire after 60 minutes.
- All database access uses SQLAlchemy with bound parameters, which prevents SQL injection (there is a test for this).
- Inputs are validated with Pydantic; unexpected server errors return a generic message and are logged server-side.
- Account and card numbers are masked in every API response (only the last 4 digits).
- Users can only see their own products and transactions; admin routes check the user's role.
- CORS only allows the origins listed in `CORS_ORIGINS`.

## Git and GitHub

```powershell
cd banking-dashboard
git init -b main
git add .
git commit -m "Initial commit: banking products dashboard"
```

Create an empty repository on github.com (no README), then:

```powershell
git remote add origin https://github.com/<your-username>/banking-dashboard.git
git push -u origin main
```

GitHub Actions will then run `.github/workflows/ci.yml` on every push and pull request: backend lint and tests, frontend build, and a Docker Compose smoke test.

## Not included

MongoDB, Redis and AWS/Azure deployment were left out to keep the stack to React, Python, SQL and Git/GitHub. The Docker setup is a good base for adding them later.

The database tables are created automatically at startup. For a real deployment you would add Alembic migrations.
