# Employee Attendance & Analytics API

FastAPI + MongoDB backend for an HR attendance system with punch-in/out, regularization, and analytics.

## Setup & Run

```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
# Set MONGO_URI and MONGO_DB via environment variables or a .env file
uvicorn app.main:app --port 8000
```

MongoDB **6.0+** required. All indexes are created idempotently at startup.

## What is implemented

- **Part A:** Reviewed starter code, found and fixed 23 defects (see `REVIEW.md`).
- **Part B:** All endpoints implemented — CRUD, punch-in/out, regularization (PATCH with history), and four analytics aggregation pipelines (employee monthly, department summary, late leaderboard, department trend with gap-filling and 7-day moving average).
- **Part C:** Explain endpoint and `DECISIONS.md` completed.

## Notes

- `MongoClient(tz_aware=True)` ensures all datetimes from PyMongo are UTC-aware.
- Rounding uses Python `Decimal(ROUND_HALF_UP)` everywhere (R8) — MongoDB's `$round` uses banker's rounding, so analytics pipelines delegate final rounding to Python.
- Concurrent safety: unique compound index on `(emp_code, date)` with `DuplicateKeyError` handling for punch-in; atomic `find_one_and_update` with `punch_out: None` guard for punch-out; optimistic concurrency via `history.$size` check for regularization.
