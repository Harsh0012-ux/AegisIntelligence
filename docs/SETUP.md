# Local setup

> The original project notes state that the Python, Docker, migration and API commands below were not re-run in a clean checkout when the foundation was written. If a step fails, please open an issue.

## Prerequisites

- Node.js 24 LTS and npm
- Docker with Docker Compose
- Python **3.13**

PostgreSQL runs only through Docker Compose; do not install it natively.

## 1. Configure

```bash
cp .env.example .env
```

Edit `.env`:

1. Set a unique `POSTGRES_PASSWORD` and use the same password inside `DATABASE_URL`.
2. Choose the API port (these steps use `8000`) and set `VITE_API_BASE_URL=http://127.0.0.1:8000`.
3. Optional: set `LLM_API_KEY` to enable the Groq-hosted analyst. Leave blank to use the deterministic fallback.

`.env` is git-ignored. Never commit it.

## 2. Backend environment

Windows (PowerShell):

```powershell
py -3.13 -m venv .venv
.venv\Scripts\activate
python -m pip install -e ".\backend[dev]"
```

macOS / Linux:

```bash
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -e "./backend[dev]"
```

## 3. Database

```bash
docker compose up -d postgres
docker compose ps          # wait until healthy
cd backend
python -m alembic upgrade head
python -m app.seed         # synthetic demo data; safe to re-run
```

## 4. Run

```bash
# backend (from backend/)
python -m uvicorn app.main:app --reload --port 8001

# frontend (new terminal, from repo root)
npm install --prefix frontend
npm run dev --prefix frontend
```

Open `http://localhost:5173`. Verify the API with `GET http://localhost:8001/api/health`. A healthy response is `{"status":"ok","database":"connected"}`.

## 5. Test and build

```bash
cd backend && python -m pytest
npm run build --prefix frontend
```

## Housekeeping

```bash
docker compose logs postgres
docker compose down              # add --volumes only to discard local data
```

## Troubleshooting

- **Frontend cannot reach the API:** check that `VITE_API_BASE_URL` matches the uvicorn port and that `CORS_ORIGINS` includes the frontend origin.
- **Database connection refused:** confirm `docker compose ps` shows PostgreSQL healthy and that the `DATABASE_URL` password matches `POSTGRES_PASSWORD`.
