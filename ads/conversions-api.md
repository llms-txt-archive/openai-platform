# Conversions API

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

The Conversions API is a more reliable tracking source than the pixel alone. We encourage using the Conversions API when possible for more accurate insights.

## Send events

Send events to the Conversions API from your server only.

```bash
curl -X POST "https://bzr.openai.com/v1/events?pid=<PIXEL-ID>" \
  -H "Authorization: Bearer <API-KEY>" \
  -H "Content-Type: application/json" \
  --data '{
    "validate_only": false,
    "events": []
  }'
```

You can provision a Pixel ID and Conversions API key from the conversions tab in Ads Manager. Approved API partners can use the Ads API key associated with a client account to provision both resources with the [conversion setup endpoints](https://developers.openai.com/ads/api-reference/conversion-setup).

| Value                | Required | Description                                              |
| -------------------- | -------- | -------------------------------------------------------- |
| `pid`                | Yes      | Your Pixel ID.                                           |
| `validate_only`      | No       | Validates events without saving them when `true`.        |
| `integration_source` | No       | Stable identifier for the integration sending the batch. |
| `events`             | Yes      | The events to send.                                      |

The API accepts batches of up to 1,000 events. If one event in the batch fails,
the full batch fails.

For app lifecycle events, use the Pixel ID from an existing web data source.
Send `app_installed` and `app_opened` from your server with `action_source`
set to `mobile_app`. Native mobile SDK setup and mobile data sources are not
currently supported.

## Identify partner integrations

If you send events on behalf of advertisers, include `integration_source` at
the top level of every Conversions API request. Mobile measurement partners
and other integrations should use the same stable identifier on every request,
such as `acme_measurement` or `example_analytics`. The value applies to every
event in the batch.

For example, a measurement partner can identify itself when sending an app
install event:

```json
{
  "integration_source": "acme_measurement",
  "events": [
    {
      "id": "app_installed_123",
      "type": "app_installed",
      "timestamp_ms": <TIMESTAMP_MS>,
      "action_source": "mobile_app",
      "data": {
        "type": "customer_action"
      }
    }
  ]
}
```

Replace `<TIMESTAMP_MS>` with the event timestamp in milliseconds.

Use 1–64 ASCII characters. Start with a letter or digit, and use only letters,
digits, periods (`.`), underscores (`_`), or hyphens (`-`). The API trims
whitespace and converts the value to lowercase before validation.

Use `integration_source` to identify the integration sending the request. This
field does not affect authentication or authorization.

## Event structure

Each event includes the event metadata and a `data` object.

```json
{
  "id": "order_12345",
  "type": "order_created",
  "timestamp_ms": 1773892800000,
  "oppref": "oppref_abc",
  "source_url": "https://shop.example.com/checkout/confirmation",
  "action_source": "web",
  "user": {
    "obref": "123e4567-e89b-42d3-a456-426614174000",
    "email_sha256": "b4c9a289323b21a01c3e940f150eb9b8c542587f1abfd8f0e1cc1ffc5e475514",
    "external_id_sha256": "73d83a078369bb4f0971b317aa7797a91cf5c0df1b62161c2e47d75c33ab5b6e",
    "country": "US",
    "city": "San Francisco",
    "zip_code": "94107",
    "ip_address": "203.0.113.1",
    "user_agent": "Mozilla/5.0"
  },
  "data": {
    "type": "contents"
  }
}
```

| Field               | Required | Description                                                                                                                                                                |
| ------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                | Yes      | Your unique event identifier. Used with `type` to deduplicate events.                                                                                                      |
| `type`              | Yes      | A supported standard event name, or `custom`.                                                                                                                              |
| `timestamp_ms`      | Yes      | Event time in milliseconds. Must be within the last 7 days and no more than 10 minutes ahead.                                                                              |
| `custom_event_name` | Depends  | Required when `type` is `custom`.                                                                                                                                          |
| `oppref`            | No       | OpenAI-provided privacy-preserving identifier.                                                                                                                             |
| `source_url`        | Depends  | Required when `action_source` is `web`.                                                                                                                                    |
| `action_source`     | Depends  | One of `web`, `mobile_app`, `offline`, `physical_store`, `phone_call`, `email`, or `other`. For `app_installed` and `app_opened`, you must set this field to `mobile_app`. |
| `user`              | No       | Optional user fields that can improve attribution accuracy for advertising conversions.                                                                                    |
| `opt_out`           | No       | Set to `true` to opt out the event from future user-level personalization. Defaults to `false`.                                                                            |
| `data`              | Yes      | Event data matching the event type.                                                                                                                                        |

See [Supported Events](https://developers.openai.com/ads/supported-events) for event names and data shapes.

Unlike the pixel, the API does not capture `oppref` for you. Capture the value
yourself and pass it with the server event when it is available.

## Send user data

Add an optional `user` object to each event to improve conversion matching. The
object is event-scoped, so put it inside each entry in `events`, not at the
request root.

Every field in the `user` object is optional. Include only the fields you have
for the user.

### User object example

Place this object inside an event at `events[].user`:

```json
{
  "obref": "123e4567-e89b-42d3-a456-426614174000",
  "email_sha256": "b4c9a289323b21a01c3e940f150eb9b8c542587f1abfd8f0e1cc1ffc5e475514",
  "external_id_sha256": "73d83a078369bb4f0971b317aa7797a91cf5c0df1b62161c2e47d75c33ab5b6e",
  "country": "US",
  "city": "San Francisco",
  "zip_code": "94107",
  "ip_address": "203.0.113.1",
  "user_agent": "Mozilla/5.0"
}
```

| Field                | Description                                                                                        |
| -------------------- | -------------------------------------------------------------------------------------------------- |
| `obref`              | Opaque browser reference from the Pixel's `__obref` cookie. Pass it without hashing.               |
| `email_sha256`       | SHA-256 hash of the email address after trimming whitespace and converting it to lowercase.        |
| `external_id_sha256` | SHA-256 hash of a stable, pseudonymous customer identifier from your system.                       |
| `country`            | Two-letter ISO 3166-1 country code, such as `US`.                                                  |
| `city`               | City name, with a maximum of 128 characters. OpenAI trims whitespace and converts it to lowercase. |
| `zip_code`           | Postal or ZIP code. Use letters, numbers, spaces, or hyphens, with a maximum of 32 characters.     |
| `ip_address`         | Valid IPv4 or IPv6 address.                                                                        |
| `user_agent`         | Non-empty user agent string from the client that generated the event.                              |

For hybrid Pixel and Conversions API integrations, read the `__obref`
first-party cookie in the browser, send it to your server, and include it
unchanged as `events[].user.obref` when available. Send a non-blank string.
Before collecting or forwarding the cookie, follow your site's measurement
consent requirements. If the user revokes consent, stop sending it. Unlike
`oppref`, which is an event-level field, `obref` belongs inside `user`.

Send hashes as lowercase, 64-character hexadecimal strings. Send the geographic,
IP address, and user agent fields as raw values. Don't send raw email addresses,
raw external IDs, phone numbers, or phone number hashes.

## Example event

```bash
curl -X POST "https://bzr.openai.com/v1/events?pid=<PIXEL-ID>" \
  -H "Authorization: Bearer <API-KEY>" \
  -H "Content-Type: application/json" \
  --data '{
    "validate_only": false,
    "events": [
      {
        "id": "order_12345",
        "type": "order_created",
        "timestamp_ms": 1773892800000,
        "oppref": "oppref_abc",
        "source_url": "https://shop.example.com/checkout/confirmation",
        "action_source": "web",
        "user": {
          "obref": "123e4567-e89b-42d3-a456-426614174000",
          "email_sha256": "b4c9a289323b21a01c3e940f150eb9b8c542587f1abfd8f0e1cc1ffc5e475514",
          "external_id_sha256": "73d83a078369bb4f0971b317aa7797a91cf5c0df1b62161c2e47d75c33ab5b6e",
          "country": "US",
          "city": "San Francisco",
          "zip_code": "94107",
          "ip_address": "203.0.113.1",
          "user_agent": "Mozilla/5.0"
        },
        "data": {
          "type": "contents",
          "amount": 2599,
          "currency": "USD",
          "contents": [
            {
              "id": "sku_123",
              "name": "Starter bundle",
              "content_type": "product",
              "quantity": 1
            }
          ]
        }
      }
    ]
  }'
```

## App lifecycle events

App lifecycle events use the `customer_action` data shape and require
`action_source` to be `mobile_app`.

### App installed

```json
{
  "id": "app_installed_123",
  "type": "app_installed",
  "timestamp_ms": <TIMESTAMP_MS>,
  "action_source": "mobile_app",
  "data": {
    "type": "customer_action"
  }
}
```

### App opened

```json
{
  "id": "app_opened_123",
  "type": "app_opened",
  "timestamp_ms": <TIMESTAMP_MS>,
  "action_source": "mobile_app",
  "data": {
    "type": "customer_action"
  }
}
```

## Deduplicate browser and server events

If you send the same conversion from the pixel and the Conversions API, reuse
the same value as the API `id` and pixel `event_id`. Send both events with the
same Pixel ID. For custom events, use the same `custom_event_name` on both sides
as well.