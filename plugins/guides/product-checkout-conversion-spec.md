# Product checkout conversion spec

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

Product checkout conversion plugins in ChatGPT are currently in beta and being
  tested with approved partners. To apply for access, fill out this form 
  [here](https://chatgpt.com/merchants)

## Purpose

Our goal is to let ChatGPT directly invoke partner plugins for high-intent use
cases such as product checkout.

Once partners provide us with a product feed for search, we can connect their
MCP servers for bottom-of-funnel conversion actions. To do this, partner
plugins must follow a standardized contract for widget name, tool name, and
tool input.

If you want to build a plugin that follows this spec, apply for access through the
[ChatGPT merchants form](https://chatgpt.com/merchants/).

## User experience

When users search for products, the product entity sidebar can show **Open**
buttons for sellers. If a seller has a plugin, ChatGPT can open that plugin
inline for checkout instead of punching out to an external website.

## Required contract (today)

- Widget name: `ui://widget/checkout-session.html`
- Tool name: `checkout_session`

`checkout_session` must set:

```ts
_meta.ui.resourceUri = "ui://widget/checkout-session.html";
```

Any tool called directly from a widget must set:

```ts
_meta["openai/widgetAccessible"] = true;
```

## `checkout_session` input

Current input shape:

```json
{
  "checkout_session": {
    "items": [
      {
        "id": "string",
        "quantity": 1,
        "offerId": "string"
      }
    ]
  }
}
```

This payload aligns with the Commerce checkout session shape documented
[here](https://developers.openai.com/commerce/specs/checkout/#post-checkout_sessions).