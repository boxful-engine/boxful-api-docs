# Payments

A Payment belongs to an Invoice and represents a payment attempt. A Payment can be rejected but its Invoice can still be paid if a subsequent payment attempt succeeds. For accurate account status tracking, consult the associated Invoice.

- [Fields](#fields)
- [Endpoints](#endpoints)
  - [List all payments from a customer](#list-all-payments-from-a-customer)
  - [Get a payment](#get-a-payment)
  - [Create a payment with invoice ID](#create-a-payment-with-invoice-id)
  - [Create a payment with external reference](#create-a-payment-with-external-reference)

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | `true` | Read | - |
| `object` | string | - | Read | Resource type |
| `amount` | float | `true` | Read | - |
| `currency` | string | - | Read | - |
| `customer_id` | integer | `true` | Read | - |
| `date_approved` | datetime | - | Read | Approval date at the external service |
| `date_created` | datetime | - | Read | Creation date at the external service |
| `invoice_id` | integer | `true` | Read | - |
| `payment_type` | string | - | Read | - |
| `reason_id` | integer | `true` | Read | - |
| `reason_type` | string | `true` | Read | Valid values: `SubscriptionCart` |
| `status` | string | `true` | Read | Valid values: `pending`, `approved`, `in_process`, `failed`, `rejected`, `cancelled`, `refunded`, `charged_back` |
| `status_detail` | string | - | Read | - |
| `customer_doc_number` | string | - | Read | Customer's document number |
| `customer_doc_type` | string | - | Read | Customer's document type |
| `customer_email` | string | - | Read | Customer's email |
| `customer_external_reference` | string | - | Read | Customer's external reference |

## Endpoints

### List all payments from a customer

- `GET /api/v1/customers/{customer_id}/payments`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/customers/1/payments \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "data": [...],
  "meta": {
    "total": 108,
    "per_page": 25,
    "pages": 5
  },
  "links": {
    "self": "https://<subdomain>.boxful.io/api/v1/customers/1/payments?page=1",
    "first": "https://<subdomain>.boxful.io/api/v1/customers/1/payments?page=1",
    "prev": null,
    "next": "https://<subdomain>.boxful.io/api/v1/customers/1/payments?page=2",
    "last": "https://<subdomain>.boxful.io/api/v1/customers/1/payments?page=5"
  }
}
```

### Get a payment

- `GET /api/v1/payments/{payment_id}`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/payments/1 \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "id": 1,
  "object": "Payment",
  "amount": 100.0,
  "currency": "ARS",
  "customer_id": 1,
  "date_approved": null,
  "date_created": null,
  "invoice_id": 1,
  "payment_type": "credit_card",
  "reason_id": 206,
  "reason_type": "Subscription",
  "status": "paid",
  "status_detail": ""
}
```

### Create a payment with invoice ID

- `POST /api/v1/invoices/{invoice_id}/payments`

This endpoint supports two modes of operation:

1. **Trigger a payment attempt**: When called without a request body, initiates a manual payment attempt for invoices that are eligible for automatic processing.

2. **Create an offline payment**: When called with a `payment` body, creates a manual payment record for payments received outside of the automated system (e.g., cash, bank transfers). Only available for invoices with status `scheduled` or `unpaid`.

#### Offline payment fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `payment_type` | string | `true` | Valid values: `cash`, `cbu` |
| `status` | string | `true` | Valid values: `approved`, `rejected` |
| `date_created` | datetime | - | The date when the payment was received. Defaults to current time if not provided |
| `comment` | string | - | Optional comment about the payment |

###### Example request (trigger payment attempt)

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/invoices/1/payments \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response (trigger payment attempt)

```json
{
  "description": "Payment attempt in process.",
  "success": true,
  "messages": {
    "client": "Procesamiento de pago iniciado exitosamente",
    "subscriber": "Su pago está siendo procesado"
  }
}
```

###### Example request (offline payment)

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/invoices/1/payments \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "payment": {
      "payment_type": "cash",
      "status": "approved",
      "comment": "Payment received in person"
    }
  }'
```

###### Error response

```json
{
  "description": "Invoice cannot be processed due to current status or processing stage.",
  "success": false,
  "messages": {
    "client": "La factura no puede ser procesada en su estado actual",
    "subscriber": "Esta factura no puede ser procesada en este momento"
  }
}
```

### Create a payment with external reference

- `POST /api/v1/invoices/{invoice_id}/payments?identifier=external_reference`

Initiates a manual payment attempt for the specified invoice using its `external_reference`.

**Note**: This endpoint requires the `identifier=external_reference` query parameter to identify invoices by their external reference.

###### Example request

```shell
curl -s -X POST "https://<subdomain>.boxful.io/api/v1/invoices/AVE18L0000000484628399/payments?identifier=external_reference" \
  -H "Authorization: Bearer $TOKEN"
```
