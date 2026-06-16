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
| `amount` | float | `true` on create | Write | Also accepts integers. **On update:** rejected if any subscription already uses this plan (any status except `unselected`). |
| `base_plan` | BasePlan | - | Read | Parent BasePlan resource |
| `created_at` | datetime | - | Read | - |
| `currency` | string | - | Read | - |
| `metadata` | json | - | Write | - |
| `per_unit_price` | integer | - | Write | **On update:** rejected if any subscription already uses this plan (any status except `unselected`). |
| `per_unit_price_description` | string | - | Write | - |
| `per_unit_quantities_configuration` | json | - | Read | Object with `min`, `max`, and `step` (see the three fields below). Returned on read; on **create** you can send either this object under `plan` or the separate `_min` / `_max` / `_step` attributes. |
| `per_unit_quantities_configuration_min` | float | - | Write | For per-unit (non-metered) base plans: minimum quantity the customer can choose at checkout. |
| `per_unit_quantities_configuration_max` | float | - | Write | For per-unit (non-metered) base plans: maximum quantity the customer can choose at checkout. |
| `per_unit_quantities_configuration_step` | float | - | Write | For per-unit (non-metered) base plans: increment between allowed quantities (e.g. `1` allows 1, 2, 3, …). |
| `start_at` | datetime | - | Write | The date from which the plan's price will be effective. Must be unique per BasePlan. |
| `setup_fee` | float | - | Write | Also accepts integers |
| `setup_fee_description` | string | - | Write | - |
| `store_discount` | integer | - | Write | 0–100 |
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
    "start_at": "2020-03-25T13:05:14.699-03:00"
  }'
```

###### Example response

Returns the created Plan resource with its `base_plan`.

### Update a plan

- `PATCH /api/v1/plans/{plan_id}`

You may send any writable field from the [fields](#fields) table: for example `amount`, `store_discount`, `start_at`, `setup_fee`, `setup_fee_description`, `per_unit_price`, `per_unit_price_description`, `per_unit_quantities_configuration` (with nested `min`, `max`, `step`) or `per_unit_quantities_configuration_min`, `per_unit_quantities_configuration_max`, `per_unit_quantities_configuration_step`, and `metadata`. Standard validations apply (for example `start_at` must remain unique per base plan).

**Plans in use:** if at least one subscription uses this plan (any status except `unselected`), **`amount`** and **`per_unit_price`** cannot be updated; the API responds with `422`. To introduce a new price for new or migrating customers, create a new plan version with `POST /api/v1/base_plans/{base_plan_id}/plans` and a distinct `start_at`.

###### Example request (metadata)

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/plans/1 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "metadata": { "our_id": 4567 } }'
```

###### Example request (several fields under `plan`)

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/plans/1 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "plan": {
      "start_at": "2026-05-01T00:00:00.000-03:00",
      "setup_fee": 0,
      "metadata": { "our_id": 4567 }
    }
  }'
```

###### Example response

Returns the updated Plan resource (same shape as [Get a plan](#get-a-plan)).

###### Error responses

- `422 Unprocessable Entity` — validation failed (e.g. locked `amount` / `per_unit_price`, invalid `start_at`, or other model errors). Body includes an `errors` array of messages.
