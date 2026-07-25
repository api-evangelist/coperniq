---
name: Onboard a new customer project
description: Create an account, add its primary contact, open a project, and schedule the first field work order in Coperniq.
api: openapi/coperniq-openapi.yml
operations: [create-account, create-contact, create-project, list-work-order-templates, create-project-work-order]
---

# Onboard a new customer project

Use this flow to take a new residential/commercial customer from account creation to a scheduled first visit.

## Auth
- Send `x-api-key: <YOUR_API_KEY>` on every request. Base URL is `https://api.coperniq.io/v1`.
- If you have no key yet, generate one with `POST /api-keys` using HTTP Basic auth (see `openapi/coperniq-api-key-openapi.yml`). Keys inherit the creating user's role and data scope and do not expire.

## Steps
1. **Create the account** — `create-account` (`POST /accounts`) with the organization/homeowner `title` and address. Capture the returned `accountId`.
2. **Add the primary contact** — `create-contact` (`POST /contacts`) linked to the account with name, email, and phone.
3. **Open the project** — `create-project` (`POST /projects`) referencing the account. Capture `projectId`. Note: if a project has no `title`, reads fall back to the parent account's title.
4. **Pick a work order template** — `list-work-order-templates` (`GET /work-orders/templates`) to find the right field-visit template.
5. **Schedule the first work order** — `create-project-work-order` (`POST /projects/{projectId}/work-orders`) with `startDate`/`endDate`, `assigneeId`, and `isField: true`.

## Conventions & errors
- Pagination on list calls is page-number (`page`, `page_size`) with `order_by` and `updated_after`/`updated_before` filters.
- Respect rate limits: 100 req/s and 5,000 req/day per key; back off on `429` using `X-RateLimit-Remaining`.
- Errors return `{ message, code, field? }`; on `400` inspect `field` to fix the offending input. See `errors/coperniq-problem-types.yml`.
- There is no idempotency-key contract — guard against duplicate creates on the client side.
- Subscribe to `PROJECT_CREATED` and `TASK_CREATED` webhooks to react downstream (see `asyncapi/coperniq-webhooks.yml`).
