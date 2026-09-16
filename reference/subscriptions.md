# Subscriptions

Subscriptions are created during the checkout process. Therefore, editable fields can only be set at Update a Subscription.

- [Fields](#fields)
- [Endpoints](#endpoints)
  - [List all subscriptions](#list-all-subscriptions)
  - [Get a subscription](#get-a-subscription)
  - [Create a subscription](#create-a-subscription)
  - [Update a subscription](#update-a-subscription)

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | `true` | Read | - |
| `object` | string | - | Read | Resource type |
| `amount` | float | `true` | Read | Calculated automatically |
| `authorized_at` | datetime | - | Read | - |
| `cancel_on` | date | - | Read/Write (only update) | Scheduled cancellation date. Must be a future date. |
| `cancellation_reason` | string | - | Write (only update) | Valid values: `debt`, `dissatisfaction`, `other`, `price`, `regret` |
| `cancelled_at` | datetime | - | Read | - |
| `complete_date` | date | - | Read | The date a finite subscription was completed |
| `coupon_id` | integer | - | Write (only update) | - |
| `created_at` | datetime | - | Read | - |
| `currency` | string | - | Read | - |
| `debit_date_attributes` | json | - | Write (only update) | Nested attributes for debit date configuration. Contains `date_type` (string) and `value` (string) fields |
| `customer_id` | integer | `true` | Read | - |
| `delivery_price_item_id` | integer | `true` | Write (only update) | - |
| `external_service` | string | - | Read | Valid options: `MercadoPago`, `PayULatam` |
| `last_invoice` | json | - | Read | Last Invoice resource |
| `next_invoice` | json | - | Read | Next Invoice resource |
| `metadata` | json | - | Write (only update) | - |
| `payment_type` | string | `true` | Write (only update) | Valid values: `cbu`, `credit_card`, `individual_payment` |
| `plan_id` | integer | `true` | Write (only update) | - |
| `plan_quantity` | integer | - | Write (only update) | - |
| `items` | array | - | Read | Present when `:multi_plan_subscriptions` is enabled for the account. Additive — top-level `plan_id` / `plan_quantity` / `delivery_price_item_id` stay. Omitted from nested customer embeds and webhook payloads. Each element: `id`, `object` (`SubscriptionItem`), `plan_id`, `plan_quantity`, `delivery_price_item_id`, `client_external_reference`. |
| `status` | string | `true` | Write (only update) | Valid values: `authorized`, `in_trial`, `paused`, `cancelled`, `completed` |
| `trial_end` | date | - | Read | Date of free-trial end |
| `trial_start` | date | - | Read | Date of free-trial start |
| `updated_at` | datetime | - | Read | - |
| `customer_doc_number` | string | - | Read | Customer's document number |
| `customer_doc_type` | string | - | Read | Customer's document type |
| `customer_email` | string | - | Read | Customer's email |
| `customer_external_reference` | string | - | Read | Customer's external reference |
| `debit_date` | json | - | Read | Associated DebitDate resource (if configured) |
| `external_reference` | string | - | Read | Can be used as an identifier with `?identifier=external_reference` on Get and Update endpoints |

## Endpoints

### List all subscriptions

- `GET /api/v1/subscriptions`

> [!WARNING]
> This endpoint was not part of the original API documentation. It may change without notice.

> **Restricted**: This endpoint is not available to all accounts. Contact your Boxful representative for access.

#### Filtering options

| Field | Valid operators | Valid filter values |
|---|---|---|
| `status` | `is`, `is_not` | Subscription statuses |
| `customer_id` | `is` | customer.id (integer) |
| `customer_external_reference` | `is` | customer.external_reference (string) |

Both `customer_id` and `customer_external_reference` filter subscriptions by their associated customer.

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/subscriptions \
  -H "Authorization: Bearer $TOKEN"
```

### Get a subscription

- `GET /api/v1/subscriptions/{subscription_id}`

**Alternative identification**: Instead of using the `subscription_id`, the `external_reference` can be used as a parameter. To do so it must be specified in the query string that the `external_reference` is being used as an identifier.

For a Subscription with `id` `14039` and `external_reference` `sub-ext-123` these options are equivalent:
- `GET /api/v1/subscriptions/14039`
- `GET /api/v1/subscriptions/sub-ext-123?identifier=external_reference`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/subscriptions/1 \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

When `:multi_plan_subscriptions` is enabled for the account, the payload includes additive `items[]`. Accounts without the flag omit the key; top-level `plan_id` is unchanged in both cases.

```json
{
  "id": 14039,
  "object": "Subscription",
  "amount": 100.0,
  "authorized_at": "2021-03-29T13:43:32.140-03:00",
  "cancel_on": null,
  "cancellation_reason": null,
  "cancelled_at": null,
  "complete_date": null,
  "coupon_id": null,
  "created_at": "2021-01-23T13:25:13.861-03:00",
  "currency": "ARS",
  "customer_id": 14038,
  "delivery_price_item_id": null,
  "external_service": "MercadoPago",
  "last_invoice": {
    "id": 2123,
    "object": "Invoice",
    "amount": 100.0,
    "base_amount": 100.0,
    "category": "flat_fee",
    "currency": "ARS",
    "discounts": [],
    "due_date": "2021-03-29",
    "invoice_items": [
      {
        "id": 326,
        "object": "InvoiceItem",
        "amount": 100.0,
        "base_amount": 100.0,
        "category": "flat_fee",
        "currency": "ARS",
        "description": "Description",
        "discounted_amount": 0.0,
        "item_type": "Plan",
        "item_id": 12
      }
    ],
    "status": "paid",
    "reason_type": "Subscription",
    "reason_id": 100
  },
  "next_invoice": {
    "id": 2124,
    "object": "Invoice",
    "amount": 100.0,
    "base_amount": 100.0,
    "category": "flat_fee",
    "currency": "ARS",
    "discounts": [],
    "due_date": "2021-04-29",
    "invoice_items": [
      {
        "id": 327,
        "object": "InvoiceItem",
        "amount": 100.0,
        "base_amount": 100.0,
        "category": "flat_fee",
        "currency": "ARS",
        "description": "Description",
        "discounted_amount": 0.0,
        "item_type": "Plan",
        "item_id": 12
      }
    ],
    "status": "scheduled",
    "reason_type": "Subscription",
    "reason_id": 100
  },
  "metadata": null,
  "payment_type": "credit_card",
  "plan_id": 1,
  "plan_quantity": 0,
  "items": [
    {
      "id": 1,
      "object": "SubscriptionItem",
      "plan_id": 1,
      "plan_quantity": 0.0,
      "delivery_price_item_id": null,
      "client_external_reference": null
    }
  ],
  "status": "authorized",
  "trial_end": null,
  "trial_start": null,
  "updated_at": "2021-03-29T13:43:32.158-03:00",
  "external_reference": "sub-ext-123"
}
```

### Create a subscription

- `POST /api/v1/subscriptions`

> [!WARNING]
> This endpoint was not part of the original API documentation. It may change without notice.

> **Restricted**: This endpoint is not available to all accounts. Contact your Boxful representative for access.

###### Example request

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/subscriptions \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "customer_id": 14038,
      "plan_id": 1,
      "payment_type": "credit_card"
    }
  }'
```

### Update a subscription

- `PATCH /api/v1/subscriptions/{subscription_id}`

**Alternative identification**: Same as Get a subscription — `external_reference` can be used with `?identifier=external_reference`.

###### Example request

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/subscriptions/1 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "status": "cancelled",
      "debit_date_attributes": {
        "date_type": "ordinal",
        "value": "-1"
      }
    }
  }'
```

###### Example response

Returns the updated Subscription resource with `cancelled_at` set and `status` changed to `cancelled`.

To add a plan line, see [Create a subscription item](subscription-items.md#create-a-subscription-item). To change a plan line by item id, see [Update a subscription item](subscription-items.md#update-a-subscription-item). On a one-item subscription while the account is still in **single-item** write mode, nested `plan_id` / `plan_quantity` / `delivery_price_item_id` use this same cart-version write path. `client_external_reference` on the item is a row update and does not process a cart.

To remove an item, see [Delete a subscription item](subscription-items.md#delete-a-subscription-item). Deleting the last item returns **422** and does not leave an itemless subscription.

#### Multi-item write mode

When Boxful enables **multi-item write mode** for your account (one-way), legacy parent/cart writes to `plan_id`, `delivery_price_item_id`, or non-metered `plan_quantity` return **409**:

```json
{
  "errors": ["use /api/v1/subscriptions/:id/items"]
}
```

- Default is **single-item** write mode: the fields above keep working on this endpoint.
- **Metered** `plan_quantity` (consumption reporting) is still accepted on this endpoint. On subscriptions with **more than one item**, parent PATCH applies to the sole metered item when there is exactly one; when two or more metered items exist, parent PATCH returns **422** — use [Update a subscription item](subscription-items.md#update-a-subscription-item) with the target `item_id`.
- Metadata, status, coupon, and other non-item fields continue to return **200**.
- After the flip, plan-line changes on subscriptions with **more than one item** use [Subscription Items](subscription-items.md) when item-level invoicing exists (**409** until then). On a **one-item** subscription, parent PATCH with `plan_id` or `delivery_price_item_id` returns **409**; nested PATCH on `/items/:item_id` returns **200** and processes `current_version`. Metered `plan_quantity` and `client_external_reference` follow the same rules as before the flip on supported routes — see [Compatibility](subscription-items.md#compatibility).
