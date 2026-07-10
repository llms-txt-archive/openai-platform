# Product Feed Spec

## Feed Reference

This reference describes non-Ads Commerce product feeds. If you are setting up
a product feed through Ads Manager, use the [Ads product feeds
guide](https://developers.openai.com/ads/product-feeds#use-the-correct-feed-schema), which explains the
additional Ads eligibility requirement.

For non-Ads Commerce feeds, merchants provide a structured product feed file
that OpenAI ingests and indexes. This specification defines the product schema
for file uploads: field names, data types, constraints, and example values
needed for accurate discovery, pricing, availability, and seller context.

Each table below groups fields by schema object and indicates whether a field
is Required or Optional for this non-Ads schema, along with validation rules to
help your engineering team build and maintain a compliant upload file.

Supplying all required fields ensures your products can be displayed correctly,
while optional fields enrich relevance and user trust.