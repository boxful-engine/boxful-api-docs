# Multi-Item Subscriptions

Migrate from legacy top-level `plan_id` writes to nested `/items` routes without breaking existing single-plan integrations.

- [Prerequisites](#prerequisites)
- [Step 1: Enable multi-plan subscriptions](#step-1-enable-multi-plan-subscriptions)
- [Step 2: Read `items[]` on subscriptions](#step-2-read-items-on-subscriptions)
- [Step 3: Adopt nested PATCH for plan changes](#step-3-adopt-nested-patch-for-plan-changes)
- [Step 4: Adopt nested POST to append items](#step-4-adopt-nested-post-to-append-items)
- [Step 5: Flip to multi-item write mode](#step-5-flip-to-multi-item-write-mode)
- [Step 6: Bootstrap checkout lines on customer create](#step-6-bootstrap-checkout-lines-on-customer-create)
- [Step 6b: Bootstrap checkout lines on cart PATCH](#step-6b-bootstrap-checkout-lines-on-cart-patch)
- [Step 7: Stop PATCHing top-level `plan_id`](#step-7-stop-patching-top-level-plan_id)
- [Step 8: Report metered consumption per item](#step-8-report-metered-consumption-per-item)
- [Notes](#notes)

## Prerequisites

- A Boxful account with API access
- At least one active subscription with a plan
- Boxful enables multi-plan subscriptions for your account (contact your representative)

While the account stays in default **single-item** mode, legacy `PATCH /api/v1/subscriptions/:id` with top-level `plan_id` continues to work. You can adopt nested routes early without changing that contract.

## Step 1: Enable multi-plan subscriptions

Boxful enables multi-plan subscriptions per account. Once enabled:

- `GET /api/v1/subscriptions/:id` and the list endpoint include additive `items[]`
- Nested `/items` routes become available

When multi-plan subscriptions are not enabled, `items[]` is omitted from responses and nested writes return **403**.

## Step 2: Read `items[]` on subscriptions

Fetch a subscription and persist each item's `id` — you need it for nested PATCH/DELETE:

```shell
curl -s https://<subdomain>.boxful.io/api/v1/subscriptions/14039 \
  -H "Authorization: Bearer $TOKEN"
```

The response includes top-level `plan_id` (unchanged for backward compatibility) and, when multi-plan subscriptions are enabled, an `items` array:

```json
{
  "id": 14039,
  "plan_id": 1,
  "items": [
    {
      "id": 12,
      "object": "SubscriptionItem",
      "plan_id": 1,
      "plan_quantity": 0.0,
      "delivery_price_item_id": null,
      "client_external_reference": null
    }
  ]
}
```

Nested customer embeds and webhook payloads do **not** include `items[]`. Use the subscription endpoints above.

## Step 3: Adopt nested PATCH for plan changes

On a single-item subscription, change the plan through the item route instead of top-level `plan_id`:

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/subscriptions/14039/items/12 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "plan_id": 88 }'
```

This processes a new invoiceable cart (`current_version`), same as the legacy parent PATCH — while the account is still in **single-item** write mode. You can also set `client_external_reference`: a stored reference, unique per account, for your own reconciliation.

See [Update a subscription item](../reference/subscription-items.md#update-a-subscription-item) for status codes.

## Step 4: Adopt nested POST to append items

To add a plan line, use:

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/subscriptions/14039/items \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "plan_id": 99,
    "client_external_reference": "kid-b"
  }'
```

While the account is still in **single-item** write mode, a second POST returns **409** — only one item is allowed until Boxful enables multi-item write mode (Step 5). The first item on an itemless subscription is allowed in single-item write mode.

Duplicate `plan_id` values on the same subscription are allowed after the flip (e.g. two children on the same tuition plan).

See [Create a subscription item](../reference/subscription-items.md#create-a-subscription-item).

## Step 5: Flip to multi-item write mode

When your integration is ready for 2+ items, ask Boxful to enable multi-item write mode (one-way). Requirements:

- Multi-plan subscriptions must already be enabled for your account
- No subscription on the account may already have more than one item

After the flip, legacy `PATCH /api/v1/subscriptions/:id` with `plan_id`, `delivery_price_item_id`, or non-metered `plan_quantity` returns **409**. The error body directs you to nested `/items`. Use nested PATCH for billing-field changes on one-item and multi-item subscriptions alike — parent PATCH with `plan_id` or `delivery_price_item_id` still returns **409**, but nested PATCH returns **200** and processes `current_version`.

**Pilot billing:** subscriptions with 2+ items are not fully invoiced until item-level billing is enabled. Do not go live on multi-item billing before Boxful confirms readiness.

## Step 6: Bootstrap checkout lines on customer create

For **checkout** subscriptions (`status: unselected`) you cannot use `POST /api/v1/subscriptions/:id/items` — that route only resolves **activated** subscriptions. When the account is in multi-plan **and** multi-item write mode, bootstrap plan lines in the same call as customer create:

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/customers \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "customer": {
      "fname": "Ada",
      "lname": "Lovelace",
      "email": "ada@example.com",
      "birthdate": "1990-01-01",
      "phone_country_code": "54",
      "area_code": "11",
      "phone": "12345678",
      "doc_type": "DNI",
      "doc_number": "12345678",
      "external_reference": "order_98765",
      "subscription_items": [
        { "plan_id": 101, "plan_quantity": 0 },
        { "plan_id": 202, "client_external_reference": "woo-line-2" }
      ]
    }
  }'
```

Each object in `subscription_items` uses the same fields as [Create a subscription item](../reference/subscription-items.md#create-a-subscription-item). The request is **all-or-nothing**: if any line fails, no customer or subscription rows are persisted.

On success the response still includes `msgver` for hosted checkout and **`subscription: null`** until the subscription is activated (unchanged). Use `GET /api/v1/customers/{id}/subscription_cart` to read aggregated `checkout_data.prices` before payment.

**Do not** combine `subscription_items` with legacy plan fields on customer create (`subscription_cart_attributes.base_plan_id`, root `plan_id`, or `base_plan_id`). On cart PATCH use `items` and do not combine them with legacy `plan_id` or `base_plan_id` — send `coupon_id` only when you need a coupon on the pending cart. See [Create a customer](../reference/customers.md#create-a-customer) and [Update a subscription cart](../reference/subscription-carts.md#update-a-subscription-cart).

## Step 6b: Bootstrap checkout lines on cart PATCH

When you prefer a **two-step** checkout API flow (customer first, cart second), create the customer without `subscription_items`, then bootstrap plan lines on the pending cart:

```shell
# 1. Customer + msgver for hosted checkout
curl -s -X POST https://<subdomain>.boxful.io/api/v1/customers \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "customer": {
      "fname": "Ada",
      "lname": "Lovelace",
      "email": "ada@example.com",
      "birthdate": "1990-01-01",
      "phone_country_code": "54",
      "area_code": "11",
      "phone": "12345678",
      "doc_type": "DNI",
      "doc_number": "12345678"
    }
  }'

# 2. Multi-item pending cart (payment_type, coupon_id, etc. allowed in the same PATCH)
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/customers/{customer_id}/subscription_cart \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "items": [
      { "plan_id": 101, "plan_quantity": 0 },
      { "plan_id": 202, "client_external_reference": "woo-line-2" }
    ],
    "payment_type": "individual_payment"
  }'
```

Open hosted checkout with the `msgver` from step 1:

`https://<subdomain>.boxful.io/subscription_signups?msgver=<token>`

The customer lands on checkout with the N plan lines from step 2. Same pilot gates and error contract as Step 6.

## Step 7: Stop PATCHing top-level `plan_id`

After the flip, route plan-line changes through nested routes where supported:

| Action | Route |
|---|---|
| Change `plan_id` / delivery on one item (N>1 subscription) | `PATCH /api/v1/subscriptions/:id/items/:item_id` — processes `current_version` |
| Change `plan_id` / delivery on one item (N=1 subscription) | `PATCH /api/v1/subscriptions/:id/items/:item_id` — parent PATCH **409** |
| Update `client_external_reference` | `PATCH /api/v1/subscriptions/:id/items/:item_id` |
| Add a plan line | `POST /api/v1/subscriptions/:id/items` |
| Remove a plan line (not the last one) | `DELETE /api/v1/subscriptions/:id/items/:item_id` |
| Cancel subscription | `PATCH /api/v1/subscriptions/:id` with `status: cancelled` |
| Report metered consumption (exactly one metered item on the subscription) | `PATCH /api/v1/subscriptions/:id` with `plan_quantity` (works on N>1 when only one metered line) |
| Report metered consumption (two or more metered items) | `PATCH /api/v1/subscriptions/:id/items/:item_id` with `plan_quantity` |

To remove the last item, cancel the subscription or change the item's plan — DELETE on the sole item returns **422**.

## Step 8: Report metered consumption per item

When a subscription has **exactly one metered item** (including N>1 subscriptions with preset siblings), parent PATCH with `plan_quantity` still works — same as single-item behavior.

When **two or more metered items** exist on the subscription, parent PATCH with `plan_quantity` returns **409** — report consumption on the specific item:

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/subscriptions/14039/items/12 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "plan_quantity": 25 }'
```

The quantity is stored on the item row and propagated to the live invoiceable cart for that item. Preset siblings are unchanged. The parent `plan_quantity` column stays nil on multi-item subscriptions.

See [Update a subscription item](../reference/subscription-items.md#update-a-subscription-item) for status codes (**409** on non-metered items when N>1).

## Notes

- **Feature vs write mode:** Multi-plan subscriptions (enabled by Boxful) gates `items[]` reads and nested routes. Multi-item write mode gates whether legacy parent PATCH is allowed.
- **Disabling multi-plan subscriptions** does not roll back item rows or undo a multi-item write-mode flip.
- Full field and status-code reference: [Subscription Items](../reference/subscription-items.md#compatibility) and [Subscriptions](../reference/subscriptions.md#multi-item-write-mode).
