# Insights

## Insights endpoints

The Ads API exposes the same reporting envelope at four levels:

- `GET /ad_account/insights`
- `GET /campaigns/{campaign_id}/insights`
- `GET /ad_groups/{ad_group_id}/insights`
- `GET /ads/{ad_id}/insights`

Use the endpoint that matches the scope you want to get results for. Each endpoint
returns the same top-level response shape, with IDs and metadata appropriate to
that scope.

| Parameter           | Type     | Required | Notes                                                             |
| ------------------- | -------- | -------- | ----------------------------------------------------------------- |
| `time_granularity`  | string   | No       | Aggregation bucket size: `daily` or `none`.                       |
| `aggregation_level` | string   | No       | Aggregation scope: `ad_account`, `campaign`, `ad_group`, or `ad`. |
| `limit`             | integer  | No       | Between `1` and `10000`.                                          |
| `before`            | string   | No       | Cursor for the previous page.                                     |
| `after`             | string   | No       | Cursor for the next page.                                         |
| `time_ranges`       | string[] | No       | A filter on time range.                                           |
| `filters`           | string[] | No       | One or more filter expressions.                                   |
| `fields`            | string[] | No       | Fields to project in each row.                                    |
| `sort`              | string[] | No       | Sort expressions.                                                 |

## Examples

These examples use clicks as the ranking metric for best-performing ads. You can
sort by another supported metric if that better matches how you measure
performance.



First, find the best-performing ad across the full window.

```bash
curl -sS -G "https://api.ads.openai.com/v1/campaigns/cmpn_101/insights" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  --data-urlencode 'time_granularity=none' \
  --data-urlencode 'aggregation_level=ad' \
  --data-urlencode 'limit=1' \
  --data-urlencode 'fields[]=ad_id' \
  --data-urlencode 'fields[]=ad_name' \
  --data-urlencode 'fields[]=clicks' \
  --data-urlencode 'fields[]=impressions' \
  --data-urlencode 'time_ranges[]={"type":"date_range","since":"2026-04-25","until":"2026-05-01"}' \
  --data-urlencode 'sort[]={"field":"clicks","direction":"desc"}'
```

Then use the returned `ad_id` to get daily performance for that ad.

```bash
curl -sS -G "https://api.ads.openai.com/v1/ads/$AD_ID/insights" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  --data-urlencode 'time_granularity=daily' \
  --data-urlencode 'fields[]=readable_time' \
  --data-urlencode 'fields[]=clicks' \
  --data-urlencode 'fields[]=impressions' \
  --data-urlencode 'fields[]=ctr' \
  --data-urlencode 'time_ranges[]={"type":"date_range","since":"2026-04-25","until":"2026-05-01"}'
```





Use the ad account endpoint to rank ads across the whole project.

```bash
curl -sS -G "https://api.ads.openai.com/v1/ad_account/insights" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  --data-urlencode 'time_granularity=none' \
  --data-urlencode 'aggregation_level=ad' \
  --data-urlencode 'limit=1' \
  --data-urlencode 'fields[]=ad_id' \
  --data-urlencode 'fields[]=ad_name' \
  --data-urlencode 'fields[]=campaign_name' \
  --data-urlencode 'fields[]=clicks' \
  --data-urlencode 'fields[]=impressions' \
  --data-urlencode 'time_ranges[]={"type":"date_range","since":"2026-04-25","until":"2026-05-01"}' \
  --data-urlencode 'sort[]={"field":"clicks","direction":"desc"}'
```





Use a seven-day `date_range` with daily granularity to get one row per day.

```bash
curl -sS -G "https://api.ads.openai.com/v1/campaigns/cmpn_101/insights" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  --data-urlencode 'time_granularity=daily' \
  --data-urlencode 'fields[]=readable_time' \
  --data-urlencode 'fields[]=clicks' \
  --data-urlencode 'fields[]=impressions' \
  --data-urlencode 'fields[]=spend' \
  --data-urlencode 'time_ranges[]={"type":"date_range","since":"2026-04-25","until":"2026-05-01"}'
```



Each insights row includes `id`, `start_time`, and `end_time`, and can also
include derived bucket fields such as `readable_time` and `timezone`, delivery
metrics such as `impressions`, `clicks`, `spend`, `ctr`, `cpc`, and `cpm`, and
resource metadata such as `campaign_name`, `ad_group_name`, and `ad_name`.