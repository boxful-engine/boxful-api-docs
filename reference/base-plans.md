# Base Plans

A BasePlan holds the immutable information of a subscription plan such as `name`, `code` and `short_description`. A BasePlan has many Plans which are the objects that hold all price related information. On BasePlan creation a Plan will be created. To create subsequent price versions, Plans should be created (see [Create a Plan](plans.md#create-a-plan)).

A BasePlan has one DebitDate (optional) which determines when customers should be charged. If not defined, it falls back to the account's default DebitDate, and if that is not defined, customers are charged on their subscription's anniversary.

- [Embedded Resources](#embedded-resources)
- [Fields](#fields)
- [Endpoints](#endpoints)
  - [List all base plans](#list-all-base-plans)
  - [Get a base plan](#get-a-base-plan)
  - [Create a base plan](#create-a-base-plan)
  - [Update a base plan](#update-a-base-plan)
  - [List all period descriptions from a base plan](#list-all-period-descriptions-from-a-base-plan)

## Embedded Resources

### DebitDate

| Field | Type | Notes |
|---|---|---|
| `object` | string | Always `DebitDate` |
| `date_type` | string | `cardinal` or `ordinal` |
| `value` | integer/string | The day value |

### BasePlanPeriodDescription

A BasePlanPeriodDescription belongs to a BasePlan and can hold information specific for a defined period. For example, if a different product is delivered to customers each period, its `description` may hold the product's SKU number.

Period descriptions can be created in advance for upcoming periods, but the period dates cannot overlap.

An Order created between the period description's `start_date` and `end_date` may hold the defined `description` in its `metadata`.

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | `true` | Read | - |
| `object` | string | - | Read | Resource type |
| `start_date` | date | `true` | Read | The date when the period starts |
| `end_date` | date | `true` | Read | The date when the period ends |
| `description` | string | `true` | Read | For example, a product's SKU # |
| `base_plan_id` | integer | - | Read | The id of the BasePlan it belongs to |
| `created_at` | datetime | - | Read | - |
| `updated_at` | datetime | - | Read | - |

## Fields

| Field | Type | Required | Read/Write | Notes |
|---|---|---|---|---|
| `id` | integer | `true` | Read | - |
| `object` | string | - | Read | Resource type |
| `archived_at` | datetime | - | Read | If the BasePlan is not listed this will contain the date it was archived |
| `auto_selection_link` | string | - | Read | Link to checkout with current plan preselected (visitor must not be logged in) |
| `code` | string | `true` | Write | - |
| `cover_file_name` | string | - | Read | - |
| `created_at` | datetime | - | Read | - |
| `current_plan` | json | - | Read | Current associated Plan resource |
| `debit_date` | json | - | Read | Default is null (charged on subscription anniversary). If set, contains DebitDate resource |
| `deliverable` | boolean | `true` | Write | Whether the subscription object should be delivered. Default: false |
| `external_reference` | string | - | Read | The id representing the resource in an external service |
| `metadata` | json | - | Write | - |
| `metered` | boolean | - | Read | For per_unit pricing: whether consumption is metered (post-usage) vs preset |
| `name` | string | `true` | Write | - |
| `period` | integer | - | Read | Period component of the periodicity |
| `period_unit` | string | - | Read | Period unit component of the periodicity |
| `periodicity` | string | - | Read/Write (only create) | Allowed: `1-week`, `1-month`, `2-month`, `3-month`, `4-month`, `6-month`, `1-year`. Default: `1-month` |
| `pricing_model` | string | - | Read | Valid values: `flat_fee`, `per_unit`. Default: `flat_fee` |
| `retry_schema` | json | - | Read | RetrySchema associated resource |
| `return_url` | string | - | Write | Redirect URL after checkout completion |
| `selector_tag` | string | `true` | Write | - |
| `setup_fee` | boolean | - | Write (only create) | - |
| `setup_fee_charged_at_free_trial_start` | boolean | - | Write | - |
| `short_description` | string | `true` | Write | - |
| `subscription_completes_after_periods` | integer | - | Write (only create) | For finite plans: duration in periods |
| `trial_period` | integer | - | - | - |
| `trial_period_unit` | string | - | - | Valid values: `day`, `month` |
| `unlisted` | boolean | `true` | Write | Whether the BasePlan is visible to customers. Default: false |
| `updated_at` | datetime | - | Read | - |

## Endpoints

### List all base plans

- `GET /api/v1/base_plans/`

#### Filtering options

| Field | Valid operators | Valid filter values |
|---|---|---|
| `archived_at` | `is`, `is_not` | `null` |

###### Example request

```shell
curl -s "https://<subdomain>.boxful.io/api/v1/base_plans?archived_at[is]=null" \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "data": [...],
  "meta": {
    "total": 30,
    "per_page": 25,
    "pages": 2
  },
  "links": {
    "self": "https://<subdomain>.boxful.io/api/v1/base_plans?page=1",
    "first": "https://<subdomain>.boxful.io/api/v1/base_plans?page=1",
    "prev": null,
    "next": "https://<subdomain>.boxful.io/api/v1/base_plans?page=2",
    "last": "https://<subdomain>.boxful.io/api/v1/base_plans?page=2"
  }
}
```

### Get a base plan

- `GET /api/v1/base_plans/{base_plan_id}`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/base_plans/1 \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "id": 1,
  "object": "BasePlan",
  "archived_at": "2020-03-31T13:05:14.721-03:00",
  "auto_selection_link": "https://<subdomain>.boxful.io/subscription_signups?base_plan=1",
  "code": "default",
  "cover_file_name": "berr.jpg",
  "created_at": "2020-03-25T13:05:14.721-03:00",
  "current_plan": {
    "id": 52,
    "object": "Plan",
    "amount": 100.0,
    "currency": "ARS",
    "metadata": null,
    "per_unit_price": 0.0,
    "per_unit_price_description": null,
    "per_unit_quantities_configuration": null,
    "start_at": "2020-05-31T00:00:00.000-03:00",
    "setup_fee": 0.0,
    "setup_fee_description": null
  },
  "debit_date": {
    "date_type": "cardinal",
    "object": "DebitDate",
    "value": 3
  },
  "deliverable": true,
  "external_reference": null,
  "metadata": null,
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
  "unlisted": true,
  "updated_at": "2020-04-07T15:58:14.373-03:00"
}
```

### Create a base plan

- `POST /api/v1/base_plans`

###### Example request

```shell
curl -s -X POST https://<subdomain>.boxful.io/api/v1/base_plans \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "New Plan",
    "code": "newplan",
    "short_description": "New plan description",
    "selector_tag": "New Plan",
    "subscription_completes_after_periods": null,
    "plans_attributes": [
      {
        "amount": 100.0,
        "currency": "ARS",
        "metadata": null,
        "start_at": "2020-06-31",
        "metadata": { "a_key": "a_value" }
      }
    ],
    "debit_date_attributes": {
      "date_type": "cardinal",
      "object": "DebitDate",
      "value": 3
    }
  }'
```

###### Example response

Returns the created BasePlan resource with its `current_plan` and `debit_date`.

### Update a base plan

- `PATCH /api/v1/base_plans/{base_plan_id}`

###### Example request

```shell
curl -s -X PATCH https://<subdomain>.boxful.io/api/v1/base_plans/1 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "short_description": "new short description" }'
```

###### Example response

Returns the updated BasePlan resource.

### List all period descriptions from a base plan

- `GET /api/v1/base_plans/{base_plan_id}/base_plan_period_descriptions`

###### Example request

```shell
curl -s https://<subdomain>.boxful.io/api/v1/base_plans/1/base_plan_period_descriptions \
  -H "Authorization: Bearer $TOKEN"
```

###### Example response

```json
{
  "data": [
    {
      "id": 1,
      "object": "BasePlanPeriodDescription",
      "start_date": "17-10-2022",
      "end_date": "20-10-2022",
      "description": "The delivered product during this period is SKU #1234",
      "base_plan_id": 6,
      "created_at": "2022-10-18T12:34:25.852-03:00",
      "updated_at": "2022-10-19T10:07:37.238-03:00"
    }
  ],
  "meta": {
    "total": 5,
    "per_page": 25,
    "pages": 1
  },
  "links": {
    "self": "https://<subdomain>.boxful.io/api/v1/base_plans/1/base_plan_period_descriptions?page=1",
    "first": "https://<subdomain>.boxful.io/api/v1/base_plans/1/base_plan_period_descriptions?page=1",
    "prev": null,
    "next": null,
    "last": "https://<subdomain>.boxful.io/api/v1/base_plans/1/base_plan_period_descriptions?page=1"
  }
}
```
