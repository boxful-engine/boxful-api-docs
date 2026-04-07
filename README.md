# Boxful API v1

Base URL: `https://<your_subdomain>.boxful.io/api/v1`

The Boxful API is organized around REST. Our API has predictable resource-oriented URLs, accepts form-encoded request bodies, returns JSON-encoded responses, and uses standard HTTP response codes, authentication, and verbs.

For `PATCH` and `POST` requests it is mandatory to include the `Content-Type` header with `application/json` as value.

```
-H "Content-Type: application/json"
```

## Authentication

The Boxful API uses API access tokens to authenticate requests. You can request your API access tokens to your Boxful's account representative.

Please be sure to keep your API access tokens secure. Do not share your secret API access tokens in publicly accessible areas such as GitHub, client-side code, and so forth.

All the requests must be authenticated via bearer. To do so use:

```
-H "Authorization: Bearer <your-access-token>"
```

## Rate limiting

The Boxful API has a limit of requests per minute per account. If you exceed it you will receive the error response `HTTP 429 Too Many Requests`.

Limits vary depending on the account's plan.

| Account Plan | API rate limit |
|---|---|
| **ADVANCE** | `50` |
| **SCALE** | `75` |
| **CUSTOM** | Contact your Boxful's account representative |

## Pagination

Collection endpoints return paginated results:

| Field | Notes |
|---|---|
| `data` | Array of resource objects |
| `meta` | Collection metadata: `total`, `per_page`, `pages` |
| `links` | Pagination links: `self`, `first`, `prev`, `next`, `last` |

## Resources

- [Customers](reference/customers.md)
- [Subscriptions](reference/subscriptions.md)
- [Subscription Carts](reference/subscription-carts.md)
- [Invoices](reference/invoices.md)
- [Payments](reference/payments.md)
- [Refunds](reference/refunds.md)
- [Payment Instructions](reference/payment-instructions.md)
- [Base Plans](reference/base-plans.md)
- [Plans](reference/plans.md)
- [Orders](reference/orders.md)
- [Credit Cards](reference/credit-cards.md)
- [Webhooks](reference/webhooks.md)

## How-To Guides

- [Iframe Subscription Checkout](how-to-guides/iframe-subscription-checkout.md) — Embed Boxful's payment forms in your application via iframe
