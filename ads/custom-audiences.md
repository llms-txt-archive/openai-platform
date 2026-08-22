# Custom Audiences

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

Custom audiences let you use customer or prospect lists to control who can see
your ads. Upload email addresses or phone numbers, wait for the audience to
finish processing, then include or exclude matched customers in a campaign or
adjust bids for an ad group.

Custom audiences are not supported for campaigns targeting the European
  Economic Area (EEA) or Switzerland, where personalized ads are not yet
  available.

Before you begin, create an Ads API key in the **Settings** tab of
[Ads Manager](https://ads.openai.com). Store the key as
`OPENAI_ADS_API_KEY` and send it as a bearer token. Each key can access only
the audiences associated with its ad account.

Only upload first-party audience data that you have the right to use for ads.
Don't upload broker-sourced data. Before uploading, confirm that your use
complies with required rights, notices, consents, permissions, legal bases, and
the [Ad Tools Terms](https://openai.com/policies/ad-tools-terms/), and get
privacy or legal approval for your use case.

Campaign audience targeting and ad-group bid multipliers are enabled separately
for each account. If either supported request returns `403`, contact your
OpenAI representative to confirm that the required feature is enabled.

## Prepare an audience file

Create a UTF-8 CSV or TXT file that contains only one identifier type. Files
must not exceed 500 MB or contain more than 5,000,000 identifiers.

A TXT file must contain one identifier per line. A CSV file can optionally
include a header matching the selected identifier type:

| Identifier type       | CSV header            | Format                                                                                |
| --------------------- | --------------------- | ------------------------------------------------------------------------------------- |
| `email`               | `email`               | An email address containing one `@`. The API trims it and converts it to lowercase.   |
| `phone`               | `phone_number`        | A phone number in E.164 format, including `+` and the country code.                   |
| `email_sha256`        | `email_sha256`        | The 64-character SHA-256 hexadecimal digest of the normalized email address.          |
| `phone_number_sha256` | `phone_number_sha256` | The 64-character SHA-256 hexadecimal digest of the normalized E.164 telephone number. |

For example, an email audience CSV can contain:

```text
email
alex@example.com
jamie@example.com
sam@example.com
```

An audience generally needs about 25,000 matched users before you can use it for
targeting or bid adjustments. Uploading 25,000 identifiers doesn't guarantee
enough matches. The `ready` status from the API is the authoritative signal
that an audience can be used.

## Upload the audience file

Upload the CSV or TXT file to `POST /uploads`. Set the multipart `purpose` field
to `custom_audience`:

```bash
curl -X POST "https://api.ads.openai.com/v1/uploads" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  -F "file=@audience.csv;type=text/csv" \
  -F "purpose=custom_audience"
```

The response contains the file ID:

```json
{
  "file_id": "oaisdmntci_123"
}
```

Save the `file_id`, the original filename, the file's MIME type, and the exact
file size in bytes. You must provide these values when you create the audience.

## Create the custom audience

Send the uploaded file details to `POST /custom_audiences`:

```bash
curl -X POST "https://api.ads.openai.com/v1/custom_audiences" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "High-value customers",
    "description": "Customers eligible for the summer campaign",
    "file_id": "oaisdmntci_123",
    "identifier_type": "email",
    "filename": "audience.csv",
    "mimetype": "text/csv",
    "file_size": 123456
  }'
```

| Field             | Required | Description                                                        |
| ----------------- | -------- | ------------------------------------------------------------------ |
| `name`            | Yes      | Audience name containing at least three characters.                |
| `description`     | No       | A description of the audience.                                     |
| `file_id`         | Yes      | The file ID returned by `POST /uploads`.                           |
| `identifier_type` | No       | `email`, `phone`, `email_sha256`, or `phone_number_sha256`.        |
| `filename`        | Yes      | The uploaded filename, including its `.csv` or `.txt` extension.   |
| `mimetype`        | Yes      | The uploaded file's MIME type, such as `text/csv` or `text/plain`. |
| `file_size`       | Yes      | The exact file size in bytes, from `1` through `500000000`.        |

If you omit `identifier_type`, the API defaults to `email`. Set the identifier
type explicitly to make the request match the uploaded file.

The API returns the new audience and starts processing the uploaded file:

```json
{
  "id": "caud_123",
  "created_at": 1783962000,
  "updated_at": 1783962000,
  "name": "High-value customers",
  "description": "Customers eligible for the summer campaign",
  "status": "processing",
  "hash_spec_version": "custom_audience_join_hash_v1",
  "uploaded_identifier_count_range": "none",
  "matched_identifier_count_range": "none",
  "matched_user_count_range": "none",
  "invalid_identifier_count_range": "none",
  "membership_revision": 0
}
```

## Check processing status

Retrieve the audience with `GET /custom_audiences/{custom_audience_id}`:

```bash
curl "https://api.ads.openai.com/v1/custom_audiences/caud_123" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY"
```

Processing typically takes 20 to 30 minutes, depending on file size. Check the
audience periodically and wait until `status` is `ready` before using it.

| Status                   | Meaning                                                                     |
| ------------------------ | --------------------------------------------------------------------------- |
| `upload_pending`         | The uploaded file is waiting for processing to begin.                       |
| `processing`             | The file is being processed and the audience isn't ready to use.            |
| `rockset_ingest_pending` | Processed identifiers are waiting to be ingested.                           |
| `publishing`             | The audience is being prepared for targeting and bidding.                   |
| `ready`                  | Processing succeeded and the audience can be used for targeting or bidding. |
| `too_small`              | Too few users matched, so the audience can't be used.                       |
| `failed`                 | Processing failed. Check the file format, identifier type, and file limits. |
| `archived`               | The audience is archived and can no longer be used.                         |

The response returns identifier and matched-user counts as privacy-preserving
ranges, such as `under_25k`, `25k_100k`, `100k_500k`, `500k_1m`, and
`1m_5m`. Use these ranges for reporting only; wait for `status: ready` before
targeting or bidding.

## Update audience membership

List and retrieve responses include `membership_revision`. Pass it as
`expected_revision` when you want a stale membership change to fail instead of
overwriting a newer one.

Add and remove operations accept exactly one uploaded `file_id` or an
`identifiers` array. Each inline identifier includes its own `identifier_type`:

```bash
curl -X POST "https://api.ads.openai.com/v1/custom_audiences/caud_123/add" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: custom-audience-add-001" \
  -d '{
    "expected_revision": 0,
    "identifiers": [
      {
        "identifier_type": "email",
        "identifier": "new.customer@example.com"
      }
    ]
  }'
```

Use these endpoints for each membership change:

| Goal                      | Endpoint                                              | Required body fields           |
| ------------------------- | ----------------------------------------------------- | ------------------------------ |
| Add identifiers           | `POST /custom_audiences/{custom_audience_id}/add`     | `file_id` or `identifiers`     |
| Remove identifiers        | `POST /custom_audiences/{custom_audience_id}/remove`  | `file_id` or `identifiers`     |
| Replace all identifiers   | `POST /custom_audiences/{custom_audience_id}/replace` | `file_id`, `expected_revision` |
| Merge into a new audience | `POST /custom_audiences/merge`                        | `name`, `custom_audience_ids`  |

For file-based changes, upload a CSV or TXT file with `purpose=custom_audience`
first. A merge requires 2 through 64 source audiences and creates an independent
audience without changing its sources. During a replacement, the existing ready
audience stays available until the replacement publishes.

Every membership operation requires an `Idempotency-Key` header. Reuse the key
only to retry the same operation; retries return or resume the first accepted
input. The response contains a privacy-safe operation object:

```json
{
  "operation_id": "caudop_123",
  "custom_audience_id": "caud_123",
  "operation": "add",
  "status": "processing"
}
```

Poll `GET /custom_audiences/{custom_audience_id}/operations/{operation_id}`
until `status` is `succeeded` or `failed`. The response exposes only the
operation ID, audience ID, operation type, and status; it doesn't return raw
identifiers, matching counts, or individual membership outcomes.

## Include or exclude audiences in a campaign

Use ready audiences in the campaign's `targeting` object. Add audience IDs to
`custom_audiences.ids` to include matched users or
`excluded_custom_audiences.ids` to exclude them:

```bash
curl -X POST "https://api.ads.openai.com/v1/campaigns" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: custom-audience-campaign-001" \
  -d '{
    "name": "High-value customer campaign",
    "description": "Campaign targeting selected customer audiences",
    "status": "paused",
    "bidding_type": "clicks",
    "budget": {
      "lifetime_spend_limit_micros": 300000000
    },
    "targeting": {
      "locations": {
        "countries": ["US"]
      },
      "custom_audiences": {
        "ids": ["caud_123"]
      },
      "excluded_custom_audiences": {
        "ids": ["caud_456"]
      }
    }
  }'
```

Audience inclusion and exclusion work as follows:

- Include audiences to deliver only to users who belong to at least one
  included audience.
- Exclude audiences to prevent delivery to users who belong to an excluded
  audience.
- If you use both, exclusions take precedence and the remaining audience must
  still meet the minimum size requirement.
- Don't include and exclude the same audience in a campaign.

For the remaining campaign parameters, see
[Campaigns](https://developers.openai.com/ads/api-reference/campaigns).

## Adjust bids for an audience

Add `custom_audience_bid_multipliers` to an ad group's `bidding_config` to
raise or lower the maximum bid for a ready audience. Bid multipliers don't
change which users are eligible to see a campaign.

```bash
curl -X POST "https://api.ads.openai.com/v1/ad_groups" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: custom-audience-ad-group-001" \
  -d '{
    "campaign_id": "cmpn_123",
    "name": "High-value customers",
    "description": "Higher bid for selected customers",
    "status": "paused",
    "bidding_config": {
      "billing_event_type": "click",
      "max_bid_micros": 7500000,
      "custom_audience_bid_multipliers": [
        {
          "custom_audience_id": "caud_123",
          "bid_multiplier_micros": 2000000
        }
      ]
    }
  }'
```

Multipliers are expressed in millionths:

| `bid_multiplier_micros` | Bid multiplier |
| ----------------------- | -------------- |
| `100000`                | 0.1×           |
| `1000000`               | 1×             |
| `2000000`               | 2×             |
| `10000000`              | 10×            |

The supported range is `100000` through `10000000`. If a user matches multiple
configured audiences, the highest matching multiplier applies. For the
remaining ad group parameters, see [Ad Groups](https://developers.openai.com/ads/api-reference/ad-groups).

## List and archive audiences

List the custom audiences associated with your API key's ad account:

```bash
curl -G "https://api.ads.openai.com/v1/custom_audiences" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY" \
  --data-urlencode "limit=20"
```

Use the membership operations above to update an audience. Archive an audience
only when you no longer need it:

```bash
curl -X POST \
  "https://api.ads.openai.com/v1/custom_audiences/caud_123/archive" \
  -H "Authorization: Bearer $OPENAI_ADS_API_KEY"
```

Archiving is permanent. An archived audience can't be restored or used in
campaign targeting or bid adjustments.