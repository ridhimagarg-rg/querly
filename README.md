# Querly

Querly is a FastAPI OPD appointment, queue and e-slip system with a static
JavaScript frontend. The production database is PostgreSQL, hosted locally or
in Supabase.

## Local development

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
Copy-Item .env.example .env
uvicorn app.main:app --reload --port 8000
```

Serve `frontend/` with any static server (for example
`python -m http.server 5500 --directory frontend`) and open
`http://localhost:5500`. Local frontend pages automatically use
`http://127.0.0.1:8000/api`.

## Supabase setup

1. Create a Supabase project and copy its **direct PostgreSQL connection
   string** (Project Settings → Database → Connection string). Use the URI
   form, not the Supabase REST URL.
2. In Supabase SQL Editor, paste and run [`sql/schema.sql`](sql/schema.sql).
   This creates every ORM table, including `patient_profiles`, doctor
   schedule/qualification/experience/bio fields, and all foreign keys.
3. Set these backend environment variables:

```dotenv
DATABASE_URL=postgresql://postgres:<database-password>@db.<project-ref>.supabase.co:5432/postgres?sslmode=require
SECRET_KEY=<long-random-secret>
ACCESS_TOKEN_EXPIRE_MINUTES=1440
CORS_ORIGINS=https://frontend.example.com,http://localhost:5500
```

`postgres://` and `postgresql://` Supabase URLs are both accepted. The
backend selects the psycopg SQLAlchemy driver and adds `sslmode=require` for
Supabase URLs when it is not already present. Never commit `.env`.

## Seed data

After the schema exists and `DATABASE_URL` is set:

```powershell
cd backend
python -m app.seed
```

The seed command is safe to use for a fresh development database. Change the
generated/default credentials before sharing a deployed environment.

## Deployment

### Backend (Render/Railway-style)

Use `backend` as the service root, install from `requirements.txt`, and set
the environment variables above. The start command is:

```text
uvicorn app.main:app --host 0.0.0.0 --port $PORT
```

Set `CORS_ORIGINS` to the exact frontend origin(s), comma-separated. The API
does not enable wildcard CORS; localhost origins are allowed for development.

### Frontend

Deploy `frontend/` to any static host. A deployed browser uses same-origin
`/api` by default, which requires the host/reverse proxy to route `/api` to
the backend. If frontend and API are on different hosts, configure the API
before loading `shared/js/api.js`:

```html
<script>
  window.QUERLY_API_BASE = "https://api.example.com/api";
</script>
<script src="shared/js/config.js"></script>
<script src="shared/js/api.js"></script>
```

Alternatively set `window.QUERLY_CONFIG = { apiBase: "..." }` in
`shared/js/config.js`. Users may override it at runtime with the server
configuration dialog; the value is stored in `localStorage`.

## Project layout

- `backend/app/` — FastAPI routes, SQLAlchemy models, auth and seed logic
- `frontend/` — static patient, doctor and admin dashboards
- `sql/schema.sql` — PostgreSQL/Supabase schema
