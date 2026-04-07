# Customers

A Customer represents an end-user within an account. Depending on your account setup other fields may be required.

- [Embedded Resources](#embedded-resources)
- [Fields](#fields)
- [Endpoints](#endpoints)
  - [List all customers](#list-all-customers)
  - [Get a customer](#get-a-customer)
  - [Create a customer](#create-a-customer)
  - [Update a customer](#update-a-customer)
  - [Create a customer's login token](#create-a-customers-login-token)

## Embedded Resources

### Address

| Field | Type | Notes |
|---|---|---|
| `city` | string | - |
| `additional_info` | string | - |
| `postal_code` | string | - |
| `unit_number` | string | - |
| `street_number` | string | - |
| `state` | string | - |
| `street` | string | - |

### CustomField

Custom fields allow storing additional information for customers.

| Field | Type | Notes |
|---|---|---|
| `name` | string | The field name |
| `field_type` | string | Type of field: `string`, `text`, or `date` |
| `content_string` | string | **DEPRECATED** Value for string type fields |
| `content_text` | string | **DEPRECATED** Value for text type fields |
| `content_date` | string | **DEPRECATED** Value for date type fields (format: YYYY-MM-DD) |
| `value` | string/date | Value for string, text, or date type fields |

**Note**: When updating custom fields by name, the system will automatically match existing fields or create new ones. The `value` attribute provides a simpler, more consistent API compared to the `content_*` fields and is recommended for new integrations.

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | - | Read | - |
| `object` | string | - | Read | Resource type |
| `address` | json | true | Read/Write (only create) | Address resource |
| `area_code` | string | `true` if Phone is a required field during checkout | Read/Write | 1-5 digits. No symbols |
| `birthdate` | date | - | Write | - |
| `custom_fields` | array | true if present | Read/Write | Array of CustomField resources |
| `description` | text | - | Write | - |
| `doc_number` | string | - | Write | - |
| `doc_type` | string | - | Write | - |
| `email` | string | `true` | Write | Must be unique |
| `external_reference` | string | - | Write | Must be unique |
| `fname` | string | `true` | Write | - |
| `legal_name` | string | - | Write | - |
| `lname` | string | `true` | Write | - |
| `metadata` | json | - | Write | - |
| `phone` | string | `true` if Phone is a required field during checkout | Read/Write | 5-10 digits. No symbols |
| `phone_country_code` | string | `true` if Phone is a required field during checkout | Read/Write | 1-7 digits, no symbols. E.g: `54`, `1` |
| `subscription` | json | - | Read | **Deprecated**: The platform now supports multiple subscriptions. Until this key is removed it will return the first subscription. |
| `tax_id_number` | string | - | Write | - |
| `tax_id_type` | string | - | Write | - |
| `terms_accepted_at` | datetime | - | Read | - |
| `created_at` | datetime | - | Read | - |
| `updated_at` | datetime | - | Read | - |

## Endpoints

### List all customers

- `GET /api/v1/customers/`

#### Filtering options

| Field | Valid operators | Valid filter values |
|---|---|---|
| `email` | `is` | - |
| `external_reference` | `is` | Valid external reference or `null` (as a string) |
| `tax_id_number` | `is` | Valid tax id number or `null` (as a string) |
| `doc_number` | `is` | Valid national identity document number or `null` (as a string) |

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/customers?email[is]=user@mail.com \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "data": [...],
  "meta": {
    "total": 40,
    "per_page": 25,
    "pages": 2
  },
  "links": {
    "self": "https://<subdomain>.boxful.io/api/v1/customers?page=1",
    "first": "https://<subdomain>.boxful.io/api/v1/customers?page=1",
    "prev": null,
    "next": "https://<subdomain>.boxful.io/api/v1/customers?page=2",
    "last": "https://<subdomain>.boxful.io/api/v1/customers?page=2"
  }
}
```

### Get a customer

- `GET /api/v1/customers/{customer_id}`

**Alternative identification**: Instead of using the `customer_id`, the `external_reference` can be used as a parameter. To do so it must be specified in the query string that the `external_reference` is being used as an identifier.

For a Customer with `id` `14038` and `external_reference` `a1b2c3d4` these options are equivalent:
- `GET /api/v1/customers/14038`
- `GET /api/v1/customers/a1b2c3d4?identifier=external_reference`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/customers/14038 \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "id": 14038,
  "object": "Customer",
  "address": {
    "city": "Avellaneda",
    "additional_info": null,
    "postal_code": "Actualizar datos e informar",
    "unit_number": "8C",
    "street_number": "4123",
    "state": "Buenos Aires",
    "street": "Niceto Vega"
  },
  "area_code": "11",
  "birthdate": "1991-05-01",
  "custom_fields": [
    {
      "id": "4",
      "type": "custom_field",
      "attributes": {
        "name": "Empresa donde trabaja",
        "field_type": "string",
        "content_string": "ACME Co.",
        "content_text": null,
        "content_date": null,
        "value": "ACME Co."
      }
    }
  ],
  "description": null,
  "doc_number": null,
  "doc_type": null,
  "email": "juan.perez@mail.com",
  "external_reference": null,
  "fname": "Juan",
  "legal_name": null,
  "lname": "Perez",
  "metadata": "{}",
  "phone": "12345678",
  "phone_country_code": "54",
  "subscription": {
    "id": 100,
    "object": "Subscription",
    "last_invoice": { "..." : "..." },
    "next_invoice": { "..." : "..." },
    "payment_type": "credit_card",
    "status": "authorized"
  },
  "tax_id_type": null,
  "tax_id_number": null,
  "terms_accepted_at": null
}
```

### Create a customer

- `POST /api/v1/customers`

The successful response contains a `msgver` that can be used to create a magic link that will automatically log in the customer (only valid for 2 hours).

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/customers \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "area_code": 11,
    "address": {
      "street": "Niceto Vega",
      "street_number": 4123,
      "unit_number": "8C",
      "state": "Buenos Aires",
      "city": "Avellaneda"
    },
    "birthdate": "1991-05-01",
    "custom_fields": [
      {
        "name": "Empresa donde trabaja",
        "field_type": "string",
        "content_string": "ACME Co."
      },
      {
        "name": "Fecha de inicio",
        "field_type": "date",
        "value": "2024-01-15"
      }
    ],
    "doc_number": 12345678,
    "doc_type": "DNI",
    "description": "This is a long text with my comments or description",
    "email": "juan.perez@mail.com",
    "fname": "Juan",
    "lname": "Perez",
    "metadata": {
      "our_id": 123,
      "customer": "Prime Customer"
    },
    "phone": 12345678,
    "phone_country_code": "54"
  }'
```

###### Example response

```json
{
  "id": 178,
  "object": "Customer",
  "address": {
    "city": "Avellaneda",
    "additional_info": "This is a long text with my comments or description",
    "postal_code": "Actualizar datos e informar",
    "unit_number": "8C",
    "street_number": "4123",
    "state": "Buenos Aires",
    "street": "Niceto Vega"
  },
  "area_code": "11",
  "birthdate": "1991-05-01",
  "custom_fields": [
    {
      "id": "17",
      "type": "custom_field",
      "attributes": {
        "name": "Empresa donde trabaja",
        "field_type": "string",
        "content_string": "ACME Co.",
        "content_text": null,
        "content_date": null
      }
    }
  ],
  "description": "This is a long text with my comments or description",
  "doc_number": "12345678",
  "doc_type": "DNI",
  "email": "juan.perez@mail.com",
  "external_reference": null,
  "fname": "Juan",
  "legal_name": null,
  "lname": "Perez",
  "metadata": {},
  "msgver": "BAh7BzoMdXNlcl9pZGkBsjoPZXhwaXJhdGlvbkkiHjIw...",
  "phone": "12345678",
  "phone_country_code": "54",
  "subscription": {
    "id": 169,
    "object": "Subscription",
    "last_invoice": null,
    "next_invoice": null,
    "payment_type": "credit_card",
    "status": "unselected"
  },
  "tax_id_number": null,
  "tax_id_type": null,
  "terms_accepted_at": null
}
```

### Update a customer

- `PATCH /api/v1/customers/{customer_id}`

**Alternative identification**: Same as Get a customer — `external_reference` can be used with `?identifier=external_reference`.

###### Example request

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/customers/14038 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "customer": {
      "lname": "Perez Updated",
      "custom_fields": [
        {
          "name": "Empresa donde trabaja",
          "value": "New Company SA"
        }
      ]
    }
  }'
```

###### Example response

Returns the updated Customer resource.

### Create a customer's login token

- `POST /api/v1/customers/{customer_id}/login_token`

**Alternative identification**: Same as Get a customer — `external_reference` can be used with `?identifier=external_reference`.

The `msgver` can be used to create a magic link that will automatically log in the customer. The token expires.

###### Example request

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/customers/14038/login_token \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "msgver": "BAh7BzoMdXNlcl9pZGkGOg9leHBpcmF0aW9uSSIeMjAy...",
  "expiration": "2020-07-22 18:17:15 -0300"
}
```
