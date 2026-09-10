# Campaign Targeting

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

Use campaign targeting to choose who can see your ads and where they can appear.
You can combine location, platform, and custom audience targeting in a campaign.
Choose a guide for the targeting criteria you want to configure.

## Location targeting




Choose the countries, regions, or Markets where your campaign can deliver. See
[Location Targeting](https://developers.openai.com/ads/location-targeting) to find location IDs and use them
in a campaign.

## Platform targeting

Choose the ChatGPT apps and web browsers where your campaign can deliver. See
[Platform Targeting](https://developers.openai.com/ads/platform-targeting) for supported platform values,
examples, and how to update or clear a platform selection.

## Custom audiences

Include customer or prospect lists in campaign targeting, or exclude audiences
you don't want to reach. See
[Custom Audiences](https://developers.openai.com/ads/custom-audiences#include-or-exclude-audiences-in-a-campaign)
for audience inclusion, exclusion, and eligibility requirements.

## Campaign creation

Pass your criteria in the `targeting` object when you create or update a campaign.
Each guide explains its fields, defaults, and examples. For the campaign request
and response fields, see [Campaigns](https://developers.openai.com/ads/api-reference/campaigns).