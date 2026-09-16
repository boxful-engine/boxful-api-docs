# Multi-Item Subscriptions

Migrate from legacy top-level `plan_id` writes to nested `/items` routes without breaking existing single-plan integrations.

- [Prerequisites](#prerequisites)
- [Step 1: Enable multi-plan subscriptions](#step-1-enable-multi-plan-subscriptions)
- [Step 2: Read `items[]` on subscriptions](#step-2-read-items-on-subscriptions)
- [Step 3: Adopt nested PATCH for plan changes](#step-3-adopt-nested-patch-for-plan-changes)
- [Step 4: Adopt nested POST to append items](#step-4-adopt-nested-post-to-append-items)
- [Step 5: Flip to multi-item write mode](#step-5-flip-to-multi-item-write-mode)
- [Step 6: Stop PATCHing top-level `plan_id`](#step-6-stop-patching-top-level-plan_id)
- [Step 7: Report metered consumption per item](#step-7-report-metered-consumption-per-item)
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

After the flip, legacy `PATCH /api/v1/subscriptions/:id` with `plan_id`, `delivery_price_item_id`, or non-metered `plan_quantity` returns **409**. The error body directs you to nested `/items`. On a **one-item** subscription, use nested PATCH for billing-field changes — parent PATCH with `plan_id` or `delivery_price_item_id` still returns **409**, but nested PATCH returns **200** and processes `current_version`.

**Pilot billing:** subscriptions with 2+ items are not fully invoiced until item-level billing is enabled. Do not go live on multi-item billing before Boxful confirms readiness.

## Step 6: Stop PATCHing top-level `plan_id`

After the flip, route plan-line changes through nested routes where supported:

| Action | Route |
|---|---|
| Change `plan_id` / delivery on one item (N>1 subscription) | Not supported until item-level invoicing — **409** |
| Change `plan_id` / delivery on one item (N=1 subscription) | `PATCH /api/v1/subscriptions/:id/items/:item_id` — parent PATCH **409** |
| Update `client_external_reference` | `PATCH /api/v1/subscriptions/:id/items/:item_id` |
| Add a plan line | `POST /api/v1/subscriptions/:id/items` |
| Remove a plan line (not the last one) | `DELETE /api/v1/subscriptions/:id/items/:item_id` |
| Cancel subscription | `PATCH /api/v1/subscriptions/:id` with `status: cancelled` |
| Report metered consumption (exactly one metered item on the subscription) | `PATCH /api/v1/subscriptions/:id` with `plan_quantity` (works on N>1 when only one metered line) |
| Report metered consumption (two or more metered items) | `PATCH /api/v1/subscriptions/:id/items/:item_id` with `plan_quantity` |

To remove the last item, cancel the subscription or change the item's plan — DELETE on the sole item returns **422**.

## Step 7: Report metered consumption per item

When a subscription has **exactly one metered item** (including N>1 subscriptions with preset siblings), parent PATCH with `plan_quantity` still works — same as single-item behavior.

When **two or more metered items** exist on the subscription, report consumption on the specific item:

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
