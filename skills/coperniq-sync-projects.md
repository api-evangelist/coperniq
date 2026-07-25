---
name: Sync projects into an external system
description: Incrementally pull Coperniq projects and their work orders into a CRM, ERP, or BI stack using page + date filters.
api: openapi/coperniq-openapi.yml
operations: [list-projects, list-project-work-orders, list-project-forms]
---

# Sync projects into an external system

Use this flow for a recurring, incremental export of Coperniq project data.

## Auth
- Send `x-api-key: <YOUR_API_KEY>` on every request. Base URL is `https://api.coperniq.io/v1`.

## Steps
1. **Page through projects** — `list-projects` (`GET /projects`) with `page`/`page_size`. On the first run omit date filters; on later runs pass `updated_after=<last_sync_timestamp>` to fetch only changed records. Use `order_by` for stable ordering.
2. **Pull each project's work orders** — `list-project-work-orders` (`GET /projects/{projectId}/work-orders`) to capture field-operations state (status, schedule, completion).
3. **Pull structured field data** — `list-project-forms` (`GET /projects/{projectId}/forms`) for inspection/checklist form submissions if you need field-collected data.
4. **Persist a high-water mark** — store the max `updatedAt` you saw so the next run resumes incrementally.

## Conventions & errors
- Pagination is page-number; keep requesting until a short/empty page. Filter with `updated_after`/`updated_before`.
- Rate limits: 100 req/s and 5,000 req/day per key — spread large syncs and watch `X-RateLimit-Remaining`; on `429`, back off and resume.
- Errors return `{ message, code, field? }`. A `401` means the key is missing/invalid; `404` means the record is out of the key user's data scope. See `errors/coperniq-problem-types.yml`.
- For near-real-time sync instead of polling, subscribe to `PROJECT_STATUS_MOVEMENT`, `TASK_STATUS_MOVEMENT`, and `PROJECT_PHASE_COMPLETED` webhooks (see `asyncapi/coperniq-webhooks.yml`).
