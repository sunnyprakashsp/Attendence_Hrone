# Employee Attendance & Analytics API

FastAPI + MongoDB implementation for the HROne Software Engineer Trainee assignment.

## Run locally

1. Install Python 3.11+ and MongoDB 6.0+.
2. Install dependencies: `pip install -r requirements.txt`
3. Set `MONGO_URI` and `MONGO_DB` in your environment (or a local `.env`, which must not be committed).
4. Start the service from the repository root: `uvicorn app.main:app --port 8000`
5. Open `http://localhost:8000/docs` to inspect the API.

Indexes are created idempotently at application startup. The sample seed script is optional; do not use it against a database containing data you need to keep.

## Before submitting

Run the app against MongoDB 6.0+ and test every endpoint with additional cases: concurrent duplicate punch-ins and punch-outs, overnight shifts, exact 10-minute grace boundary, half-up rounding, no-log employees, ties at leaderboard cutoff, weekends, mid-month joiners, and missing dates. Review `REVIEW.md` and rewrite `DECISIONS.md` in your own words so you can explain the trade-offs in a live walkthrough.

No `.env`, credentials, Dockerfile, or virtual environment should be committed.
