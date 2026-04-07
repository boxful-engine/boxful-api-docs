# Credit Cards

> [!WARNING]
> This resource was not part of the original API documentation. It may change without notice.

> **Restricted**: These endpoints are not available to all accounts. Contact your Boxful representative for access.

Credit cards are stored payment methods associated with a customer. They are used for tokenized payment processing.

- [Fields](#fields)
- [Endpoints](#endpoints)
  - [List all credit cards from a customer](#list-all-credit-cards-from-a-customer)
  - [Create a credit card](#create-a-credit-card)

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `brand` | string | `true` | Write | Card brand (e.g., `visa`, `mastercard`) |
| `expiration_date` | string | `true` | Write | Card expiration date |
| `external_id` | string | `true` | Write | Token from the payment gateway |
| `first_six_digits` | string | `true` | Write | First six digits of the card number |
| `funding_type` | string | `true` | Write | Card funding type (e.g., `credit`, `debit`) |
| `last_four_digits` | string | `true` | Write | Last four digits of the card number |
| `customer_id` | integer | - | Read | Associated customer ID |
| `customer_doc_number` | string | - | Read | Customer's document number |
| `customer_doc_type` | string | - | Read | Customer's document type |
| `customer_email` | string | - | Read | Customer's email |
| `customer_external_reference` | string | - | Read | Customer's external reference |
| `created_at` | datetime | - | Read | - |
| `updated_at` | datetime | - | Read | - |

## Endpoints

### List all credit cards from a customer

- `GET /api/v1/customers/{customer_id}/credit_cards`

**Alternative identification**: `external_reference` can be used with `?identifier=external_reference`.

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/customers/14038/credit_cards \
  -H "Authorization: Bearer $TOKEN"
```

### Create a credit card

- `POST /api/v1/customers/{customer_id}/credit_cards`

**Alternative identification**: `external_reference` can be used with `?identifier=external_reference`.

###### Example request

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/customers/14038/credit_cards \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "credit_card": {
      "brand": "visa",
      "expiration_date": "12/2028",
      "external_id": "tok_abc123",
      "first_six_digits": "411111",
      "funding_type": "credit",
      "last_four_digits": "1234"
    }
  }'
```
