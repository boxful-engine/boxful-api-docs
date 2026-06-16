# Iframe Subscription Checkout

Embed Boxful's payment forms directly into your application using an iframe, providing a seamless checkout experience for your customers.

- [Prerequisites](#prerequisites)
- [Step 1: Create a customer with a subscription cart](#step-1-create-a-customer-with-a-subscription-cart-optional)
- [Step 2: Generate an embed token](#step-2-generate-an-embed-token)
  - [Subscription checkout](#subscription-checkout)
  - [Payment method transition](#payment-method-transition)
- [Step 3: Embed the iframe](#step-3-embed-the-iframe)
- [Message types](#message-types)
- [Notes](#notes)

## Prerequisites

- A Boxful account with API access
- For **subscription checkout**: a [Customer](../reference/customers.md) with a [Subscription Cart](../reference/subscription-carts.md) ready to checkout (pending cart).
- For **payment method transition**: the same API access; the customer must own the target [Subscription](../reference/subscriptions.md) (embed support is gateway-specific, e.g. dLocal for card replacement).

## Step 1: Create a customer with a subscription cart (optional)

If the customer doesn't exist yet, create one with a pre-configured subscription cart in a single API call:

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/customers \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "customer": {
      "email": "customer@example.com",
      "fname": "John",
      "lname": "Doe",
      "doc_type": "RUT",
      "doc_number": "12345678",
      "subscription_cart_attributes": {
        "base_plan_id": 123,
        "coupon_id": 45
      }
    }
  }'
```

- `base_plan_id` is required in `subscription_cart_attributes`.
- `coupon_id` is optional.
- If subscription cart creation fails, the customer is still created successfully.
- You can also omit `subscription_cart_attributes` entirely and configure the cart later via the [Update a Subscription Cart](../reference/subscription-carts.md#update-a-subscription-cart) endpoint.

## Step 2: Generate an embed token

Use `POST /api/v1/embed/tokens` with a Bearer token from your Boxful account. The response includes a signed JWT (`token`), an `expires_at` timestamp, and an `iframe_type` echoing the request. **Tokens expire in 5 minutes.**

Shared fields:

| Field | Type | Required | Notes |
|---|---|---|---|
| `iframe_type` | string | `true` | `subscription_checkout` or `membership_payment_method_transition` (see below). |
| `customer_id` | integer | `true` | Must exist on the account. For checkout, the customer must have a **pending** subscription cart. For transition, they must **own** `subscription_id`. |
| `origins` | string | Required for `subscription_checkout`; optional for `membership_payment_method_transition` | Comma-separated parent origin(s) for `Content-Security-Policy: frame-ancestors`. Supports wildcards (e.g., `http://127.0.0.1:*`). Omitting `origins` on transition may restrict embedding depending on server defaults—set it when the iframe runs on a known parent origin. |

More detail, routes, and error behavior: [Iframe subscription checkout (technical)](../../../embed/iframe-subscription-checkout.md) (including [payment method transition](../../../embed/iframe-subscription-checkout.md#payment-method-transition-existing-subscription)).

### Subscription checkout

For a **new** subscription checkout, the JWT includes `subscription_cart_id` (the customer’s pending cart).

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/embed/tokens \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "iframe_type": "subscription_checkout",
    "customer_id": 12345,
    "origins": "https://your-site.com"
  }'
```

### Payment method transition

For changing or replacing the **payment method** on an **existing** subscription inside the same iframe flow (e.g. credit card replacement; gateway-specific—**dLocal** is supported for card flows). The JWT includes `subscription_id` and **not** `subscription_cart_id`.

| Field | Type | Required | Notes |
|---|---|---|---|
| `subscription_id` | integer | `true` for this `iframe_type` | Subscription must exist on the account and belong to `customer_id`. Otherwise the API returns **404**. |

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/embed/tokens \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "iframe_type": "membership_payment_method_transition",
    "customer_id": 12345,
    "subscription_id": 67890,
    "origins": "https://your-site.com"
  }'
```

After the iframe completes `POST /embed/sessions`, the app redirects to `GET /embed/membership/payment_method_transitions/:id/edit`, where **`:id` must equal** the token’s `subscription_id`. A `subscription_checkout` token **cannot** be used on those URLs—the embed returns an error. Implementers normally only load `https://<subdomain>.boxful.io/embed/sessions/new` and pass the transition token the same way as for checkout; no manual path construction is required.

###### Response

```json
{
  "token": "eyJpZnJhbWVfd...",
  "expires_at": "2025-10-01 20:45:23 -0300",
  "iframe_type": "subscription_checkout"
}
```

For a payment method transition request, `iframe_type` in the response is `membership_payment_method_transition`.

## Step 3: Embed the iframe

Add an iframe pointing to your Boxful subdomain’s embed session entrypoint (`GET /embed/sessions/new`). The parent listens for `postMessage`, supplies the token when asked, then handles **checkout** (`subscription:success`) or **payment method transition** (`payment_method_transition:success`) outcomes the same way for both token types.

```html
<iframe id="boxful-checkout" src="https://<your-subdomain>.boxful.io/embed/sessions/new"></iframe>
<script>
  const token = "your-token-here";

  window.addEventListener("message", (e) => {
    switch (e.data.type) {
      case "embed:token_request":
        e.source.postMessage({ type: "embed:token", token: token }, "*");
        break;
      case "embed:token_received":
        break;
      case "subscription:success":
        console.log("Subscription created", e.data);
        break;
      case "payment_method_transition:success":
        console.log("Payment method updated", e.data);
        break;
      case "subscription:error":
        console.log("Subscription error", e.data);
        break;
      case "embed:subscription_cart:promotion_applied":
        console.log("Promotion applied", e.data);
        break;
      case "embed:subscription_cart:promotion_removed":
        console.log("Promotion removed", e.data);
        break;
      case "embed:error":
        console.log("Embed error", e.data);
        break;
    }
  });
</script>
```

## Message types

### Messages from iframe to parent

| Type | When | Payload |
|---|---|---|
| `embed:token_request` | Iframe loads and has no token yet | Parent must reply with `embed:token` and the token |
| `embed:token_received` | After parent sends `embed:token` | Confirmation only |
| `embed:error` | Token missing/expired, invalid account, consumed subscription cart, or invalid transition context | `message` (string) |
| `subscription:success` | After successful subscription creation | `data`: `id`, `base_plan_id`, `customer_id` |
| `payment_method_transition:success` | After a successful payment method transition (e.g. new card applied) | `data`: `subscription_id` (integer) |
| `subscription:error` | Business rule failure (e.g., already has active subscription) | `message` (string) |
| `embed:subscription_cart:promotion_applied` | BIN lookup resulted in a promotion being applied | `data.promotion_applied` (true), `data.checkout_data` |
| `embed:subscription_cart:promotion_removed` | BIN changed and promotion was removed | `data.promotion_applied` (false), `data.checkout_data` |

### Messages from parent to iframe

| Type | When | Payload |
|---|---|---|
| `embed:token` | In response to `embed:token_request` | `token` (string) |

### Success message example (subscription checkout)

```json
{
  "type": "subscription:success",
  "message": "Subscription created successfully",
  "data": {
    "id": 789,
    "base_plan_id": 25,
    "customer_id": 123
  }
}
```

### Success message example (payment method transition)

```json
{
  "type": "payment_method_transition:success",
  "message": "Success message from Boxful (localized)",
  "data": {
    "subscription_id": 67890
  }
}
```

### Error messages

| Error message | Description |
|---|---|
| Token missing | The token is missing |
| Expired token | The token has expired |
| Invalid account | The token's account does not match the request's account |
| Invalid or consumed subscription cart | The subscription cart is invalid or already processed |
| Token verification failed | Fallback error |

### Promotion messages

Sent when the user enters or changes the card number and a BIN-based promotion is applied or removed.

```json
{
  "type": "embed:subscription_cart:promotion_applied",
  "message": "",
  "data": {
    "promotion_applied": true,
    "checkout_data": { "..." }
  }
}
```

`checkout_data` contains the serialized subscription cart checkout data (prices). It may be `null` in some responses.

## Notes

- **Payment failures** (e.g., card declined) are handled within the iframe — the form re-renders with the error and does not require parent intervention.
- **CSP**: Embed responses set `Content-Security-Policy: frame-ancestors` using the token's `origins`, so the iframe can only be embedded in allowed parent origins.
- **Payment method transition**: Listen for `payment_method_transition:success` in addition to `subscription:success` and `embed:error`. Gateway support for card replacement in embed is not universal; confirm with your Boxful configuration (dLocal is supported for this flow).
