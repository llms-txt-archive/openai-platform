# Campaigns

## List campaigns

List campaigns in the current ad account.

`GET /campaigns`

| Parameter | Type    | Required | Notes                              |
| --------- | ------- | -------- | ---------------------------------- |
| `limit`   | integer | No       | Between `1` and `500`. Default 20. |
| `after`   | string  | No       | Cursor for the next page.          |
| `before`  | string  | No       | Cursor for the previous page.      |
| `order`   | string  | No       | `asc` or `desc`.                   |

```bash
curl -X GET "https://api.ads.openai.com/v1/campaigns?limit=20&order=desc" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY"
```

```json
{
  "object": "list",
  "data": [
    {
      "id": "cmpn_101",
      "created_at": 1735689600,
      "updated_at": 1735776000,
      "name": "Spring launch",
      "description": "Promote the new productivity bundle.",
      "status": "active",
      "start_time": 1735689600,
      "end_time": 1738368000,
      "budget": {
        "lifetime_spend_limit_micros": 25000000
      }
    }
  ],
  "first_id": "cmpn_101",
  "last_id": "cmpn_101",
  "has_more": false
}
```

## Create a campaign

Create a campaign for the current ad account. The Ads belonging to a campaign will only show between the defined start and end time, and only in the countries specified in `targeting.locations.countries`.

### Defaults

If you exclude a start_time, the campaign will begin delivering immediately, and if you exclude a country, the campaign will target all countries.

Note that time and currency fields will respect your account-set timezone and currency defaults.

`POST /campaigns`

| Field                                    | Type     | Required | Notes                                                       |
| ---------------------------------------- | -------- | -------- | ----------------------------------------------------------- |
| `name`                                   | string   | Yes      | `3` to `1000` chars and must include a non-space character. |
| `description`                            | string   | No       | Campaign description.                                       |
| `start_time`                             | integer  | No       | Unix timestamp between `946684800` and `4102444800`.        |
| `end_time`                               | integer  | No       | Unix timestamp between `946684800` and `4102444800`.        |
| `status`                                 | string   | Yes      | `active` or `paused`.                                       |
| `budget.lifetime_spend_limit_micros`     | integer  | Yes      | Minimum `1000000`.                                          |
| `targeting.locations.countries`          | string[] | No       | Included country codes.                                     |
| `targeting.excluded_locations.countries` | string[] | No       | Excluded country codes.                                     |

```bash
curl -X POST "https://api.ads.openai.com/v1/campaigns" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Spring launch",
    "description": "Promote the new productivity bundle.",
    "start_time": 1735689600,
    "end_time": 1738368000,
    "status": "active",
    "budget": {
      "lifetime_spend_limit_micros": 25000000
    },
    "targeting": {
      "locations": {
        "countries": ["US", "CA"]
      }
    }
  }'
```

```json
{
  "id": "cmpn_101",
  "created_at": 1735689600,
  "updated_at": 1735689600,
  "name": "Spring launch",
  "description": "Promote the new productivity bundle.",
  "status": "active",
  "start_time": 1735689600,
  "end_time": 1738368000,
  "budget": {
    "lifetime_spend_limit_micros": 25000000
  },
  "targeting": {
    "locations": {
      "countries": ["US", "CA"]
    }
  }
}
```

## Retrieve a campaign

Fetch one campaign by ID.

`GET /campaigns/{campaign_id}`

```bash
curl -X GET "https://api.ads.openai.com/v1/campaigns/cmpn_101" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY"
```

## Update a campaign

Update a campaign with `POST`, not `PATCH` or `PUT`.

`POST /campaigns/{campaign_id}`

All fields are optional on update. If you include `budget`, send the full
budget object. `description`, `start_time`, `end_time`, and `targeting` can be
set to `null` to clear them. `status` accepts `active`, `paused`, or
`archived`.

```bash
curl -X POST "https://api.ads.openai.com/v1/campaigns/cmpn_101" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "description": "Updated launch window and budget.",
    "status": "paused",
    "budget": {
      "lifetime_spend_limit_micros": 30000000
    }
  }'
```

## Change state with dedicated actions

The Ads API also exposes explicit state transitions. Each endpoint returns the
updated campaign object. Paused campaigns won't deliver ads to customers. Only archive objects you have no further use for, as archiving isn't reversible.

- `POST /campaigns/{campaign_id}/activate`
- `POST /campaigns/{campaign_id}/pause`
- `POST /campaigns/{campaign_id}/archive`

```bash
curl -X POST "https://api.ads.openai.com/v1/campaigns/cmpn_101/pause" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY"
```

```json
{
  "id": "cmpn_101",
  "created_at": 1735689600,
  "updated_at": 1735862400,
  "name": "Spring launch",
  "description": "Promote the new productivity bundle.",
  "status": "paused",
  "start_time": 1735689600,
  "end_time": 1738368000,
  "budget": {
    "lifetime_spend_limit_micros": 25000000
  }
}
```