# Ads

## List ads

List ads for an ad group.

`GET /ads`

| Parameter     | Type    | Required | Notes                              |
| ------------- | ------- | -------- | ---------------------------------- |
| `ad_group_id` | string  | Yes      | Parent ad group ID.                |
| `limit`       | integer | No       | Between `1` and `500`. Default 20. |
| `after`       | string  | No       | Cursor for the next page.          |
| `before`      | string  | No       | Cursor for the previous page.      |
| `order`       | string  | No       | `asc` or `desc`.                   |

```bash
curl -X GET "https://api.ads.openai.com/v1/ads?ad_group_id=adgrp_301&limit=10" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY"
```

```json
{
  "object": "list",
  "data": [
    {
      "id": "ad_501",
      "name": "Planner launch card",
      "created_at": 1735689800,
      "updated_at": 1735776200,
      "creative": {
        "type": "chat_card",
        "title": "Try the new workspace planner",
        "body": "Coordinate tasks, docs, and meetings in one place.",
        "file_id": "file_901",
        "image_url": "https://cdn.openai.com/ads/file_901.png",
        "target_url": "https://example.com/workspace-planner"
      },
      "status": "active",
      "review_status": "approved"
    }
  ],
  "first_id": "ad_501",
  "last_id": "ad_501",
  "has_more": false
}
```

## Create an ad

Create an ad for an ad group.

`POST /ads`

| Field                 | Type   | Required | Notes                                                                                                         |
| --------------------- | ------ | -------- | ------------------------------------------------------------------------------------------------------------- |
| `ad_group_id`         | string | Yes      | Parent ad group ID.                                                                                           |
| `name`                | string | Yes      | `3` to `1000` chars and must include a non-space character. Used for organization, is not shown to end users. |
| `creative.type`       | string | Yes      | Today, must be `chat_card`.                                                                                   |
| `creative.title`      | string | Yes      | `3` to `50` chars.                                                                                            |
| `creative.body`       | string | Yes      | Maximum `100` chars.                                                                                          |
| `creative.target_url` | string | Yes      | Destination URL.                                                                                              |
| `creative.file_id`    | string | Yes      | File returned by `POST /upload`.                                                                              |
| `status`              | string | Yes      | `active` or `paused`.                                                                                         |

```bash
curl -X POST "https://api.ads.openai.com/v1/ads" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "ad_group_id": "adgrp_301",
    "name": "Planner launch card",
    "status": "active",
    "creative": {
      "type": "chat_card",
      "title": "Try the new workspace planner",
      "body": "Coordinate tasks, docs, and meetings in one place.",
      "target_url": "https://example.com/workspace-planner",
      "file_id": "file_901"
    }
  }'
```

## Retrieve an ad

Fetch one ad by ID.

`GET /ads/{ad_id}`

```bash
curl -X GET "https://api.ads.openai.com/v1/ads/ad_501" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY"
```

## Update an ad

Update an ad with `POST`.

`POST /ads/{ad_id}`

All fields are optional on update. If you include `creative`, send the full
creative object. `status` accepts `active`, `paused`, or `archived`.

```bash
curl -X POST "https://api.ads.openai.com/v1/ads/ad_501" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Planner launch card v2",
    "status": "paused",
    "creative": {
      "type": "chat_card",
      "title": "Plan work faster",
      "body": "Bring tasks, docs, and meetings together.",
      "target_url": "https://example.com/workspace-planner",
      "file_id": "file_901"
    }
  }'
```

## Review status

Every ad response includes `review_status`, which can be:

- `in_review`
- `rejected`
- `approved`

If your ad has been rejected, it violates one of our [ads policies](https://openai.com/policies/ad-policies/). Please edit your ad for it to be re-reviewed.

## Change state with dedicated actions

The Ads API also exposes explicit state transitions. Paused ads won't deliver to customers. Only archive objects you have no further use for, as archiving isn't reversible.

- `POST /ads/{ad_id}/activate`
- `POST /ads/{ad_id}/pause`
- `POST /ads/{ad_id}/archive`

```bash
curl -X POST "https://api.ads.openai.com/v1/ads/ad_501/pause" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY"
```

```json
{
  "id": "ad_501",
  "name": "Planner launch card",
  "created_at": 1735689800,
  "updated_at": 1736035200,
  "creative": {
    "type": "chat_card",
    "title": "Try the new workspace planner",
    "body": "Coordinate tasks, docs, and meetings in one place.",
    "file_id": "file_901",
    "image_url": "https://cdn.openai.com/ads/file_901.png",
    "target_url": "https://example.com/workspace-planner"
  },
  "status": "paused",
  "review_status": "approved"
}
```