# Products

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

If your feed uses a Google-compatible product data format, OpenAI will use that
formatting; see the [Google product data
specification](https://support.google.com/merchants/answer/7052112?hl=en#basic_product_data)
for field definitions.



<h2 id="feed-reference">Feed Reference</h2>

      This reference defines the shared flat-file schema that OpenAI ingests and
      indexes. For a non-Ads product feed, follow the Required and Optional
      labels on this page. Ads product feeds follow the [Ads product feeds
      guide](https://developers.openai.com/ads/product-feeds#use-the-correct-feed-schema) and use the same
      base schema plus its additional eligibility requirement.

      Each table below groups fields by schema object and indicates whether a
      field is Required or Optional for a non-Ads feed, along with validation
      rules to help your engineering team build and maintain a compliant upload
      file.

      Supplying all required fields ensures your products can be displayed
      correctly. For Ads processing, also set `is_ads_eligible` to `true` on
      each product that Ads should process.




### Google-compatible product data feeds

If OpenAI confirms that your registered feed supports this format, you can
upload a compatible delimited product data feed without renaming its columns to
OpenAI field names. Otherwise, continue to use the OpenAI specification. OpenAI
maps the supported input fields into the stable product schema and ignores
columns that it does not use.

This compatibility path covers the core profile described in this section. It
does not support every program, market, attribute, or feed representation. For
field definitions, see the [product data specification](https://support.google.com/merchants/answer/7052112?hl=en#basic_product_data).

#### Meet the core requirements

- Upload a UTF-8, tab-delimited `.txt` or `.tsv` file, or a comma-delimited
  `.csv` file. Gzip-compressed `.txt.gz`, `.txt.gzip`, `.tsv.gz`, and `.csv.gz`
  files are also supported.
- Include one header row with canonical lowercase, underscore-separated field
  names. For example, use `image_link`. `.txt` uploads also accept common
  space-separated display labels such as `image link`. `g:`-prefixed XML names
  are not supported.
- Include one product or variant per row.
- Include `id`, `title`, `description`, `link`, `image_link`, `availability`,
  `price`, and `brand` on every row.
- Use plain `title` and `description` values. Structured title and description
  alternatives are not mapped.
- Use valid HTTP or HTTPS URLs for `link` and `image_link`.
- Write `price` as an amount followed by a three-letter currency code. If OpenAI
  confirms market-specific processing for your registered feed, use a currency
  configured for that feed. Most products require a positive amount. A zero
  price passes validation only for supported mobile-device categories with a
  valid `subscription_cost`. If present, `sale_price` must be positive, use the
  same currency, and be less than `price`.
- Provide a valid `gtin` or `mpn` when you omit `identifier_exists` or set it to
  `yes`. Set `identifier_exists` to `no` only when the product genuinely has no
  identifier.
- Include `availability_date` when `availability` is `preorder` or `backorder`.

Register a merchant display name that contains at least one non-whitespace
character and is not a placeholder such as `unknown`, `null`, or `n/a`. OpenAI
trims and uses that registered name as the seller identity on every row. An
uploaded `seller_name` cannot override it.

If OpenAI confirms market-specific processing for the registered feed, OpenAI
applies its configured currencies and target countries. In that case, OpenAI
rejects products whose price currency is not configured for the feed.

JSON, spreadsheet, XML, RSS, and Atom sources are not part of this compatibility
path. Export a supported delimited product data feed before uploading it.

#### How OpenAI selects the parser

OpenAI automatically selects one parser for each upload:

1. OpenAI samples records from every eligible file and checks the OpenAI product
   schema first. Empty files do not contribute a sample.
2. If the OpenAI parser does not accept at least one sampled record from every
   file that yielded records, OpenAI checks the Google-compatible profile.
3. OpenAI selects the Google-compatible profile only if it accepts at least one
   sampled record from each file that yielded records.
4. OpenAI uses the selected parser for every eligible file and row. It does not
   switch parsers from row to row.

A feed that matches the OpenAI schema keeps OpenAI field semantics. Selecting a
parser does not guarantee that every row is valid: each row is still validated
during processing. If neither sampled profile matches, OpenAI keeps the OpenAI
parser and reports its validation errors.

#### Field mapping




| Input field                                        | OpenAI field                                   | Handling                                                                                                                                                   |
| :------------------------------------------------- | :--------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                                               | `item_id`                                      | Copied as the stable product or variant ID.                                                                                                                |
| `title`                                            | `title`                                        | Copied as plain text, up to 150 characters.                                                                                                                |
| `description`                                      | `description`                                  | Copied as plain text, up to 5,000 characters.                                                                                                              |
| `brand`                                            | `brand`                                        | Copied as the product brand.                                                                                                                               |
| `seller_name`                                      | `seller_name`                                  | OpenAI replaces the uploaded value with the registered merchant name.                                                                                      |
| `link`                                             | `url`, `seller_url`                            | Copied to `url`. OpenAI derives `seller_url` from the link's scheme and authority.                                                                         |
| `image_link`                                       | `image_url`                                    | Copied as the main public image URL.                                                                                                                       |
| `additional_image_link`                            | `additional_image_urls`                        | Copies valid comma-separated image URLs. OpenAI omits malformed optional URLs, but a URL that contains a username or password rejects the row.             |
| `availability`                                     | `availability`                                 | Copies `in_stock`, `out_of_stock`, and `backorder`; maps `preorder` to `pre_order`.                                                                        |
| `availability_date`                                | `availability_date`                            | Copied when provided and required for `preorder` or `backorder`. A date without a time is normalized to the start of that day in UTC.                      |
| `price`                                            | `price`                                        | Copied with its three-letter currency. Must be positive except for the supported mobile-subscription case described below.                                 |
| `sale_price`                                       | `sale_price`                                   | Copied when positive, in the same currency as `price`, and less than `price`.                                                                              |
| `sale_price_effective_date`                        | `sale_price_start_date`, `sale_price_end_date` | Requires `sale_price` and a start before the end. Splits the input range into two fields. Date-only values use the start and end of the day in UTC.        |
| `gtin`                                             | `gtin`                                         | Removes spaces and dashes and keeps the first valid GTIN.                                                                                                  |
| `mpn`                                              | `mpn`                                          | Copied when provided.                                                                                                                                      |
| `identifier_exists`                                | Not stored                                     | Determines whether OpenAI requires a valid `gtin` or a nonempty `mpn`.                                                                                     |
| `condition`                                        | `condition`                                    | Copied when provided, with supported values normalized.                                                                                                    |
| `product_type`, `google_product_category`          | `product_category`                             | Uses the first nonempty comma-separated `product_type`; otherwise uses `google_product_category`.                                                          |
| `item_group_id`                                    | `group_id`, `listing_has_variations`           | Groups variants and marks the listing as having variations. Without this field, OpenAI uses `id` as `group_id` and treats the product as a single listing. |
| `item_group_title`                                 | `item_group_title`                             | Copied when provided.                                                                                                                                      |
| `color`, `size`, `material`, `age_group`, `gender` | Same field names and `variant_dict`            | Normalizes the matching OpenAI field. For grouped variants, adds the trimmed input value to `variant_dict`.                                                |
| `pattern`, `size_type`                             | `variant_dict`                                 | Adds the value to `variant_dict` for grouped variants.                                                                                                     |
| `size_system`                                      | `size_system`                                  | Copied when provided.                                                                                                                                      |
| `video_link`                                       | `video_url`                                    | Copied when it is a valid HTTP or HTTPS URL.                                                                                                               |
| `virtual_model_link`                               | `model_3d_url`                                 | Copied when it is a valid HTTP or HTTPS URL.                                                                                                               |
| `expiration_date`                                  | `expiration_date`                              | Copied when it includes a time zone: `YYYY-MM-DDThh:mmZ`, optionally with seconds, or the equivalent with a numeric UTC offset.                            |




For accepted products, OpenAI enables search and disables checkout. For an Ads
feed, Ads eligibility comes from the registered feed configuration. Uploaded
eligibility fields do not override these settings.

#### Fields accepted but not stored

OpenAI accepts the following input fields but does not store them in the
normalized product:




| Input field          | Current behavior                                                     |
| :------------------- | :------------------------------------------------------------------- |
| `adult`              | Accepted, but not used.                                              |
| `external_seller_id` | Accepted, but not used.                                              |
| `installment`        | Accepted, but not used.                                              |
| `is_bundle`          | Accepted, but not used.                                              |
| `is_digital`         | Accepted, but not used.                                              |
| `loyalty_program`    | Accepted, but not used.                                              |
| `multipack`          | Accepted, but not used.                                              |
| `subscription_cost`  | Used only to validate an allowed zero `price`; otherwise not stored. |




For the zero-price exception, `google_product_category` must identify a
supported mobile-device category and `subscription_cost` must use
`month|year:positive integer:positive amount CURRENCY`. Its currency must match
`price`.

OpenAI also accepts and drops unrecognized columns. Uploaded `url`,
`seller_url`, eligibility, checkout, return-policy, and Ads control fields do
not control the normalized product. An uploaded `url` or `seller_url` that
contains a username or password rejects the row; OpenAI redacts the credential
from the validation details.

Upload History does not create a diagnostic for accepted-but-unused or
unrecognized fields. It reports missing required columns and values in
supported fields that fail validation. OpenAI can reject one malformed product
while other valid rows in the upload continue processing.


### OpenAI Flags

Use these flags to control whether a product is discoverable or purchasable
inside ChatGPT. These fields do not affect how the product is displayed on your
own site. They simply enable or disable the ChatGPT integrations.

The `Requirement` column and `Required fields only` filter describe non-Ads
feeds unless a row says otherwise. For Ads product feeds, see the [Ads product
feeds guide](https://developers.openai.com/ads/product-feeds#use-the-correct-feed-schema).

| Attribute            | Data Type | Supported Values | Description                                                                                                                          | Example | Requirement                                             | Dependencies                       | Validation Rules  |
| :------------------- | :-------- | :--------------- | :----------------------------------------------------------------------------------------------------------------------------------- | :------ | :------------------------------------------------------ | :--------------------------------- | :---------------- |
| is_eligible_search   | Boolean   | `true`, `false`  | Controls whether the product can be surfaced in ChatGPT search results.                                                              | `true`  | Required                                                | —                                  | Lower-case string |
| is_eligible_checkout | Boolean   | `true`, `false`  | Allows direct purchase inside ChatGPT. `is_eligible_search` must be `true` for `is_eligible_checkout` to be enabled for the product. | `true`  | Required                                                | Requires `is_eligible_search=true` | Lower-case string |
| is_ads_eligible      | Boolean   | `true`, `false`  | Controls whether the product can be processed for ChatGPT ads. Use `is_eligible_ads` only as a legacy alias.                         | `true`  | Required for Ads processing; optional for non-Ads feeds | —                                  | Lower-case string |

### Basic Product Data

Provide the core identifiers and descriptive text needed to uniquely reference
each product. These fields establish the canonical record that ChatGPT Search
uses to display and link to your product.

| Attribute   | Data Type             | Supported Values | Description                              | Example                                        | Requirement | Dependencies | Validation Rules                            |
| :---------- | :-------------------- | :--------------- | :--------------------------------------- | :--------------------------------------------- | :---------- | :----------- | :------------------------------------------ |
| item_id     | String (alphanumeric) | —                | Merchant product ID (unique per variant) | `SKU12345`                                     | Required    | —            | Max 100 chars; must remain stable over time |
| gtin        | String (numeric)      | GTIN, UPC, ISBN  | Universal product identifier             | `123456789543`                                 | Optional    | —            | 8-14 digits; no dashes or spaces            |
| mpn         | String (alphanumeric) | —                | Manufacturer part number                 | `GPT5`                                         | Optional    | —            | Max 70 chars                                |
| title       | String (UTF-8 text)   | —                | Product title                            | `Men's Trail Running Shoes Black`              | Required    | —            | Max 150 chars; avoid all-caps               |
| description | String (UTF-8 text)   | —                | Full product description                 | `Waterproof trail shoe with cushioned sole...` | Required    | —            | Max 5,000 chars; plain text only            |
| url         | URL                   | RFC 1738         | Product detail page URL                  | `https://example.com/product/SKU12345`         | Required    | —            | Must resolve with HTTP 200; HTTPS preferred |

### Item Information

Capture the physical characteristics and classification details of the product.
This data helps ensure accurate categorization, filtering, and search
relevance.

| Attribute        | Data Type | Supported Values                                | Description          | Example                                                    | Requirement | Dependencies                                                | Validation Rules                               |
| :--------------- | :-------- | :---------------------------------------------- | :------------------- | :--------------------------------------------------------- | :---------- | :---------------------------------------------------------- | :--------------------------------------------- |
| brand            | String    | —                                               | Product brand        | `OpenAI`                                                   | Required    | —                                                           | Max 70 chars                                   |
| condition        | String    | —                                               | Condition of product | `new`                                                      | Optional    | —                                                           | Lower-case string                              |
| product_category | String    | Category taxonomy                               | Category path        | `Apparel & Accessories > Shoes`                            | Optional    | —                                                           | Use `>` separator                              |
| material         | String    | —                                               | Primary material(s)  | `Leather`                                                  | Optional    | —                                                           | Max 100 chars                                  |
| dimensions       | String    | `LxWxH unit`                                    | Overall dimensions   | `12x8x5 in`                                                | Optional    | —                                                           | Units required if provided                     |
| length           | String    | —                                               | Individual dimension | `10`                                                       | Optional    | Provide all three if using individual fields                | Use `dimensions_unit`                          |
| width            | String    | —                                               | Individual dimension | `10`                                                       | Optional    | Provide all three if using individual fields                | Use `dimensions_unit`                          |
| height           | String    | —                                               | Individual dimension | `10`                                                       | Optional    | Provide all three if using individual fields                | Use `dimensions_unit`                          |
| dimensions_unit  | String    | —                                               | Dimensions unit      | `in`                                                       | Optional    | Required if any of `length`, `width`, `height` are provided | Unit abbreviation (for example, `in`, `cm`)    |
| weight           | String    | —                                               | Product weight       | `1.5`                                                      | Optional    | —                                                           | Use `item_weight_unit`                         |
| item_weight_unit | String    | —                                               | Product weight unit  | `lb`                                                       | Optional    | Required if `weight` is provided                            | Unit abbreviation (for example, `lb`, `kg`)    |
| age_group        | Enum      | `newborn`, `infant`, `toddler`, `kids`, `adult` | Target demographic   | `adult`                                                    | Optional    | —                                                           | Lower-case string                              |
| ads_metadata     | Object    | —                                               | Ad metadata values   | `{ "bidding_tier": "high", "product_line": "essentials" }` | Optional    | —                                                           | JSON object with string keys and string values |

### Media

Supply visual and rich media assets that represent the product. High-quality
images and optional videos or 3D models improve user trust and engagement.

| Attribute             | Data Type | Supported Values | Description            | Example                              | Requirement | Dependencies | Validation Rules            |
| :-------------------- | :-------- | :--------------- | :--------------------- | :----------------------------------- | :---------- | :----------- | :-------------------------- |
| image_url             | URL       | RFC 1738         | Main product image URL | `https://example.com/image1.jpg`     | Required    | —            | JPEG/PNG; HTTPS preferred   |
| additional_image_urls | String    | —                | Extra images           | `https://example.com/image2.jpg,...` | Optional    | —            | Comma-separated list        |
| video_url             | URL       | RFC 1738         | Product video          | `https://youtu.be/12345`             | Optional    | —            | Must be publicly accessible |
| model_3d_url          | URL       | RFC 1738         | 3D model               | `https://example.com/model.glb`      | Optional    | —            | GLB/GLTF preferred          |

### Price & Promotions

Define standard and promotional pricing information. These attributes power
price display, discount messaging, and offer comparisons.

| Attribute                           | Data Type         | Supported Values | Description                 | Example                    | Requirement | Dependencies | Validation Rules                      |
| :---------------------------------- | :---------------- | :--------------- | :-------------------------- | :------------------------- | :---------- | :----------- | :------------------------------------ |
| price                               | Number + currency | ISO 4217         | Regular price               | `79.99 USD`                | Required    | —            | Must include currency code            |
| sale_price                          | Number + currency | ISO 4217         | Discounted price            | `59.99 USD`                | Optional    | —            | Must be less than or equal to `price` |
| sale_price_start_date               | Date              | ISO 8601         | Sale start date             | `2025-07-01`               | Optional    | —            | Must be valid ISO 8601 date           |
| sale_price_end_date                 | Date              | ISO 8601         | Sale end date               | `2025-07-15`               | Optional    | —            | Must be valid ISO 8601 date           |
| unit_pricing_measure / base_measure | Number + unit     | —                | Unit price and base measure | `16 oz / 1 oz`             | Optional    | —            | Both fields required together         |
| pricing_trend                       | String            | —                | Lowest price in N months    | `Lowest price in 6 months` | Optional    | —            | Max 80 chars                          |

### Availability & Inventory

Describe current stock levels and key timing signals for product availability.
Accurate inventory data ensures users only see items they can actually
purchase.

| Attribute         | Data Type         | Supported Values                                                | Description                    | Example      | Requirement                          | Dependencies             | Validation Rules        |
| :---------------- | :---------------- | :-------------------------------------------------------------- | :----------------------------- | :----------- | :----------------------------------- | :----------------------- | :---------------------- |
| availability      | Enum              | `in_stock`, `out_of_stock`, `pre_order`, `backorder`, `unknown` | Product availability           | `in_stock`   | Required                             | —                        | Lower-case string       |
| availability_date | Date              | ISO 8601                                                        | Availability date if pre-order | `2025-12-01` | Required if `availability=pre_order` | —                        | Must be future date     |
| expiration_date   | Date              | ISO 8601                                                        | Remove product after date      | `2025-12-01` | Optional                             | —                        | Must be future date     |
| pickup_method     | Enum              | `in_store`, `reserve`, `not_supported`                          | Pickup options                 | `in_store`   | Optional                             | —                        | Lower-case string       |
| pickup_sla        | Number + duration | —                                                               | Pickup SLA                     | `1 day`      | Optional                             | Requires `pickup_method` | Positive integer + unit |

### Variants

Specify how related SKUs vary by size, color, or other options. Variant data
supports better matching, cleaner product grouping, and more precise product
recommendations.

| Attribute                | Data Type           | Supported Values | Description                            | Example                                                               | Requirement                          | Dependencies | Validation Rules                        |
| :----------------------- | :------------------ | :--------------- | :------------------------------------- | :-------------------------------------------------------------------- | :----------------------------------- | :----------- | :-------------------------------------- |
| group_id                 | String              | —                | Shared group identifier                | `SHOE123`                                                             | Recommended, if listing has variants | —            | Stable across related variants          |
| listing_has_variations   | Boolean             | `true`, `false`  | Indicates whether listing has variants | `true`                                                                | Recommended                          | —            | Lower-case string                       |
| variant_dict             | Object              | —                | Variant attributes map                 | A map of option names to values, such as color to Blue and size to 10 | Recommended, if listing has variants | —            | JSON object with string values          |
| item_group_title         | String (UTF-8 text) | —                | Group product title                    | `Men's Trail Running Shoes`                                           | Optional                             | —            | Max 150 chars; avoid all-caps           |
| color                    | String              | —                | Variant color                          | `Blue`                                                                | Optional                             | —            | Max 40 chars                            |
| size                     | String              | —                | Variant size                           | `10`                                                                  | Recommended (apparel)                | —            | Max 20 chars                            |
| size_system              | Country code        | ISO 3166         | Size system                            | `US`                                                                  | Recommended (apparel)                | —            | 2-letter country code                   |
| gender                   | String              | —                | Gender target                          | `male`                                                                | Optional                             | —            | Lower-case string                       |
| offer_id                 | String              | —                | Offer ID (SKU+seller+price)            | `SKU12345-Blue-79.99`                                                 | Optional                             | —            | Unique within feed                      |
| Custom_variant1_category | String              | —                | Custom variant dimension 1             | `Size_Type`                                                           | Optional (deprecated)                | —            | Deprecated; use `variant_dict` instead. |
| Custom_variant1_option   | String              | —                | Custom variant 1 option                | `Petite / Tall / Maternity`                                           | Optional (deprecated)                | —            | Deprecated; use `variant_dict` instead. |
| Custom_variant2_category | String              | —                | Custom variant dimension 2             | `Wood_Type`                                                           | Optional (deprecated)                | —            | Deprecated; use `variant_dict` instead. |
| Custom_variant2_option   | String              | —                | Custom variant 2 option                | `Oak / Mahogany / Walnut`                                             | Optional (deprecated)                | —            | Deprecated; use `variant_dict` instead. |
| Custom_variant3_category | String              | —                | Custom variant dimension 3             | `Cap_Type`                                                            | Optional (deprecated)                | —            | Deprecated; use `variant_dict` instead. |
| Custom_variant3_option   | String              | —                | Custom variant 3 option                | `Snapback / Fitted`                                                   | Optional (deprecated)                | —            | Deprecated; use `variant_dict` instead. |

### Fulfillment

Outline shipping methods, costs, and estimated delivery times. Providing
detailed shipping information helps users understand fulfillment options
upfront.

| Attribute  | Data Type | Supported Values                                                                                         | Description                         | Example                             | Requirement | Dependencies | Validation Rules                                                             |
| :--------- | :-------- | :------------------------------------------------------------------------------------------------------- | :---------------------------------- | :---------------------------------- | :---------- | :----------- | :--------------------------------------------------------------------------- |
| shipping   | String    | country:region:service_class:price:min_handling_days:max_handling_days:min_transit_days:max_transit_days | Shipping information                | `US:CA:Overnight:16.00 USD:1:2:1:3` | Optional    | —            | Omitting fields is allowed (`US::Overnight:16.00 USD`); use colon separators |
| is_digital | Boolean   | `true`, `false`                                                                                          | Indicates if the product is digital | `false`                             | Optional    | —            | Lower-case string                                                            |

### Merchant Info

Identify the seller and link to any relevant merchant policies or storefront
pages. This ensures proper attribution and enables users to review seller
credentials.

Note about 3P sellers and marketplaces: If your feed contains products that
are shipped with 3rd party sellers, please also include a
`marketplace_seller` in your feed. The `marketplace_seller` would be the point
of checkout in this scenario, and the `seller_name` would be the shipment
fulfiller.

| Attribute             | Data Type | Supported Values | Description                      | Example                       | Requirement                                | Dependencies | Validation Rules |
| :-------------------- | :-------- | :--------------- | :------------------------------- | :---------------------------- | :----------------------------------------- | :----------- | :--------------- |
| seller_name           | String    | —                | Seller name                      | `Example Store`               | Required / Display                         | —            | Max 70 chars     |
| marketplace_seller    | String    | —                | Marketplace seller of record     | `Marketplace Name`            | Optional                                   | —            | Max 70 chars     |
| seller_url            | URL       | RFC 1738         | Seller page                      | `https://example.com/store`   | Required                                   | —            | HTTPS preferred  |
| seller_privacy_policy | URL       | RFC 1738         | Seller-specific policies         | `https://example.com/privacy` | Required if `is_eligible_checkout` is true | —            | HTTPS preferred  |
| seller_tos            | URL       | RFC 1738         | Seller-specific terms of service | `https://example.com/terms`   | Required if `is_eligible_checkout` is true | —            | HTTPS preferred  |

### Returns

Provide return policies and time windows to set clear expectations for buyers.
Transparent return data builds trust and reduces post-purchase confusion.

Use `return_deadline_in_days` as the canonical field for return windows in the
feed schema.

| Attribute               | Data Type | Supported Values | Description             | Example                       | Requirement | Dependencies | Validation Rules  |
| :---------------------- | :-------- | :--------------- | :---------------------- | :---------------------------- | :---------- | :----------- | :---------------- |
| accepts_returns         | Boolean   | `true`, `false`  | Accepts returns         | `true`                        | Optional    | —            | Lower-case string |
| return_deadline_in_days | Integer   | Days             | Days allowed for return | `30`                          | Optional    | —            | Positive integer  |
| accepts_exchanges       | Boolean   | `true`, `false`  | Accepts exchanges       | `false`                       | Optional    | —            | Lower-case string |
| return_policy           | URL       | RFC 1738         | Return policy URL       | `https://example.com/returns` | Required    | —            | HTTPS preferred   |

### Performance Signals

Share popularity and return-rate metrics where available. These signals can be
used to enhance ranking and highlight high-performing products.

| Attribute        | Data Type | Supported Values | Description          | Example | Requirement | Dependencies | Validation Rules              |
| :--------------- | :-------- | :--------------- | :------------------- | :------ | :---------- | :----------- | :---------------------------- |
| popularity_score | Number    | —                | Popularity indicator | `4.7`   | Optional    | —            | 0-5 scale or merchant-defined |
| return_rate      | Number    | Percentage       | Return rate          | `2%`    | Optional    | —            | 0-100%                        |

### Compliance

Include regulatory warnings, disclaimers, or age restrictions. Compliance
fields help meet legal obligations and protect consumers.

| Attribute             | Data Type    | Supported Values | Description          | Example                                           | Requirement              | Dependencies | Validation Rules              |
| :-------------------- | :----------- | :--------------- | :------------------- | :------------------------------------------------ | :----------------------- | :----------- | :---------------------------- |
| warning / warning_url | String / URL | —                | Product disclaimers  | `Contains lithium battery, or CA Prop 65 warning` | Recommended for Checkout | —            | If URL, must resolve HTTP 200 |
| age_restriction       | Number       | —                | Minimum purchase age | `21`                                              | Recommended              | —            | Positive integer              |

### Reviews and Q&A

Supply aggregated review statistics and frequently asked questions.
User-generated insights strengthen credibility and help shoppers make informed
decisions.

| Attribute          | Data Type | Supported Values | Description                      | Example                                                                                                                       | Requirement | Dependencies | Validation Rules                                                                             |
| :----------------- | :-------- | :--------------- | :------------------------------- | :---------------------------------------------------------------------------------------------------------------------------- | :---------- | :----------- | :------------------------------------------------------------------------------------------- |
| review_count       | Integer   | —                | Number of product reviews        | `254`                                                                                                                         | Optional    | —            | Non-negative                                                                                 |
| star_rating        | String    | —                | Average review score             | `4.50`                                                                                                                        | Optional    | —            | 0-5 scale                                                                                    |
| store_review_count | Integer   | —                | Number of brand or store reviews | `2000`                                                                                                                        | Optional    | —            | Non-negative                                                                                 |
| store_star_rating  | String    | —                | Average store rating             | `4.50`                                                                                                                        | Optional    | —            | 0-5 scale                                                                                    |
| q_and_a            | List      | —                | FAQ content                      | A list of question and answer pairs, for example one question asking whether the item is waterproof and one answer saying Yes | Recommended | —            | List of objects containing string `q` and string `a` fields                                  |
| reviews            | List      | —                | Review entries                   | A list of review objects including title, content, minRating, maxRating, and rating values                                    | Recommended | —            | List of objects containing `title`, `content`, `minRating`, `maxRating`, and `rating` fields |

### Related Products

List products that are commonly bought together or act as substitutes. This
enables basket-building recommendations and cross-sell opportunities.

| Attribute          | Data Type | Supported Values                                                                                  | Description            | Example       | Requirement | Dependencies | Validation Rules             |
| :----------------- | :-------- | :------------------------------------------------------------------------------------------------ | :--------------------- | :------------ | :---------- | :----------- | :--------------------------- |
| related_product_id | String    | —                                                                                                 | Associated product IDs | `SKU67890`    | Recommended | —            | Comma-separated list allowed |
| relationship_type  | Enum      | `part_of_set`, `required_part`, `often_bought_with`, `substitute`, `different_brand`, `accessory` | Relationship type      | `part_of_set` | Recommended | —            | Lower-case string            |

### Geo Tagging

Indicate any region-specific pricing or availability overrides. Geo data allows
ChatGPT to present accurate offers and stock status by location.

| Attribute        | Data Type         | Supported Values             | Description                                     | Example                                     | Requirement | Dependencies | Validation Rules                     |
| :--------------- | :---------------- | :--------------------------- | :---------------------------------------------- | :------------------------------------------ | :---------- | :----------- | :----------------------------------- |
| target_countries | List              | `US`                         | Target countries of the item (first entry used) | `US`                                        | Required    | —            | Use ISO 3166-1 alpha-2 codes         |
| store_country    | String            | `US`                         | Store country of the item                       | `US`                                        | Required    | —            | Use ISO 3166-1 alpha-2 codes         |
| geo_price        | Number + currency | Region-specific price        | Price by region                                 | `79.99 USD (California)`                    | Optional    | —            | Must include ISO 4217 currency       |
| geo_availability | String            | Region-specific availability | Availability per region                         | `in_stock (Texas), out_of_stock (New York)` | Optional    | —            | Regions must be valid ISO 3166 codes |