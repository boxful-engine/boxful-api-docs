# Invoices

An Invoice has many Payments, many InvoiceItems and many Discounts.

- [Embedded Resources](#embedded-resources)
- [Fields](#fields)
- [Endpoints](#endpoints)
  - [List all invoices](#list-all-invoices)
  - [Get an invoice](#get-an-invoice)
  - [Update an invoice](#update-an-invoice)

## Embedded Resources

### InvoiceItem

An InvoiceItem belongs to an Invoice. These are the individual items that make up an Invoice. Invoice items can only be accessed through their invoice — this resource has no endpoints.

| Field | Type | Read/Write | Notes |
|---|---|---|---|
| `id` | integer | Read | - |
| `object` | string | Read | Resource type |
| `amount` | float | Read | - |
| `base_amount` | float | Read | - |
| `category` | integer | Read | Valid values: `article`, `flat_fee`, `delivery_price`, `setup_fee`, `per_unit` |
| `currency` | string | Read | - |
| `description` | string | Read | - |
| `discounted_amount` | float | Read | - |
| `item_type` | string | Read | - |
| `item_id` | integer | Read | - |

### Discount

A Discount belongs to an Invoice. Discounts can only be accessed through their invoice — this resource has no endpoints.

| Field | Type | Read/Write | Notes |
|---|---|---|---|
| `object` | string | Read | Resource type |
| `discount_type` | string | Read | - |
| `discount_id` | integer | Read | - |
| `code` | string | Read | - |
| `promotion_type` | string | Read | Valid values: `percent_off`, `amount_off` |
| `amount` | float | Read | - |

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | `true` | Read | - |
| `object` | string | - | Read | Resource type |
| `amount` | float | `true` | Read | Calculated automatically |
| `created_at` | datetime | - | Read | - |
| `currency` | string | - | Read | - |
| `customer_id` | integer | `true` | Read | - |
| `discounts` | array | - | Read | Array of Discount resources representing Plan or Coupon discounts |
| `due_date` | date | `true` | Read | - |
| `invoice_items` | array | - | Read | Array of InvoiceItem resources that compose the Invoice |
| `last_attempt_date` | date | - | Read | - |
| `next_attempt_date` | date | - | Read | - |
| `paid_at` | datetime | - | Read | - |
| `reason_id` | integer | - | Read | The associated resource's id |
| `reason_type` | string | - | Read | Valid values: `Subscription`, `SubscriptionCart`, `PaymentInstruction` |
| `status` | string | `true` | Read and Write (only update) | Valid values: `cancelled`, `charged_back`, `dunning`, `paid`, `pending`, `refunded`, `scheduled`, `unpaid` |
| `updated_at` | datetime | - | Read | - |
| `customer_doc_number` | string | - | Read | Customer's document number |
| `customer_doc_type` | string | - | Read | Customer's document type |
| `customer_email` | string | - | Read | Customer's email |
| `customer_external_reference` | string | - | Read | Customer's external reference |

## Endpoints

### List all invoices

- `GET /api/v1/invoices`

#### Filtering options

| Field | Valid operators | Valid filter values |
|---|---|---|
| `status` | `is`, `is_not` | Invoice statuses |
| `due_date` | `is`, `gt`, `lt` | `YYYY-MM-DD` |
| `customer_id` | `is` | - |
| `paid_at` | `gt`, `lt` | `YYYY-MM-DD` |

###### Example request

```shell
curl -s "https://<subdomain>.boxful.io/api/v1/invoices?status[is]=paid&due_date[gt]=2020-08-01&due_date[lt]=2020-10-01" \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "data": [...],
  "meta": {
    "total": 34,
    "per_page": 25,
    "pages": 2
  },
  "links": {
    "self": "https://<subdomain>.boxful.io/api/v1/invoices?page=1",
    "first": "https://<subdomain>.boxful.io/api/v1/invoices?page=1",
    "prev": null,
    "next": "https://<subdomain>.boxful.io/api/v1/invoices?page=2",
    "last": "https://<subdomain>.boxful.io/api/v1/invoices?page=2"
  }
}
```

### Get an invoice

- `GET /api/v1/invoices/{invoice_id}`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/invoices/1 \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "id": 1,
  "object": "Invoice",
  "amount": 90.0,
  "created_at": "2020-09-30T17:26:38.698-03:00",
  "currency": "ARS",
  "customer_id": 33,
  "discounts": [
    {
      "object": "Discount",
      "discount_type": "Coupon",
      "discount_id": 1,
      "code": "NEW_COUPON",
      "promotion_type": "percent_off",
      "amount": 10
    }
  ],
  "due_date": "2020-10-22",
  "invoice_items": [
    {
      "id": 3,
      "object": "InvoiceItem",
      "amount": 90.0,
      "base_amount": 100.0,
      "category": "flat_fee",
      "currency": "ARS",
      "description": "Default plan description",
      "discounted_amount": 10.0,
      "item_type": "Plan",
      "item_id": 1
    }
  ],
  "last_attempt_date": null,
  "next_attempt_date": null,
  "paid_at": null,
  "reason_id": 33,
  "reason_type": "Subscription",
  "status": "scheduled",
  "updated_at": "2020-09-30T17:26:38.698-03:00"
}
```

### Update an invoice

- `PATCH /api/v1/invoices/{invoice_id}`

Only `status` changes are supported. Accepted target values are `cancelled` and `unpaid`.

#### Valid transitions

| From | To | Conditions |
|---|---|---|
| `scheduled` | `cancelled` | - |
| `dunning` | `unpaid` | No pending payment resolution |
| `dunning` | `cancelled` | Only when pending third-party collection |
| `unpaid` | `cancelled` | - |

###### Example request

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/invoices/1 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "invoice": { "status": "cancelled" } }'
```

###### Example response

Returns the updated Invoice resource.

###### Error responses

```json
// 400 Bad Request
{ "error": "Only 'cancelled' and 'unpaid' status changes are supported" }

// 422 Unprocessable Entity
{ "errors": { "status": ["cannot transition from current status"] } }
```
