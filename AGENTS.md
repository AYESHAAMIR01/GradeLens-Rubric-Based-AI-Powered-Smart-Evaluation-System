# AGENTS.md: GradeLens

## Project
GradeLens is a rubric-based AI evaluation platform. Teachers create classes,
assignments and rubrics; students submit PDFs; MCQs are auto-graded; written
answers are drafted by a Groq LLM (LangChain ChatGroq); the teacher reviews and
approves before the student sees anything. Full spec: spec.md (source of truth).

## Stack
- Backend: Python 3.11, FastAPI, SQLAlchemy 2.x, Alembic, PostgreSQL, pytest
- Auth: JWT (role in payload), bcrypt via passlib, TOTP 2FA via pyotp
- AI: langchain-groq ChatGroq; model name from env GROQ_MODEL
- PDF: PyMuPDF (extract), EasyOCR (optional fallback), fpdf2 (reports)
- Frontend: Next.js (App Router) + React + Tailwind + Recharts
- Database: PostgreSQL via DATABASE_URL in .env (no Docker)

## Environment
- Developers work on Windows with PowerShell. Use commands that work there.
- Python virtualenv in backend/.venv (activate: .venv\Scripts\Activate.ps1).

## Repo layout
- spec.md (repo root)
- backend/app/{main.py,config.py,db.py,models/,schemas/,routers/,services/,deps.py}
- backend/tests/
- frontend/src/{app/,components/,lib/api.ts}
- docs/ARCHITECTURE.md
- .agents/skills/<skill-name>/SKILL.md
- README.md, AI_DISCLOSURE.md, AGENTS.md

## Commands
- Backend: `cd backend; uvicorn app.main:app --reload`
- Migrate: `alembic upgrade head`; new: `alembic revision --autogenerate -m "msg"`
- Test: `cd backend; pytest -q`
- Frontend: `cd frontend; npm run dev`; lint: `npm run lint`

## Non-negotiable rules
1. Business logic lives in services/, routers stay thin, schemas are Pydantic.
2. Every endpoint requires auth except /auth/register and /auth/login.
   Enforce role (teacher/student) AND ownership (a teacher sees only their own
   classes/assignments; a student only their own submissions and enrollments).
3. A student must NEVER receive AI draft fields (ai_*) or unapproved teacher
   fields. Only return approved data. Add a test for this.
4. LLM output must be JSON-validated (keys, score range 0..marks, lists of
   strings); retry once; on failure flag the answer for manual grading.
5. Never commit secrets. Use .env (git-ignored) and .env.example.
6. Uploads: PDF only, size-limited, stored under backend/storage/, random filenames.
7. Deadline enforced on the server, not only in the UI.
8. Every schema change goes through an Alembic migration.
9. Add or update at least one pytest per new endpoint (happy path + auth failure).
10. Keep diffs small and focused. Do not refactor unrelated code.

## Workflow
- Before coding: restate the task in 3 to 5 lines and list files to touch.
- After coding: run tests and lint, then summarize changes in <=8 lines.
- Update README/docs when behavior changes. Add an entry to AI_DISCLOSURE.md
  for each chunk (what AI generated, what a human reviewed or changed).
- Commit style: `feat(scope): message`, `fix(scope): message`, `docs: message`.

## Do not
- Add dependencies not listed above without asking.
- Call the LLM for MCQs.
- Expose ai_* columns on student routes.
- Store the JWT in localStorage if httpOnly cookies are practical; otherwise
  document the tradeoff in docs/ARCHITECTURE.md.

## Definition of done (any chunk)
Runs locally, tests pass, no secrets, docs updated, committed.
