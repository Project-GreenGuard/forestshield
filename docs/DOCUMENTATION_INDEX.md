# ForestShield — documentation index

| Document | Purpose |
|----------|---------|
| [PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md) | Architecture, data flow, what is shipped vs deferred |
| [API_DOCUMENTATION.md](./API_DOCUMENTATION.md) | REST routes, ports, DynamoDB field names, env vars |
| [PRODUCTION_VERIFICATION.md](./PRODUCTION_VERIFICATION.md) | Prod smoke tests, last run results, `verify-production.sh` / `check-prod-health.sh` |
| [AI_PREDICTION_AND_TRAINING_SPEC.md](./AI_PREDICTION_AND_TRAINING_SPEC.md) | Cloud Run `/predict` contract (processing Lambda integration) |
| [WORK_LOG.md](./WORK_LOG.md) | What was built / what you still own — for demos and grading narrative |

**Repos (GitHub org `Project-GreenGuard` unless noted):**

- `forestshield-backend` — processing + API Lambdas, local Docker API
- `forestshield-frontend` — React dashboard
- `forestshield-infrastructure` — AWS (and thin GCP) Terraform
- `forestshield-iot-firmware` — ESP32 / device firmware

This monorepo (`ai-disaster-response-system`) may mirror or aggregate those trees; treat **each repo’s `README.md`** as deploy truth for that component.
