---
name: Quote a customer and collect payment
description: Create and send a quote on an opportunity, then invoice the resulting project and record a payment in Coperniq.
api: openapi/coperniq-openapi.yml
operations: [create-opportunity, create-quote, send-quote, create-invoice, create-invoice-payment]
---

# Quote a customer and collect payment

Use this flow to move an opportunity from quote to paid invoice.

## Auth
- Send `x-api-key: <YOUR_API_KEY>` on every request. Base URL is `https://api.coperniq.io/v1`.

## Steps
1. **Create the opportunity** — `create-opportunity` (`POST /opportunities`) for the inbound deal/service call. Capture `opportunityId`. (Opportunities were formerly named "Requests".)
2. **Create the quote** — `create-quote` (`POST /quotes`) against the opportunity with line items (optionally grouped into named `sections`).
3. **Send the quote** — `send-quote` (`POST /quotes/{quoteId}/send`) to deliver it to the customer; retrieve the PDF via `get-quote-pdf` if you need a copy.
4. **Invoice the work** — `create-invoice` (`POST /invoices`) once the project is underway, referencing the project/account.
5. **Record the payment** — `create-invoice-payment` (`POST /invoices/{invoiceId}/payments`) to log the collected amount.

## Conventions & errors
- Money flows: quotes -> invoices -> payments; bills and bill payments mirror the invoice/payment shape for vendor-side spend.
- Errors return `{ message, code, field? }`; validation failures (`400`) name the offending `field`. See `errors/coperniq-problem-types.yml`.
- No idempotency-key header exists — de-duplicate quote/invoice/payment creates client-side.
- Rate limits: 100 req/s, 5,000 req/day per key; handle `429` with backoff.
- React to `DEAL_STATUS_MOVEMENT` and `PROJECT_STATUS_MOVEMENT` webhooks to keep your CRM/ERP in sync.
