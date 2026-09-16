# Subscription Carts

A SubscriptionCart is the "template" that will be used to create a Subscription for a Customer. A Customer can have only one SubscriptionCart at a time. The SubscriptionCart only exists when the Customer does not have a Subscription yet.

For more details refer to our [knowledge base](https://boxful.freshdesk.com/support/solutions/articles/60000887063-recursos-de-la-api-de-boxful).

- [Fields](#fields)
- [Endpoints](#endpoints)
  - [Get a subscription cart](#get-a-subscription-cart)
  - [Update a subscription cart](#update-a-subscription-cart)
  - [Authorize a subscription cart](#authorize-a-subscription-cart)

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | - | Read | - |
| `object` | string | - | Read | Resource type |
| `base_plan_id` | integer | - | Read | Associated BasePlan ID |
| `external_reference` | string | - | Read | - |
| `coupon_id` | integer | - | Write (only update) | - |
| `delivery_price_item_id` | integer | - | Write (only update) | - |
| `payment_type` | string | - | Write (only update) | `credit_card`, `cbu` or `individual_payment`. Limited to the types the account has accepted. |
| `plan_id` | integer | - | Write (only update) | - |
| `checkout_data.prices` | object | - | Read | - |
| `checkout_data.properties` | object | - | Read | - |

## Endpoints

### Get a subscription cart

- `GET /api/v1/customers/{customer_id}/subscription_cart`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/customers/123456/subscription_cart \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "object": "SubscriptionCart",
  "coupon_id": null,
  "delivery_price_item_id": null,
  "plan_id": 6,
  "checkout_data": {
    "prices": {
      "checkout_total_cost": 100.0,
      "fixed_price_gross": 100.0,
      "fixed_price_discount": 0.0,
      "fixed_price_net": 100.0,
      "per_unit_amount_gross": 0.0,
      "per_unit_amount_discount": 0.0,
      "per_unit_amount_net": 0.0,
      "recurrent_total_cost": 100.0,
      "setup_fee": 0.0,
      "shipping_price_gross": 0.0,
      "total_cost": 100.0,
      "total_discounts": 0.0,
      "total_recurrent_cost_without_discounts": 100.0
    },
    "properties": {
      "currency": "ARS",
      "deliverable": false,
      "free_trial": false,
      "metered": false,
      "per_unit": false,
      "setup_fee": false,
      "setup_fee_billing_pending?": false,
      "setup_fee_charged_at_free_trial_start": false
    }
  }
}
```

### Update a subscription cart

- `PATCH /api/v1/customers/{customer_id}/subscription_cart`

| Param | Type | Required | Notes |
|---|---|---|---|
| `payment_type` | string | no | How the subscription will be collected: `credit_card`, `cbu` or `individual_payment`. Only the types the account has accepted are allowed — same set the customer checkout offers. Sending a type the account has not accepted returns `422`. |

###### Example request

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/customers/123456/subscription_cart \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "coupon_id": 1, "payment_type": "individual_payment" }'
```

###### Example response

Returns the updated SubscriptionCart resource with recalculated `checkout_data.prices`.

#### Multi-item write mode

When the account is in **multi-item** API mode, legacy cart writes to `plan_id`, `delivery_price_item_id`, or non-metered `plan_quantity` return **409** with `"use /api/v1/subscriptions/:id/items"`. Default **single-item** mode keeps these fields working. **Metered** `plan_quantity` (consumption) is still accepted. See [Update a subscription](subscriptions.md#update-a-subscription) for the full contract.

###### Error response

```json
{
  "errors": ["payment_type is not accepted by this account"]
}
```

When multi-item API mode is enabled, legacy item-field writes return **409**:

```json
{
  "errors": ["use /api/v1/subscriptions/:id/items"]
}
```

### Authorize a subscription cart

- `POST /api/v1/customers/{customer_id}/subscription_cart/authorize`

> [!WARNING]
> This endpoint was not part of the original API documentation. It may change without notice.

> **Restricted**: This endpoint is not available to all accounts. Contact your Boxful representative for access.

Authorizes the customer's pending cart and activates the subscription. The
action takes no parameters — set the cart up first (including `payment_type`)
with `PATCH /subscription_cart`, then authorize it.

How the cart is authorized depends on its `payment_type`:

| `payment_type` | Behavior |
|---|---|
| `credit_card` | The card set as the cart's default payment method is charged through the account's gateway. |
| `individual_payment` | The subscription is authorized without charging: a **scheduled** first invoice is created awaiting offline collection, and no payment record is created. |

Authorize only ever activates the customer's pending subscription. Once it has
been authorized there is nothing left to authorize, so a repeated call returns
the `invalid_state` body below rather than activating a second subscription.

###### Example request

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/customers/123456/subscription_cart/authorize \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "status": 200,
  "subscription": {
    "object": "Subscription",
    "status": "authorized",
    "payment_type": "individual_payment"
  }
}
```

###### Error response

Returned with HTTP `200` — inspect the in-body `status`.

```json
{
  "status": 422,
  "error": "invalid_state",
  "message": "SubscriptionCart is not processable"
}
```
