# What Runs Today Checklist
> Generated: 2026-02-22
> Scope: `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI`

## 1. Current Runability Summary

| Area | Current State | Notes |
|---|---|---|
| Root `docker-compose.yml` | Partial | Most service contexts exist, but `web-console` context is missing and some monitoring mount paths are missing. |
| `nexus-office-suite/docker-compose.yml` | Partial | Core backend services exist; `forms/tasks/notes` and some monitoring/init paths are missing. |
| `docker-compose.complete.yml` | Not runnable end-to-end | Many declared services have no source directories. |
| Standalone core services | Mixed | `nexus-engine`, `services/crm`, `services/finance`, `services/ai`, `services/idaas`, `services/time-attendance` have source entrypoints. |
| Standalone office-suite services | Mostly runnable with dependencies | API gateway, auth, writer, sheets, slides, drive, mail, calendar, chat, meet, notification, collaboration have source entrypoints. |

## 2. Runnable Today (Minimal Path)

### A. Core stack minus missing `web-console`

- [ ] Start infrastructure and core backend services from root compose:

```bash
cd /Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI
docker-compose up -d postgres redis redpanda minio nexus-engine crm-service finance-service ai-service documents-service hr-service inventory-service projects-service marketing-service support-service google-workspace-integration odoo-integration zoho-integration
```

- [ ] Validate health:

```bash
curl -f http://localhost:8080/health
curl -f http://localhost:8081/health
curl -f http://localhost:8082/health
curl -f http://localhost:8086/health
```

### B. Office suite backend stack (implemented services only)

- [ ] Start implemented services only:

```bash
cd /Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite
docker-compose up -d postgres redis kafka zookeeper minio elasticsearch api-gateway auth-service notification-service collaboration-service writer-service sheets-service slides-service mail-service calendar-service drive-service meet-service
```

- [ ] Validate health:

```bash
curl -f http://localhost:8000/health
curl -f http://localhost:3001/health
curl -f http://localhost:8091/health
curl -f http://localhost:8092/health
curl -f http://localhost:8093/health
curl -f http://localhost:8094/health
curl -f http://localhost:8095/health
curl -f http://localhost:8096/health
```

## 3. Blockers and Required Fixes

### A. Root compose blockers (`/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/docker-compose.yml`)

- [ ] Fix missing service context:
  - `web-console` context points to `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/web-console` (does not exist).
  - Choose one:
    - Repoint to existing `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/web`, or
    - Remove/disable `web-console` service.

- [ ] Fix missing monitoring mounts:
  - Missing file: `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/ops/prometheus/prometheus.yml`
  - Missing directory: `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/ops/grafana/datasources`

- [ ] Optional cleanup:
  - Remove obsolete `version` field in compose file.
  - Review `support-service` healthcheck retries value (`53`) if unintentional.

### B. Office suite compose blockers (`/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/docker-compose.yml`)

- [ ] Remove or implement services declared but absent:
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/backend/forms-service`
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/backend/tasks-service`
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/backend/notes-service`

- [ ] Create missing init/monitoring paths referenced by compose:
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/database/init`
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/monitoring/prometheus.yml`
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/monitoring/grafana/dashboards`
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/monitoring/grafana/datasources.yml`

### C. Complete compose blockers (`/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/docker-compose.complete.yml`)

- [ ] Resolve missing build contexts (must either create or remove):
  - `services/vas/sms`
  - `services/vas/whatsapp`
  - `services/vas/telegram`
  - `services/vas/messenger`
  - `services/voice-switch`
  - `services/email`
  - `services/contact-center`
  - `services/contact-center/frontend`
  - `services/ecommerce`
  - `services/ecommerce/frontend`
  - `services/devops`
  - `services/devops/frontend`
  - `services/api-manager`
  - `services/dbaas`
  - `services/dbaas/frontend`
  - `services/webhosting`
  - `services/cdn`
  - `services/ipaas`
  - `services/ipaas/frontend`
  - `services/bpa`
  - `services/bpa/frontend`
  - `services/designer`
  - `services/designer/frontend`
  - `services/ai-agents`
  - `services/promptql`
  - `services/mmp`

### D. Test/CI blockers

- [ ] Fix AI test agent IDs in `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/tests/test_ai_agents.sh` to match IDs in `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/services/ai/config/agents.json`.
- [ ] Fix office-suite workflow paths:
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/.github/workflows/ci.yml` references `services/*` paths not present.
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/.github/workflows/test.yml` references `services/auth` and `docker-compose.test.yml` (missing).
- [ ] Add missing performance scenario files referenced by docs:
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/tests/performance/scenarios/websocket-concurrent.js`
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/nexus-office-suite/tests/performance/scenarios/file-upload.js`

## 4. Validation Checklist (After Fixes)

- [ ] `docker-compose config --services` succeeds in root.
- [ ] `docker-compose config --services` succeeds in `nexus-office-suite`.
- [ ] `docker-compose -f docker-compose.complete.yml config --services` succeeds.
- [ ] Root health endpoints return 200 (`8080`, `8081`, `8082`, `8086`).
- [ ] Office suite health endpoints return 200 (`8000`, `3001`, `8091-8096`, `8098`).
- [ ] Root scripts run without missing-path errors:
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/scripts/schema-pull.sh`
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/scripts/codegen-all.sh`
  - `/Users/AbiolaOgunsakin1/BOS/BAC-BOS-AI/scripts/test-all.sh`

## 5. Recommended Execution Order

1. Fix root compose missing paths (`web-console`, monitoring files).
2. Stabilize `nexus-office-suite/docker-compose.yml` by removing or implementing forms/tasks/notes and missing monitoring/init paths.
3. Decide target scope for `docker-compose.complete.yml`:
   - Option A: Trim to implemented services.
   - Option B: Implement all missing service directories.
4. Correct CI workflow path assumptions and missing test assets.
