# Overview

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

The Advertiser API lets you manage ad campaigns, ad groups, ads, files, and
reporting from one API. It supports CRUD-like functions with standard JSON content types.

## Authentication

Issue an API key in the Settings tab of [Ads Manager](https://ads.openai.com).
Each key is scoped to one ad account.

Pass the key as a bearer token on every request:

```bash
Authorization: Bearer $OPENAI_ADS_API_KEY
```

The Ads API works in the context of one ad account. API partners should use
  the key associated with the client account they are configuring. See [API
  partner setup](https://developers.openai.com/ads/api-partner-setup). To request partner access, [contact
  us](https://openai.com/advertisers/).

## Endpoints

| Resource                  | Use for                                                                         |
| ------------------------- | ------------------------------------------------------------------------------- |
| Campaigns                 | Create, list, retrieve, update, and change campaign state.                      |
| Ad Groups                 | Create, list, retrieve, update, and change ad group state.                      |
| Ads                       | Create, list, retrieve, update, and change ad state.                            |
| [Bulk API](https://developers.openai.com/ads/bulk-api) | Create or update campaigns, ad groups, and ads in an asynchronous job.          |
| Product feeds             | Use a merchant catalog to create product-feed campaigns.                        |
| Files                     | Upload creative assets for use in ads.                                          |
| Insights                  | Retrieve performance data across ad account, campaign, ad group, and ad scopes. |
| Conversions               | Create pixels, server-side keys, and conversion event settings when enabled.    |

Every resource belongs to the ad account associated with the API key.
Programmatic brand updates and conversion management require account
enablement. Contact your OpenAI partner representative if these operations are
not available for the account.

Use the [Quickstart](https://developers.openai.com/ads/api-quickstart) for a minimal end-to-end workflow, or go
directly to the [API reference](https://developers.openai.com/ads/api-reference/authentication). API partners
can start with [API partner setup](https://developers.openai.com/ads/api-partner-setup). To advertise
from a merchant catalog, follow the [product feeds guide](https://developers.openai.com/ads/product-feeds).

## Object Statuses

For an ad to show to users, the ad, and its parent ad group and campaign all have to be enabled. Further, the Ad has to be reviewed. Reviews typically only take a few minutes, you can monitor with the `review_status` field.

## Rate limits

The Advertiser API enforces limits by both ad account and IP address:

| Scope        | Limit                     |
| ------------ | ------------------------- |
| Per endpoint | 600 requests per minute   |
| Overall      | 1,200 requests per minute |

Requests must stay within both the ad-account and IP-address limits.

Bulk job creation has a separate limit of 10 requests per 10 seconds for each
ad account. See [Bulk API limits](https://developers.openai.com/ads/bulk-api#limits-and-retries).

## OpenAPI spec

[{"Download the OpenAPI spec"}](https://developers.openai.com/ads/openapi.json)

## Changelog

### July 16th, 2026

- Added support for passing the Pixel browser reference as `events[].user.obref` in [Conversions API](https://developers.openai.com/ads/conversions-api) requests.

### June 16th, 2026

- Added conversion-optimized campaign bidding with `bidding_type: "conversions"` and one standard conversion event setting.

### June 11th, 2026

- Added segmented insights for product, country, and device breakdowns, plus zero-impression product expansion.

### June 3rd, 2026

- Added location targeting support, including `/geo_lookup/search` and campaign `targeting.locations.include` for country, region, and DMA location IDs.
- Added conversion setup and reporting endpoints for API keys, pixels, event settings, and conversion insights.

### v1

- Published the initial API version.