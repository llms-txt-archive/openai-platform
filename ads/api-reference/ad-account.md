# Ad account

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

## Update account brand metadata

Set the account name or favicon and start a new brand review.
At least one of `name` or `favicon_file_id` is required.
This operation must be enabled for the ad account. If it returns `403`, contact
your OpenAI partner representative.

`POST /ad_account/brand`

| Field             | Type   | Required | Notes                                               |
| ----------------- | ------ | -------- | --------------------------------------------------- |
| `name`            | string | No       | Updated account display name.                       |
| `favicon_file_id` | string | No       | File ID uploaded with `purpose: "account_favicon"`. |

Upload the favicon with the [file endpoint](https://developers.openai.com/ads/api-reference/files#upload-an-account-favicon),
then assign it to the account:

```bash
curl -X POST "https://api.ads.openai.com/v1/ad_account/brand" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "favicon_file_id": "file_123"
  }'
```

The response includes the updated account. Poll `GET /ad_account` until
`review.status` is `approved`. An account with any other review status cannot
serve ads.

## Get ad account metadata

Fetch metadata for the current ad account.

`GET /ad_account`

This endpoint takes no request body or query parameters.

```bash
curl -X GET "https://api.ads.openai.com/v1/ad_account" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY"
```

```json
{
  "id": "adacct_123",
  "name": "Acme Ads",
  "url": "https://www.acme.example",
  "preview_url": null,
  "status": "active",
  "timezone": "UTC",
  "currency_code": "USD",
  "review": {
    "status": "approved"
  }
}
```

The response includes:

- `id` for the ad account
- `name` for the display name
- `url` for the primary destination
- `preview_url` for the favicon preview URL when one is available
- `status` when an account status is available
- `timezone` for the ad account timezone
- `currency_code` for the account currency
- `review` for the account's brand review status