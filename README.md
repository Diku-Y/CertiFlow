# CertiFlow
A realistic digital-banking simulation with real-time fraud detection.
**Stack:** React 19 + Vite (frontend) · Node/Express 5 + MongoDB (API) · Redis (sessions, behaviour state, locks, rate limits, caches, live events) · Python/scikit-learn (fraud model) · Docker Compose.

## Run it

```bash
cp .env.example .env            # set JWT_SECRET (e.g. openssl rand -hex 32)
docker compose up --build       # mongo (replica set) + redis + ml + backend + frontend
docker compose exec backend npm run seed      # demo data (re-run any time; it wipes the DB)
```

Open http://localhost:5173

| Role | Login | Password |
|---|---|---|
| Customer | aarav@certiflow.test (also meera@, rohan@, priya@, karthik@ … `@certiflow.test`) | `Customer@123` — transaction PIN `482916` |
| Fraud analyst | analyst@certiflow.test | `Analyst@12345` |
| Admin | admin@certiflow.test | `Admin@12345` |

Two applications (neha.verma@, siddharth@) wait for admin approval. Use the flask icon in the customer top bar
("Simulation lab") to pay from a brand-new device or another city and watch the fraud engine react.

Without Docker: `backend/` → `npm install && npm run dev`; `frontend/` → `npm install && npm run dev`;
`ml/` → `pip install -r requirements.txt && python train_model.py && python app.py`; MongoDB must be a replica set
(or set `MONGO_TRANSACTIONS=false` for a standalone mongod).

## What a payment goes through
PIN check (lock-out after 3 wrong) → Redis idempotency/lock on the sender's account → daily-limit & balance checks →
28 behavioural features from Redis (profile, 7-day attempt window, recipient fan-in) → ML score + business rules →
ALLOW / ALLOW+notify / HOLD / BLOCK → atomic MongoDB transaction (ledger, balances, alert, notifications) →
Redis state update, live push to browsers, cache invalidation, audit log. If the ML service is down, rules decide and
large payments are held (fail-safe). If Redis is down, everything falls back to MongoDB / in-process logic.

## Tests
```bash
cd backend && npm test          # feature-engine parity vs Python, Redis layer, risk engine, validators
cd ml && python tests/test_ml.py   # scoring, validation, drift, retrain → promote → rollback
```
`npm run typecheck` runs `tsc`; `npm run build` for the frontend is `vite build` (type-checking is separate: `npm run typecheck`).
=======
CertiFlow is a full-stack digital banking simulation featuring real-time fraud detection using machine learning, secure transactions, behavioural analysis, and role-based dashboards for customers, fraud analysts, and administrators. Built with React, Node.js, MongoDB, Redis, and Python.
>>>>>>> 29eb8a630417006f891069f9d7e3826eb7c38222
