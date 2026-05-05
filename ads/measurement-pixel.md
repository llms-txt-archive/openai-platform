# JavaScript Pixel

The OpenAI Ads Measurement Pixel is a browser SDK for measuring website events
after someone clicks an ad in ChatGPT. Use the pixel by adding the script to your site, initializing
it with your Pixel ID, and call `oaiq("measure", ...)` when a conversion
happens.

## Install the Measurement Pixel

Add the following snippet to the `<head>` section every page where you want to capture
conversions. Put the script near the top of your `<head>` to ensure early conversions aren't lost while other content loads.

```html
<script>
  (function (w, d, s, u) {
    if (w.oaiq) return;
    var q = function () {
      q.q.push(arguments);
    };
    q.q = [];
    w.oaiq = q;
    var js = d.createElement(s);
    js.async = true;
    js.src = u;
    var f = d.getElementsByTagName(s)[0];
    f.parentNode.insertBefore(js, f);
  })(window, document, "script", "https://bzrcdn.openai.com/sdk/oaiq.min.js");

  oaiq("init", {
    pixelId: "<YOUR-PIXEL-ID>",
  });
</script>
```

`pixelId` is required. Create a new pixelId in the conversions tab of Ads Manager. `debug` is optional and logs SDK activity to the browser
console while you test your integration.

## Send events

Call `oaiq("measure", eventName, eventProps, eventOptions)` when a conversion or
page action happens.

```js
oaiq("measure", eventName, eventProps, eventOptions);
```

| Argument       | Required | Description                                                                                               |
| -------------- | -------- | --------------------------------------------------------------------------------------------------------- |
| `eventName`    | Yes      | One of the <a href="/ads/supported-events">supported standard event names </a>, or `custom`.              |
| `eventProps`   | No       | Event data. In practice, send this for every event so the `type` and any optional fields are explicit.    |
| `eventOptions` | No       | Extra delivery options, including `event_id` for deduplication and `custom_event_name` for custom events. |

We support a set of <a href="/ads/supported-events">standard event names</a>. For custom events,
keep `eventName` as `custom`, set `eventProps.type` to `custom`, and put your
event label in `eventOptions.custom_event_name`.

> **Caution:** Put `custom_event_name` and `event_id` in `eventOptions`, not in
> `eventProps`.

## Example events

Use these examples as templates for common measurement patterns.

### Page and content views

```js
oaiq("measure", "page_viewed", {
  type: "contents",
  contents: [
    {
      id: "pricing",
      name: "Pricing page",
      content_type: "page",
    },
  ],
});

oaiq("measure", "contents_viewed", {
  type: "contents",
  contents: [
    {
      id: "sku_123",
      name: "Starter bundle",
      content_type: "product",
    },
  ],
});
```

### Commerce flow

Use the `contents` data shape for `items_added`, `checkout_started`, and
`order_created`.

```js
oaiq("measure", "items_added", {
  type: "contents",
  amount: 2599,
  currency: "USD",
  contents: [
    {
      id: "sku_123",
      name: "Starter bundle",
      content_type: "product",
      quantity: 1,
      amount: 2599,
      currency: "USD",
    },
  ],
});

oaiq("measure", "checkout_started", {
  type: "contents",
  amount: 2599,
  currency: "USD",
  contents: [
    {
      id: "sku_123",
      name: "Starter bundle",
      content_type: "product",
      quantity: 1,
    },
  ],
});

oaiq("measure", "order_created", {
  type: "contents",
  amount: 2599,
  currency: "USD",
  contents: [
    {
      id: "sku_123",
      name: "Starter bundle",
      content_type: "product",
      quantity: 1,
    },
  ],
});
```

### Lead generation and registration

Use the `customer_action` data shape for `lead_created`,
`registration_completed`, and `appointment_scheduled`.

```js
oaiq("measure", "lead_created", {
  type: "customer_action",
});

oaiq("measure", "registration_completed", {
  type: "customer_action",
});

oaiq("measure", "appointment_scheduled", {
  type: "customer_action",
  amount: 5000,
  currency: "USD",
});
```

### Subscription and trial events

Use the `plan_enrollment` data shape for `subscription_created` and
`trial_started`.

```js
oaiq("measure", "subscription_created", {
  type: "plan_enrollment",
  plan_id: "pro_monthly",
  amount: 2000,
  currency: "USD",
});

oaiq("measure", "trial_started", {
  type: "plan_enrollment",
  plan_id: "pro_trial",
});
```

### Custom events

For a custom event, keep `eventName` as `custom`, set `type` to `custom`, and
name the event in `eventOptions.custom_event_name`.

```js
oaiq(
  "measure",
  "custom",
  {
    type: "custom",
    amount: 12999,
    currency: "USD",
    plan_id: "enterprise_annual",
  },
  {
    custom_event_name: "quote_requested",
    event_id: "quote_req_123",
  }
);
```

## Deduplicate browser and server events

If you send the same conversion from both the Measurement Pixel and a
server-side integration, reuse the same `event_id` in both places.

```js
oaiq(
  "measure",
  "order_created",
  {
    type: "contents",
    amount: 2599,
    currency: "USD",
  },
  {
    event_id: "order_12345",
  }
);
```

The SDK generates an event ID when you omit one, which is enough for
client-only delivery. When you need deduplication across browser and server
events, generate the `event_id` yourself and reuse it on the same pixel and server-sent event. For custom events, keep
the same `custom_event_name` on both sides as well. Deduplication matches on
your Pixel ID, the event name, and `event_id`. For custom events,
`custom_event_name` replaces the standard event name in that match.

## What the SDK handles automatically

The Pixel handles several transport details for you:

- It captures `oppref` from the landing page URL, which is a privacy-preserving identifier
- It stores `oppref` in a first-party `__oppref` cookie so later page views can
  reuse it.
- It adds the current page origin as `source_url`.
- It timestamps each event and batches closely grouped `measure` calls.

No manual configuration of these details is necessary when using the pixel.

## Troubleshooting

- Keep `debug: true` while testing so you can inspect Pixel activity in the
  browser console.
- Use integer values for `amount` and `quantity`.
- Use only the documented fields inside `contents[]`.
- Always use the pixel on the browser. Do not call the server conversions API directly from page code.