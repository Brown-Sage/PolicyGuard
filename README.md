# PolicyGuard

PolicyGuard is a full-stack compliance scanning app for checking employee datasets against policy documents. Users can sign in, upload a policy PDF and employee CSV, extract structured rules from the policy, run a compliance scan, and review detected violations plus scan history.

The project is organized as a FastAPI backend and a Vite React frontend.

## Features

- User registration and login with JWT-based authentication.
- PDF policy upload with text extraction through `pdfplumber`.
- Rule extraction from policy text using a two-tier approach:
  - Gemini AI through `google-genai` when `GEMINI_API_KEY` is configured.
  - Regex-based fallback when Gemini is unavailable or not configured.
- CSV employee dataset import with `pandas`.
- Type-aware compliance engine that evaluates active rules against employee records without using `eval`.
- Violation storage, scan summaries, and user-scoped scan history.
- React dashboard with PDF/CSV upload, summary cards, violations table, dark mode, and responsive sidebar.
- Sample policy PDFs and sample CSV data included for local testing.

## Tech Stack

### Backend

- Python
- FastAPI
- Uvicorn
- SQLAlchemy async ORM
- SQLite through `aiosqlite`
- Pydantic settings
- JWT auth with `python-jose`
- Password hashing with `bcrypt`
- PDF parsing with `pdfplumber`
- CSV parsing with `pandas`
- Optional Gemini integration with `google-genai`

### Frontend

- React 19
- Vite 7
- Tailwind CSS
- ESLint
- Browser `localStorage` for auth token, username, and theme preference

## Project Structure

```text
.
|-- backend/
|   |-- main.py                  # FastAPI app, CORS, router registration, DB startup
|   |-- config.py                # Environment-backed settings
|   |-- database.py              # Async SQLAlchemy engine/session setup
|   |-- models/                  # SQLAlchemy database models
|   |-- schemas/                 # Pydantic response/request schemas
|   |-- routers/                 # API route modules
|   |-- services/                # Auth, PDF, CSV, AI, regex, and scan logic
|   |-- requirements.txt         # Python dependencies
|   |-- run.sh                   # Backend helper script
|   `-- policyguard.db           # Local SQLite database
|-- PolicyGuard/
|   |-- src/
|   |   |-- App.jsx              # Frontend app shell, auth gate, navigation
|   |   `-- components/          # Dashboard, auth, sidebar, tables, summaries
|   |-- package.json             # Frontend scripts and dependencies
|   |-- tailwind.config.js       # Tailwind configuration
|   |-- Global_Policy_V2.pdf     # Sample policy PDF
|   |-- Global_Policy_V3.pdf     # Sample policy PDF
|   `-- Policy_Compliance_Dataset_Updated.csv
`-- README.md
```

## How the Scan Works

1. The frontend calls `/api/scan/reset` to clear the current working scan data while preserving scan history.
2. The policy PDF is uploaded to `/api/policies/upload`.
3. The backend extracts PDF text with `pdfplumber`.
4. Policy rules are generated with Gemini if available, otherwise with the regex fallback.
5. The employee CSV is uploaded to `/api/employees/batch`.
6. The backend normalizes CSV columns into employee model fields.
7. `/api/scan/trigger` evaluates employees against all active rules.
8. Violations are saved and a scan log is written.
9. The frontend displays compliance score, record count, rule count, violations, and scan history.

## Local Setup

### 1. Backend

Run the backend from the `backend` directory so the default SQLite path resolves to `backend/policyguard.db`.

```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Create an optional `.env` file in `backend/`:

```env
PROJECT_NAME=PolicyGuard
DATABASE_URL=sqlite+aiosqlite:///./policyguard.db
GEMINI_API_KEY=your_gemini_api_key
SECRET_KEY=replace-this-with-a-long-random-secret
```

`GEMINI_API_KEY` is optional. Without it, the app still works using regex rule extraction.

Start the API:

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

The API will be available at:

- `http://localhost:8000`
- `http://localhost:8000/docs` for Swagger/OpenAPI docs

### 2. Frontend

Use a Node.js version compatible with Vite 7. The installed Vite package declares `^20.19.0 || >=22.12.0`.

```bash
cd PolicyGuard
npm install
```

Create an optional `.env` file in `PolicyGuard/` if your backend URL is different:

```env
VITE_API_URL=http://localhost:8000/api
```

Start the frontend:

```bash
npm run dev
```

Open the Vite URL shown in the terminal, usually:

```text
http://localhost:5173
```

## Sample Inputs

The frontend directory includes files you can use for a demo scan:

- `PolicyGuard/Global_Policy_V2.pdf`
- `PolicyGuard/Global_Policy_V3.pdf`
- `PolicyGuard/Policy_Compliance_Dataset_Updated.csv`

The CSV loader recognizes these source columns and maps them into backend employee fields:

- `Employee_ID`
- `Name`
- `Working_Days`
- `Target_Sales`
- `Actual_Sales`
- `Customer_Satisfaction_Score`
- `Policy_Compliance`
- `Low_Working_Days`
- `Target_Not_Met`
- `Low_Customer_Satisfaction`
- `Non_Compliance_Reason`
- `Month`

## API Overview

Base API URL: `http://localhost:8000/api`

| Method | Route | Purpose |
| --- | --- | --- |
| `POST` | `/auth/register` | Create user and return access token |
| `POST` | `/auth/login` | Authenticate user and return access token |
| `POST` | `/policies/upload` | Upload PDF, extract text, create policy rules |
| `GET` | `/policies/` | List uploaded policies with rules |
| `POST` | `/employees/` | Create one employee |
| `POST` | `/employees/batch` | Upload employee CSV |
| `GET` | `/employees/` | List employees |
| `GET` | `/rules/` | List active rules, optionally filtered by policy |
| `POST` | `/scan/reset` | Clear policies, rules, employees, and violations |
| `POST` | `/scan/trigger` | Run compliance scan |
| `GET` | `/scan/logs` | List scan logs |
| `DELETE` | `/scan/logs` | Clear scan logs |
| `GET` | `/violations/` | List violations |

Authenticated frontend requests send the JWT as:

```http
Authorization: Bearer <token>
```

## Data Model

The core database tables are:

- `users`: registered app users.
- `policies`: uploaded policy metadata and extracted PDF text.
- `rules`: structured rules extracted from policy text.
- `employees`: imported employee records from CSV.
- `violations`: saved rule violations for employees.
- `scan_logs`: scan history, optionally scoped to a user.

## Useful Commands

Backend:

```bash
cd backend
source venv/bin/activate
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

Frontend:

```bash
cd PolicyGuard
npm run dev
npm run build
npm run lint
```

## Notes and Current Limitations

- The backend currently allows all CORS origins. Restrict this before production deployment.
- `SECRET_KEY` has a development default. Set a strong secret in production.
- SQLite is used for local development. A production deployment should use a managed database and migrations.
- `/api/scan/reset` clears current policies, rules, employees, and violations before a new isolated scan. Scan logs are intentionally preserved.
- The Policy Documents screen is marked as an MVP/in-development repository view.
- `backend/setup_db.sh` appears to reference older rule fields and is not needed for the current upload-based workflow.
