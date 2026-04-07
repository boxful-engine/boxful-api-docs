# Plans

A Plan belongs to a BasePlan and works as a price version for it. The Plan retrieval response includes a `base_plan` object holding its basic information.

- [Fields](#fields)
- [Endpoints](#endpoints)
  - [List all plans from a base plan](#list-all-plans-from-a-base-plan)
  - [Get a plan](#get-a-plan)
  - [Create a plan](#create-a-plan)
  - [Update a plan](#update-a-plan)

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | `true` | Read | - |
| `object` | string | - | Read | Resource type |
| `amount` | float | `true` | Write (only create) | Also accepts integers |
| `base_plan` | BasePlan | - | Read | Parent BasePlan resource |
| `created_at` | datetime | - | Read | - |
| `currency` | string | - | Read | - |
| `main_discount_type` | string | `true` | Write (only create) | Valid values: `amount`, `percentage`, `none` |
| `main_discount_amount` | integer | `true` if `main_discount_type` is not `none` | Write (only create) | - |
| `metadata` | json | - | Write | - |
| `per_unit_price` | integer | - | Read | - |
| `per_unit_price_description` | string | - | Read | - |
| `per_unit_quantities_configuration` | json | - | Read | Minimum, maximum and step values for valid per unit quantities |
| `start_at` | datetime | - | Write (only create) | The date from which the plan's price will be effective |
| `setup_fee` | float | - | Write (only create) | Also accepts integers |
| `setup_fee_description` | string | - | Write (only create) | - |
| `updated_at` | datetime | - | Read | - |

## Endpoints

### List all plans from a base plan

- `GET /api/v1/base_plans/{base_plan_id}/plans`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/base_plans/1/plans \
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
    "self": "https://<subdomain>.boxful.io/api/v1/base_plans/1/plans?page=1",
    "first": "https://<subdomain>.boxful.io/api/v1/base_plans/1/plans?page=1",
    "prev": null,
    "next": null,
    "last": "https://<subdomain>.boxful.io/api/v1/base_plans/1/plans?page=1"
  }
}
```

### Get a plan

- `GET /api/v1/plans/{plan_id}`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/plans/1 \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "id": 1,
  "object": "Plan",
  "amount": 100.0,
  "base_plan": {
    "id": 1,
    "object": "BasePlan",
    "archived_at": null,
    "auto_selection_link": "https://<subdomain>.boxful.io/subscription_signups?base_plan=1",
    "code": "default",
    "cover_file_name": "berr.jpg",
    "debit_date": {
      "date_type": "cardinal",
      "object": "DebitDate",
      "value": 3
    },
    "deliverable": true,
    "external_reference": null,
    "metadata": {},
    "metered": false,
    "name": "Default Plan",
    "period": 1,
    "period_unit": "month",
    "periodicity": "1-month",
    "pricing_model": "flat_fee",
    "retry_schema": null,
    "return_url": "https://www.boxful.io",
    "selector_tag": "Default Plan",
    "setup_fee": false,
    "setup_fee_charged_at_free_trial_start": false,
    "short_description": "Default plan description",
    "subscription_completes_after_periods": null,
    "trial_period": null,
    "trial_period_unit": null,
    "unlisted": false
  },
  "created_at": "2020-03-25T13:05:14.723-03:00",
  "currency": "ARS",
  "main_discount_type": "amount",
  "main_discount_amount": 0,
  "metadata": null,
  "per_unit_price": null,
  "per_unit_price_description": null,
  "per_unit_quantities_configuration": null,
  "start_at": "2020-03-25T13:05:14.699-03:00",
  "setup_fee": 0.0,
  "setup_fee_description": null,
  "updated_at": "2020-03-25T13:05:14.723-03:00"
}
```

### Create a plan

- `POST /api/v1/base_plans/{base_plan_id}/plans`

###### Example request

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/base_plans/1/plans \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 100.0,
    "main_discount_type": "amount",
    "main_discount_amount": 0,
    "start_at": "2020-03-25T13:05:14.699-03:00"
  }'
```

###### Example response

Returns the created Plan resource with its `base_plan`.

### Update a plan

- `PATCH /api/v1/plans/{plan_id}`

###### Example request

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/plans/1 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "metadata": { "our_id": 4567 } }'
```

###### Example response

Returns the updated Plan resource.
