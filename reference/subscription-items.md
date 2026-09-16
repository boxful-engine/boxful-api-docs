# Subscription Items

> **Restricted**: Nested item routes require the `multi_plan_subscriptions` feature. Contact your Boxful representative for access.

A subscription item is one plan line on a subscription (`plan_id`, quantity, delivery, and optional `client_external_reference`). Read `items[]` on [Subscriptions](subscriptions.md#get-a-subscription) when the feature is enabled.

On a **single-item** subscription, `plan_id`, `plan_quantity`, and `delivery_price_item_id` use the same write path as [Update a subscription](subscriptions.md#update-a-subscription): a new cart version is processed so the invoiceable cart (`current_version`) moves with the item. `client_external_reference` is stored on the item row and does not process a cart.

When Boxful enables **multi-item write mode** for your account, legacy parent/cart writes (`plan_id`, non-metered `plan_quantity`, `delivery_price_item_id`) on the subscription and cart endpoints return **409**. Use this nested route for plan-line changes on one-item subscriptions — including after the flip, when parent/cart PATCH with those billing fields still returns **409** but nested PATCH returns **200** and processes `current_version`.

On a **multi-item subscription** (more than one item), `plan_id` and `delivery_price_item_id` return **409** until item-level invoicing exists. **`plan_quantity`** on a **metered** item is allowed — consumption is stored on that item row and propagated to the live invoiceable cart; preset siblings are unchanged. **`plan_quantity`** on a non-metered item returns **409**. `client_external_reference` can still be updated on any item.

- [Compatibility](#compatibility)
- [Fields](#fields)
- [Error responses](#error-responses)
- [Endpoints](#endpoints)
  - [Create a subscription item](#create-a-subscription-item)
  - [Update a subscription item](#update-a-subscription-item)
  - [Delete a subscription item](#delete-a-subscription-item)

## Compatibility

Two account settings work together:

| Setting | Role |
|---|---|
| Multi-plan subscriptions (enabled by Boxful) | Required for `items[]` on reads and all nested `/items` routes. |
| Multi-item write mode (enabled by Boxful, one-way) | API write contract. Default is **single-item**; Boxful switches your account to **multi-item** when you are ready for 2+ items. |

| Operation | `single_item` write mode | `multi_item` write mode |
|---|---|---|
| `GET` subscription `items[]` | Present when multi-plan subscriptions are enabled | Same |
| `PATCH /subscriptions/:id` with `plan_id` / `delivery_price_item_id` / non-metered `plan_quantity` | Allowed | **409** — use nested `/items` |
| `POST /subscriptions/:id/items` (first item on itemless sub) | **201** — parent legacy columns hydrated from the sole item; `current_version` moves | Same |
| `POST /subscriptions/:id/items` (append 2nd item) | **409** — ask Boxful to enable multi-item write mode first | **201** — append only; second item not invoiced until item-level invoicing |
| `PATCH /subscriptions/:id/items/:id` `plan_id` / `delivery_price_item_id` on N>1 sub | **409** | **409** until item-level invoicing |
| `PATCH /subscriptions/:id/items/:id` metered `plan_quantity` on N>1 sub | **200** on metered item only | Same — **409** on non-metered items |
| `PATCH /subscriptions/:id` metered `plan_quantity` on N>1 sub | **200** when exactly one metered item; **422** when two or more | Same — use nested `/items/:id` to target a specific metered item |
| `PATCH /subscriptions/:id/items/:id` `plan_id` / `delivery_price_item_id` on N=1 sub | **200** — processes `current_version` | **200** — same cart-version path as single-item write mode |
| `PATCH /subscriptions/:id/items/:id` `client_external_reference` on N=1 sub | **200** | **200** |
| `PATCH` / `POST` `client_external_reference` on N>1 sub | **200** | **200** |
| `DELETE /subscriptions/:id/items/:id` (last item) | **422** — cancel or change plan instead | Same |
| Duplicate `plan_id` on one subscription | Not reachable (max 1 item) | Allowed |
| `client_external_reference` uniqueness | Per account when present | Same |

Do not go live on multi-item billing with 2+ items until Boxful confirms item-level invoicing is enabled for your account.

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | `true` | Read | Stable item id |
| `object` | string | - | Read | Always `SubscriptionItem` |
| `plan_id` | integer | `true` on create | Read/Write | Must belong to the same account. Required on create. On update, single-item only; **409** when the subscription has more than one item |
| `plan_quantity` | float | - | Read/Write | Required to be ≥ 0 when the plan is per-unit. **Metered** consumption: on single-item subscriptions uses the same path as parent PATCH; on multi-item subscriptions allowed only when this item's plan is metered (writes the item row and live cart qty, not the parent column). **409** for non-metered items when the subscription has more than one item |
| `delivery_price_item_id` | integer | - | Read/Write | Must belong to the same account. Same presence rules as parent PATCH (required when the plan is deliverable). Single-item only; **409** when the subscription has more than one item |
| `client_external_reference` | string | - | Read/Write | Unique per account when present. Blank is stored as `null`. Allowed on single-item and multi-item subscriptions |

## Error responses

Failed writes return a JSON body with an `errors` array of human-readable strings:

```json
{
  "errors": ["Cannot add items to a cancelled subscription"]
}
```

When `multi_plan_subscriptions` is not enabled, write requests return **403** with:

```json
{
  "errors": ["Multi-plan subscriptions are not enabled for this account"]
}
```

The gate runs after the subscription and item ids in the URL are loaded, so an unknown subscription or item on the path may still return **404** before **403**. Body fields such as `plan_id` and `delivery_price_item_id` are not resolved when the feature is disabled — invalid ids in the JSON body also return **403**, not **404**.

When the feature is enabled, **404** applies to subscription, item, plan, or delivery ids that do not belong to the account. **422** and **409** responses use the same `errors` array shape.

## Endpoints

### Create a subscription item

- `POST /api/v1/subscriptions/{subscription_id}/items`

**Alternative identification**: the parent `{subscription_id}` accepts `?identifier=external_reference`, same as [Get a subscription](subscriptions.md#get-a-subscription).

Duplicate `plan_id` is allowed.

Returns **201** and the new item.

Returns **403** when `multi_plan_subscriptions` is not enabled for the account.

Returns **404** when the feature is enabled and the subscription is not in the account, or when `plan_id` does not belong to the account.

Returns **409** when the account is still in **single-item** write mode and the subscription already has an item. Ask Boxful to enable multi-item write mode before appending a second item. A concurrent first-item create that loses the row lock also returns **409**.

The first item on an itemless subscription hydrates parent `plan_id`, `delivery_price_item_id`, and `plan_quantity` from the sole item and moves the invoiceable cart (`current_version`) with that item. In **single-item** write mode that uses the same path as [Update a subscription](subscriptions.md#update-a-subscription). In **multi-item** write mode the nested create path reconciles cart lines in place (not a legacy parent PATCH).

Appending a second item (**multi-item** write mode only) syncs item lines onto pending and `current_version` carts; parent legacy columns on the subscription and carts are cleared for the multi-item shape. It does not process a new cart version for plan changes. Until item-level invoicing exists, the second item may not appear on invoice lines. Scheduled invoicing may fail rather than undercharge.

Returns **422** when `plan_id` is missing or blank, or when the item cannot be saved (cancelled subscription, sibling-incompatible plan, validation errors).

###### Example request

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/subscriptions/14039/items \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "plan_id": 88,
    "plan_quantity": 2,
    "delivery_price_item_id": 4,
    "client_external_reference": "kid-a"
  }'
```

###### Example response

```json
{
  "id": 13,
  "object": "SubscriptionItem",
  "plan_id": 88,
  "plan_quantity": 2.0,
  "delivery_price_item_id": 4,
  "client_external_reference": "kid-a"
}
```

### Update a subscription item

- `PATCH /api/v1/subscriptions/{subscription_id}/items/{item_id}`

**Alternative identification**: the parent `{subscription_id}` accepts `?identifier=external_reference`, same as [Get a subscription](subscriptions.md#get-a-subscription).

Returns **404** when the subscription is not in the account, or when `item_id` is not an item on that subscription.

Returns **403** when `multi_plan_subscriptions` is not enabled for the account.

Returns **409** when `plan_id` or `delivery_price_item_id` is sent on a subscription that already has more than one item.

Returns **409** when `plan_quantity` is sent on a **non-metered** item and the subscription has more than one item.

On a **one-item** subscription after multi-item write mode is enabled, `plan_id` and `delivery_price_item_id` process a new invoiceable cart (`current_version`), same as in single-item write mode. Parent PATCH with those fields still returns **409** — use this nested route instead.

On multi-item subscriptions, metered `plan_quantity` writes the item row and propagates to the pending cart and `current_version` cart lines for that item. The parent `plan_quantity` column stays nil. This does not process a new cart version for plan or delivery changes.

Returns **422** when the item cannot be saved (cancelled subscription, validation errors).

###### Example request (plan change)

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/subscriptions/14039/items/12 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "plan_id": 88,
    "plan_quantity": 2,
    "delivery_price_item_id": 4,
    "client_external_reference": "kid-a"
  }'
```

###### Example request (metered consumption on a multi-item subscription)

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/subscriptions/14039/items/12 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "plan_quantity": 25 }'
```

###### Example response

```json
{
  "id": 12,
  "object": "SubscriptionItem",
  "plan_id": 88,
  "plan_quantity": 2.0,
  "delivery_price_item_id": 4,
  "client_external_reference": "kid-a"
}
```

### Delete a subscription item

- `DELETE /api/v1/subscriptions/{subscription_id}/items/{item_id}`

**Alternative identification**: the parent `{subscription_id}` accepts `?identifier=external_reference`, same as [Get a subscription](subscriptions.md#get-a-subscription).

Returns **204** with an empty body.

Returns **404** when the subscription is not in the account, or when `item_id` is not an item on that subscription.

Returns **403** when `multi_plan_subscriptions` is not enabled for the account.

Returns **422** when the item is the last one on the subscription (`single_item` and `multi_item`). This endpoint does not leave an itemless subscription. Cancel the subscription, or [update the item](#update-a-subscription-item) to another plan.

Returns **422** when the subscription is cancelled.

When the delete leaves **one** item, a new invoiceable cart (`current_version`) is processed from the remaining item via `take_subscription_based_cart`. Parent legacy columns (`plan_id`, `delivery_price_item_id`, `plan_quantity`) are repopulated from the surviving item (N=1 shape) as part of that cart sync — not via a legacy `Api::SubscriptionUpdate` write, which remains blocked while the account is in multi-item mode.

When **two or more** items remain, this delete does not process a new `current_version`. Invoices bill from that cart, so a removed item is not re-invoiced until item-level invoicing exists. Until then, scheduled invoicing may fail rather than undercharge.

Invoice lines and processed cart links keep their rows; `subscription_item_id` is nullified. This is not an archive.

###### Example request

```shell
curl -s -X DELETE https://<subdomain>.boxful.io/api/v1/subscriptions/14039/items/12 \
  -H "Authorization: Bearer $TOKEN"
```
