# Refunds

A Refund represents a refund operation associated with a Payment. A Refund belongs to an `original_payment` (the payment being refunded) and may be associated with a CreditNote.

- [Embedded Resources](#embedded-resources)
- [Fields](#fields)
- [Endpoints](#endpoints)
  - [List all refunds from a customer](#list-all-refunds-from-a-customer)
  - [Get a refund](#get-a-refund)

## Embedded Resources

### CreditNote

| Field | Type | Notes |
|---|---|---|
| `id` | integer | - |
| `object` | string | Always `CreditNote` |
| `amount` | float | - |
| `currency` | string | ISO currency code |
| `status` | string | - |

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | `true` | Read | - |
| `object` | string | - | Read | Always `Refund` |
| `amount` | float | `true` | Read | The refund amount |
| `credit_note` | json | - | Read | Associated CreditNote resource (if any) |
| `currency` | string | - | Read | ISO currency code |
| `date_created` | datetime | - | Read | The date the refund was created at the external service |
| `external_service` | string | - | Read | The payment gateway that processed the refund |
| `operation_id` | string | - | Read | The external service's operation identifier |
| `original_payment` | json | - | Read | The original Payment resource that was refunded |
| `payment_type` | string | - | Read | The payment method type (e.g., `credit_card`, `debit_card`) |
| `status` | string | `true` | Read | Valid values: `pending`, `approved`, `in_process`, `failed`, `rejected` |
| `status_detail` | string | - | Read | Additional status information from the payment gateway |

## Endpoints

### List all refunds from a customer

- `GET /api/v1/customers/{customer_id}/refunds`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/customers/1/refunds \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "data": [...],
  "meta": {
    "total": 5,
    "per_page": 25,
    "pages": 1
  },
  "links": {
    "self": "https://<subdomain>.boxful.io/api/v1/customers/1/refunds?page=1",
    "first": "https://<subdomain>.boxful.io/api/v1/customers/1/refunds?page=1",
    "prev": null,
    "next": null,
    "last": "https://<subdomain>.boxful.io/api/v1/customers/1/refunds?page=1"
  }
}
```

### Get a refund

- `GET /api/v1/refunds/{refund_id}`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/refunds/1 \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "id": 1,
  "object": "Refund",
  "amount": 100.0,
  "credit_note": {
    "id": 1,
    "object": "CreditNote",
    "amount": 100.0,
    "currency": "ARS",
    "status": "refunded"
  },
  "currency": "ARS",
  "date_created": "2024-01-15T10:30:00.000-03:00",
  "external_service": "MercadoPago",
  "operation_id": "123456789",
  "original_payment": {
    "id": 1,
    "object": "Payment",
    "amount": 100.0,
    "currency": "ARS",
    "customer_id": 1,
    "date_approved": "2024-01-10T14:22:00.000-03:00",
    "date_created": "2024-01-10T14:22:00.000-03:00",
    "invoice_id": 1,
    "payment_type": "credit_card",
    "status": "approved",
    "status_detail": "accredited"
  },
  "payment_type": "credit_card",
  "status": "approved",
  "status_detail": "accredited"
}
```
