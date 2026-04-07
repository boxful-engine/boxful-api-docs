# Orders

An Order belongs to an Invoice.

- [Fields](#fields)
- [Endpoints](#endpoints)
  - [List all orders](#list-all-orders)
  - [Get an order](#get-an-order)
  - [Update an order](#update-an-order)

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | - | Read | - |
| `object` | string | - | Read | Resource type |
| `address` | json | - | Read | Address resource |
| `created_at` | datetime | - | Read | - |
| `currency` | string | - | Read | - |
| `customer_id` | integer | - | Read | - |
| `invoice_due_date` | date | - | Read | - |
| `invoice_id` | integer | - | Read | - |
| `payment_status` | string | - | Read | Valid values: `paid`, `unpaid` |
| `updated_at` | datetime | - | Read | - |
| `customer_doc_number` | string | - | Read | Customer's document number |
| `customer_doc_type` | string | - | Read | Customer's document type |
| `customer_email` | string | - | Read | Customer's email |
| `customer_external_reference` | string | - | Read | Customer's external reference |

## Endpoints

### List all orders

- `GET /api/v1/orders`

#### Filtering options

| Field | Valid operators | Valid filter values |
|---|---|---|
| `customer_id` | `is` | - |
| `invoice_due_date` | `is`, `gt`, `lt` | `YYYY-MM-DD` |

###### Example request

```shell
curl -s "https://<subdomain>.boxful.io/api/v1/orders?customer_id[is]=17069&invoice_due_date[gt]=2022-08-01&invoice_due_date[lt]=2022-10-01" \
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
    "self": "https://<subdomain>.boxful.io/api/v1/orders?page=1",
    "first": "https://<subdomain>.boxful.io/api/v1/orders?page=1",
    "prev": null,
    "next": "https://<subdomain>.boxful.io/api/v1/orders?page=2",
    "last": "https://<subdomain>.boxful.io/api/v1/orders?page=2"
  }
}
```

### Get an order

- `GET /api/v1/orders/{order_id}`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/orders/4652 \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "id": 4652,
  "object": "Order",
  "address": {
    "city": null,
    "additional_info": null,
    "postal_code": "1233",
    "unit_number": "1",
    "street_number": "1234",
    "state": "Ciudad Autónoma de Buenos Aires",
    "street": "Sarmiento"
  },
  "created_at": "2022-08-03T16:59:31.354-03:00",
  "customer_id": 17069,
  "invoice_due_date": "2022-08-03",
  "invoice_id": 87975,
  "payment_status": "paid",
  "updated_at": "2022-08-04T17:11:31.183-03:00"
}
```

### Update an order

- `PATCH /api/v1/orders/{order_id}`

> [!WARNING]
> This endpoint was not part of the original API documentation. It may change without notice.

> **Restricted**: This endpoint is not available to all accounts. Contact your Boxful representative for access.

###### Example request

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/orders/4652 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "order": { "address": { "street": "New Street" } } }'
```
