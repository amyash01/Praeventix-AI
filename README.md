This is a [Next.js](https://nextjs.org) project bootstrapped with [`create-next-app`](https://nextjs.org/docs/app/api-reference/cli/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `app/page.js`. The page auto-updates as you edit the file.

This project uses [`next/font`](https://nextjs.org/docs/app/building-your-application/optimizing/fonts) to automatically optimize and load [Geist](https://vercel.com/font), a new font family for Vercel.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/app/building-your-application/deploying) for more details.

## Run locally (Frontend + Backend)

- Frontend:
	- Install dependencies: `npm install`
	- Start dev server: `npm run dev`
	- Open: http://localhost:3000

- Backend (BentoML + FastAPI):
	- (Recommended) create and activate a virtual environment:
		- `python -m venv .venv`
		- PowerShell: `.venv\Scripts\Activate.ps1`
	- Install Python dependencies: `python -m pip install -r backend/requirements.txt`
	- Start the BentoML service from the `backend` folder:
		- `python -m bentoml serve service:BankRiskService --port 8000`
	- Health/UI: http://127.0.0.1:8000

- Notes:
	- Some packages (e.g. `torch`, `xgboost`, `lightgbm`) are large and may take several minutes to download and install.
	- On Windows, if binary wheel installation fails, consider using a Conda environment or installing prebuilt wheels for those packages.

## Project overview

Praeventix EWS (Early Warning System) is a demo Next.js frontend backed by a BentoML + FastAPI service that scores customer risk, provides interventions, and exposes model-driven endpoints used by the UI.

## Prerequisites

- Node.js (recommended v18+)
- npm (bundled with Node)
- Python 3.10+ (3.14 used in this workspace) with `pip`
- Optional: `conda` for easier native binary installs on Windows

## Quickstart

1. Clone the repo and open it in a terminal.
2. Start the frontend and backend in separate terminals (see detailed steps below).

## Frontend (Next.js)

- From the project root:

```powershell
npm install
npm run dev
# Open http://localhost:3000
```

- Notes:
	- If `npm install` fails due to locked files on Windows, remove `node_modules` and try again:
		- PowerShell: `if (Test-Path node_modules) { Remove-Item node_modules -Recurse -Force }`
	- The Next.js dev server provides fast-refresh and logs to the terminal.

## Backend (BentoML + FastAPI)

- Recommended: create a Python virtual environment to avoid polluting global packages:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1   # PowerShell
python -m pip install --upgrade pip
python -m pip install -r backend/requirements.txt
```

- Start the BentoML service from `backend`:

```powershell
cd backend
python -m bentoml serve service:BankRiskService --port 8000
# BentoML UI: http://127.0.0.1:8000
```

- Common issues & tips:
	- Large packages such as `torch`, `xgboost`, and `lightgbm` may take time or fail on Windows; using `conda` often simplifies installation:
		- `conda create -n ews python=3.10 -y && conda activate ews`
		- Then `pip install -r backend/requirements.txt` (or use `conda install` for specific wheels).
	- If scripts are installed to `...\Scripts` and the folder is not on PATH, that's safe for running from the venv; warnings appearing during `pip install` are informational.

## API examples

- List customers (example):

```bash
curl -X POST http://127.0.0.1:8000/list_customers -H "Content-Type: application/json" -d '{"limit":10}'
```

- Analyze a customer risk (example):

```bash
curl -X POST http://127.0.0.1:8000/analyze_customer_risk -H "Content-Type: application/json" -d '{"customer_id":"12345"}'
```

Check the BentoML Swagger UI at `http://127.0.0.1:8000/docs` for full request/response schemas.

## Database and data

- The project uses lightweight SQLite files under `backend/` for demo data (files: `bank_risk.db`, `bank_risk(1).db`, etc.).
- Use `backend/setup_db.py` or `ingest_csv_to_db.py` to refresh or recreate demo data as needed.

## Testing

- Backend tests: run `python -m pytest backend/test_api_endpoints.py` (if `pytest` is installed) or run included test scripts in `backend/`.
- Frontend tests: not included by default — add `jest` or preferred test runner if needed.

## Contributing

- Fork and open a PR with a clear description. Keep changes small and test locally before submitting.

## Troubleshooting

- Frontend shows "Could not fetch dashboard statistics": ensure backend is running at `http://127.0.0.1:8000` and CORS is allowed.
- If `bentoml` import fails, verify the Python environment and `pip install -r backend/requirements.txt` completed successfully.

## Contacts

- For local dev help, open an issue or reach out to the project owner listed in this repo.

## Project details & internals

This section explains the architecture, key components, data layout, and developer workflows so contributors and maintainers can quickly understand and extend the project.

- Architecture
	- Frontend: Next.js app located at the project root `app/` directory. Provides the UI for dashboards, customer risk monitoring, and intervention workflows.
	- Backend: BentoML + FastAPI service in the `backend/` folder. Exposes model-driven APIs consumed by the frontend and provides a BentoML UI at `http://127.0.0.1:8000`.
	- Data: Lightweight demo SQLite databases in `backend/` (e.g. `bank_risk.db`) seeded from CSV files under the repo root.

- Key backend files
	- `backend/service.py`: BentoML service that registers the `BankRiskService` and its HTTP APIs.
	- `backend/feature_store.py`: feature loading and transformations for model inputs.
	- `backend/ml_engine.py`: model ensemble wrapper and prediction logic.
	- `backend/intervention_engine.py`: orchestration of business interventions for risky customers.
	- `backend/setup_db.py` and `backend/ingest_csv_to_db.py`: helper scripts to (re)create or seed demo data.

- Important API endpoints (examples)
	- `POST /list_customers` — return a paginated list of customers used by the UI.
	- `POST /analyze_customer_risk` — compute a detailed risk analysis for a given `customer_id`.
	- `POST /predict_risk` — low-level model risk prediction endpoint.
	- `POST /generate_ai_insights` — (optional) uses the GenAI helper to create textual insights.
	- `GET /get_dashboard_stats` — aggregated statistics for dashboard widgets.
	- `POST /execute_intervention` — trigger business interventions for a customer.

- Data & databases
	- Demo CSVs and historical files are at the repository root: `customers_core.csv`, `payment_history.csv`, `salary_history.csv`, etc.
	- SQLite demo files live in `backend/` and can be refreshed using `backend/setup_db.py` or `backend/ingest_csv_to_db.py`.

- Environment variables
	- `OPENAI_API_KEY` — required only if you plan to enable GenAI features in `backend/genai.py`.
	- Any other secret keys used by the service should be provided via environment variables or a `.env` file loaded by `python-dotenv`.

- Development workflow
	1. Start backend (ensure virtual environment and dependencies installed):
		 ```powershell
		 cd backend
		 python -m bentoml serve service:BankRiskService --port 8000
		 ```
	2. Start frontend in a second terminal:
		 ```powershell
		 npm install
		 npm run dev
		 ```
	3. Use the BentoML UI (`/docs`) to explore API schemas, or call endpoints directly from the frontend.

- Debugging tips
	- If the frontend shows "Could not fetch dashboard statistics", open `backend/backend.log` and look for recent stack traces. Common causes: backend not running, DB file missing, or model files not found.
	- If `pip install -r backend/requirements.txt` fails on Windows for packages like `torch` or `xgboost`, try using a Conda environment or install prebuilt wheels.

- Tests and validation
	- A few backend test scripts exist under `backend/` (e.g., `test_api_endpoints.py`, `test_service.py`). Run them with `python` or `pytest` after installing test dependencies.

---

If you'd like, I can also:

- Add a small architecture diagram (ASCII or Mermaid) describing service interactions.
- Add concrete curl examples for each major endpoint with expected sample responses.
- Add an `env.example` with common environment variables.



