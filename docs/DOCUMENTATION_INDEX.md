# ForestShield — documentation index

Start with **[PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md)** for what the system does, then use the table below as needed.

| Document | Purpose |
|----------|---------|
| [PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md) | Product scope, components, data flow, repository layout, March 2026 as-built addendum |
| [API_DOCUMENTATION.md](./API_DOCUMENTATION.md) | REST routes, env vars, DynamoDB fields, production verification commands |
| [QUICK_START.md](./QUICK_START.md) | Run backend + frontend locally, test data, Lambda zip / Terraform pointers |
| [PRODUCTION_VERIFICATION.md](./PRODUCTION_VERIFICATION.md) | Prod smoke tests, how to run `verify-production.sh` / `check-prod-health.sh` |
| [AI_PREDICTION_AND_TRAINING_SPEC.md](./AI_PREDICTION_AND_TRAINING_SPEC.md) | Cloud Run `/predict` JSON contract (processing Lambda) |
| [MODEL_TRAINING.md](./MODEL_TRAINING.md) | Offline training methodology (gradient boosting / Ontario FIRMS–based dataset) |
| [WORK_LOG.md](./WORK_LOG.md) | Shipped milestones and demo narrative |

**Repos** (GitHub org `Project-GreenGuard` unless noted):

| Repo | Role |
|------|------|
| `forestshield-backend` | Processing + API Lambdas, Docker local API |
| `forestshield-frontend` | React dashboard |
| `forestshield-infrastructure` | AWS (and GCP) Terraform; see `.github/workflows/` for deploy automation |
| `forestshield-iot-firmware` | ESP32 firmware; see `docs/DEVICE_OPS.md` in that repo |
| `forestshield-ai` (optional) | Offline ML training scripts; see [MODEL_TRAINING.md](./MODEL_TRAINING.md) |

**Removed (March 2026 cleanup):** Overlapping or stale guides (`ARCHITECTURE`, `SETUP_GUIDE`, `DEVELOPMENT_GUIDE`, `TEAM_SETUP_GUIDE`, `STAGING_AND_CICD`, `DOCUMENTATION_FORMATTING_GUIDE`, `VP_SOFTWARE_MODEL`). Architecture and setup are consolidated into **PROJECT_OVERVIEW** and **QUICK_START**; CI/CD specifics live in **infrastructure workflows**.
