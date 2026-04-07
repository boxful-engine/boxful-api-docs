# Webhooks

- [Webhook authenticity verification](#webhook-authenticity-verification)
- [Events list](#events-list)
- [Webhook structure](#webhook-structure)

You can configure a webhook endpoint via the dashboard.

Your endpoint must return a 2xx HTTP status code to let us know that you received the webhook notification successfully. Return a 2xx status code quickly — your endpoint should return before any complex logic that could cause a timeout.

If we don't receive a 2xx response code, we retry calling your webhook with an increasing delay for the next couple of days. After that we mark the event as failed and stop trying to send it. After multiple days without receiving any 2xx responses, your endpoint may be disabled.

## Webhook authenticity verification

You can opt-in to receive a signature with each webhook request to verify the authenticity and integrity of the information sent to your endpoint.

If you want to receive the signature, please contact your Boxful's account representative.

The signature is computed using the HMAC algorithm with `SHA256` and the `signature_key` of your account. The signature is sent in the `Boxful-Signature` header. The timestamp of the request is sent in the `Request-Timestamp` header.

To verify the signature:

1. **Extract the HMAC** from the `Boxful-Signature` header.
2. **Extract the timestamp** from the `Request-Timestamp` header.
3. **Get the payload** of the request body as a string.
4. **Generate your own HMAC** using the same `signature_key` and `SHA256`, from the string `timestamp.payload` (e.g., `"1609459200.{\"event\":\"event.type\",\"data\":{\"id\":\"1234567890\",\"status\":\"pending\"}}"`).
5. **Compare the HMACs**. If they match, the request is authentic and has not been tampered with.

## Events list

Boxful API currently supports and sends the following events:

| Event | Description |
|---|---|
| `bank_account.created` | A bank account was created |
| `bank_account.update` | A bank account was updated |
| `base_plan.created` | A base plan was created |
| `base_plan.update` | A base plan was updated |
| `credit_card.created` | A credit card was created |
| `credit_card.update` | A credit card was updated |
| `customer.created` | A customer was created |
| `customer.trace` | A customer trace event occurred |
| `customer.update` | A customer was updated |
| `invoice.created` | An invoice was created |
| `invoice.dunning` | An invoice entered dunning |
| `invoice.paid` | An invoice was paid |
| `invoice.unpaid` | An invoice was marked unpaid |
| `invoice.update` | An invoice was updated |
| `order.created` | An order was created |
| `order.paid` | An order was paid |
| `order.unpaid` | An order was marked unpaid |
| `order.update` | An order was updated |
| `payment.charged_back` | A payment was charged back |
| `payment.created` | A payment was created |
| `payment.failed` | A payment failed |
| `payment.hard_bounced` | A payment hard bounced |
| `payment.succeed` | A payment succeeded |
| `payment.update` | A payment was updated |
| `plan.created` | A plan was created |
| `plan.update` | A plan was updated |
| `refund.approved` | A refund was approved |
| `refund.created` | A refund was created |
| `refund.failed` | A refund failed |
| `subscription.authorized` | A subscription was authorized |
| `subscription.cancelled` | A subscription was cancelled |
| `subscription.completed` | A subscription was completed |
| `subscription.created` | A subscription was created |
| `subscription.in_trial` | A subscription entered trial |
| `subscription.paused` | A subscription was paused |
| `subscription.reactivated` | A subscription was reactivated |
| `subscription.update` | A subscription was updated |

## Webhook structure

| Field | Notes |
|---|---|
| `api_version` | - |
| `event_type` | One of the events listed above |
| `occurred_at` | Timestamp of when the event occurred |
| `content` | Key-value pair where the key is the object name and the value is the full resource. E.g., a `customer` key holds a Customer resource. Refer to each resource documentation for the complete payload. |

###### Example payload

```json
{
  "api_version": "v1",
  "event_type": "subscription.update",
  "occurred_at": "Wed, 04 May 2022 10:11:06 -03 -03:00",
  "content": {
    "subscription": {
      "id": 5,
      "object": "Subscription",
      "..."
    }
  }
}
```
