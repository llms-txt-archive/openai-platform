# Conversions API

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

You can provision a Pixel ID and Conversions API key from the conversions tab on your Ads Manager account.
| Value | Required | Description |
| --------------- | -------- | ------------------------------------------------- |
| `pid` | Yes | Your Pixel ID. |
| `validate_only` | No | Validates events without saving them when `true`. |
| `events` | Yes | The events to send. |

The API accepts batches of up to 1,000 events. If one event in the batch fails,
the full batch fails.

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
  "data": {
    "type": "contents"
  }
}
```

| Field               | Required | Description                                                                                     |
| ------------------- | -------- | ----------------------------------------------------------------------------------------------- |
| `id`                | Yes      | Your unique event identifier. Used with `type` to deduplicate events.                           |
| `type`              | Yes      | A supported standard event name, or `custom`.                                                   |
| `timestamp_ms`      | Yes      | Event time in milliseconds. Must be within the last 7 days and no more than 10 minutes ahead.   |
| `custom_event_name` | Depends  | Required when `type` is `custom`.                                                               |
| `oppref`            | No       | OpenAI-provided privacy-preserving identifier.                                                  |
| `source_url`        | Depends  | Required when `action_source` is `web`.                                                         |
| `action_source`     | No       | One of `web`, `mobile_app`, `offline`, `physical_store`, `phone_call`, `email`, or `other`.     |
| `user`              | No       | Optional user fields that can improve attribution accuracy for advertising conversions.         |
| `opt_out`           | No       | Set to `true` to opt out the event from future user-level personalization. Defaults to `false`. |
| `data`              | Yes      | Event data matching the event type.                                                             |

See [Supported events](https://developers.openai.com/ads/supported-events) for event names and data shapes.

Unlike the pixel, the API does not capture `oppref` for you. Capture the value
yourself and pass it with the server event when it is available.

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

## Deduplicate browser and server events

If you send the same conversion from the pixel and the Conversions API, reuse
the same value as the API `id` and pixel `event_id`. Send both events with the
same Pixel ID. For custom events, use the same `custom_event_name` on both sides
as well.