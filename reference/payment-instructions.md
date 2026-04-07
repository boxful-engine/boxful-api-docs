# Payment Instructions

A PaymentInstruction groups the line items that will later be invoiced for a given customer. Once it is invoiced the related Invoice is embedded in the response.

- [Embedded Resources](#embedded-resources)
- [Fields](#fields)
- [Endpoints](#endpoints)
  - [List all payment instructions from a customer](#list-all-payment-instructions-from-a-customer)
  - [Get a payment instruction](#get-a-payment-instruction)
  - [Create a payment instruction](#create-a-payment-instruction)

## Embedded Resources

### PaymentInstructionItem

| Field | Type | Notes |
|---|---|---|
| `id` | integer | - |
| `object` | string | Always `PaymentInstructionItem` |
| `amount` | float | Line item amount |
| `currency` | string | ISO currency code |
| `description` | string | Required line description |

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | `true` | Read | - |
| `object` | string | - | Read | Always `PaymentInstruction` |
| `amount` | float | `true` | Read | Sum of all non-deleted items amounts |
| `collect_on` | date | - | Read | Optional target collection date |
| `created_at` | datetime | - | Read | - |
| `currency` | string | `true` | Read | ISO currency code taken from the account defaults |
| `customer_id` | integer | `true` | Read | Internal customer identifier |
| `customer_doc_number` | string | - | Read | Depends on customer profile |
| `customer_doc_type` | string | - | Read | Depends on customer profile |
| `customer_email` | string | - | Read | - |
| `customer_external_reference` | string | - | Read | - |
| `external_reference` | string | - | Write (only create) | Optional identifier set by the client |
| `invoice` | json | - | Read | Serialized Invoice when invoiced |
| `items` | array | `true` | Read | Array of PaymentInstructionItem resources |
| `status` | string | `true` | Read | Valid values: `pending`, `invoiced`, `cancelled`, `discarded` |
| `updated_at` | datetime | - | Read | - |

## Endpoints

### List all payment instructions from a customer

- `GET /api/v1/customers/{customer_id}/payment_instructions`

> [!WARNING]
> This endpoint was not part of the original API documentation. It may change without notice.

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/customers/14038/payment_instructions \
  -H "Authorization: Bearer $TOKEN"
```

### Get a payment instruction

- `GET /api/v1/payment_instructions/{payment_instruction_id}`

> [!WARNING]
> This endpoint was not part of the original API documentation. It may change without notice.

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/payment_instructions/9822 \
  -H "Authorization: Bearer $TOKEN"
```

### Create a payment instruction

- `POST /api/v1/customers/{customer_id}/payment_instructions`

Creates a payment instruction with at least one line item. The total amount is recalculated on the server from the submitted items. When linking a subscription, the `subscription_id` must belong to the same customer and the subscription must be in `authorized` or `in_trial` state.

###### Example request

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/customers/14038/payment_instructions \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "payment_instruction": {
      "external_reference": "PI-00045",
      "subscription_id": 169,
      "items_attributes": [
        {
          "amount": 1500.0,
          "description": "Servicio de mantenimiento"
        }
      ]
    }
  }'
```

###### Example response

```json
{
  "id": 9822,
  "object": "PaymentInstruction",
  "amount": 1500.0,
  "collect_on": null,
  "created_at": "2024-11-15T15:45:03.000-03:00",
  "currency": "ARS",
  "customer_id": 14038,
  "customer_doc_number": "12345678",
  "customer_doc_type": "DNI",
  "customer_email": "juan.perez@mail.com",
  "customer_external_reference": "a1b2c3d4",
  "external_reference": "PI-00045",
  "invoice": null,
  "items": [
    {
      "id": 14522,
      "object": "PaymentInstructionItem",
      "amount": 1500.0,
      "currency": "ARS",
      "description": "Servicio de mantenimiento"
    }
  ],
  "status": "pending",
  "updated_at": "2024-11-15T15:45:03.000-03:00"
}
```

###### Error responses

```json
// 422 - Validation errors
{ "errors": { "items": ["can't be blank"] } }

// 422 - Subscription not found
{ "error": "subscription_not_found", "message": "Subscription not found or does not belong to this customer" }

// 422 - Subscription in invalid state
{ "error": "subscription_invalid_state", "message": "Subscription must be in authorized or in_trial state" }
```
