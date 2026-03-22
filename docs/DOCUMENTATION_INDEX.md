# ForestShield — documentation index

Start with **[PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md)** for what the system does, then use the table below as needed.

| Document | Purpose |
|----------|---------|
| [PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md) | Product scope, components, data flow, repository layout, March 2026 as-built addendum |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | System architecture diagram and component details |
| [API_DOCUMENTATION.md](./API_DOCUMENTATION.md) | REST routes, env vars, DynamoDB fields, production verification commands |
| [QUICK_START.md](./QUICK_START.md) | Run backend + frontend locally, test data, Lambda zip / Terraform pointers |
| [PRODUCTION_VERIFICATION.md](./PRODUCTION_VERIFICATION.md) | Prod smoke tests, how to run `verify-production.sh` / `check-prod-health.sh` |
| [DEMONSTRATION_GUIDE.md](./DEMONSTRATION_GUIDE.md) | **How to demo everything you built** (video / live / marker walkthrough) |
| [AI_PREDICTION_AND_TRAINING_SPEC.md](./AI_PREDICTION_AND_TRAINING_SPEC.md) | Cloud Run `/predict` JSON contract (processing Lambda) |
| [MODEL_TRAINING.md](./MODEL_TRAINING.md) | Offline training methodology (gradient boosting / Ontario FIRMS–based dataset) |
| [TEAM_SETUP_GUIDE.md](./TEAM_SETUP_GUIDE.md) | Org, repos, access, and team onboarding |
| [VP_SOFTWARE_MODEL.md](./VP_SOFTWARE_MODEL.md) | VP / capstone software model (course artifact) |
| [DOCUMENTATION_FORMATTING_GUIDE.md](./DOCUMENTATION_FORMATTING_GUIDE.md) | How we format and structure project documentation |

**Repos** (GitHub org `Project-GreenGuard` unless noted):

| Repo | Role |
|------|------|
| `forestshield-backend` | Processing + API Lambdas, Docker local API |
| `forestshield-frontend` | React dashboard |
| `forestshield-infrastructure` | AWS (and GCP) Terraform; see `.github/workflows/` for deploy automation |
| `forestshield-iot-firmware` | ESP32 firmware; see `docs/DEVICE_OPS.md` in that repo |
| `forestshield-ai` (optional) | Offline ML training scripts; see [MODEL_TRAINING.md](./MODEL_TRAINING.md) |

**Not in this folder (removed earlier as duplicate):** `SETUP_GUIDE`, `DEVELOPMENT_GUIDE`, `STAGING_AND_CICD` — use **QUICK_START**, per-repo READMEs, and **infrastructure** workflows instead.

---

**Documentation set last reviewed:** March 2026 — keep **`API_DOCUMENTATION`**, **`ARCHITECTURE`**, **`PROJECT_OVERVIEW`** §18, and **`PRODUCTION_VERIFICATION`** in sync when you change Lambdas, routes, or infra.
