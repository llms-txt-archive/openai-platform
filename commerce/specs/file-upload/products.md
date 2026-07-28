# Products

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

<h2 id="feed-reference">Feed Reference</h2>
      

        This reference defines the shared flat-file schema that OpenAI ingests
        and indexes. For a non-Ads product feed, follow the Required and
        Optional labels on this page. Ads product feeds use the same base schema
        plus the additional eligibility requirement in the 
        [Ads product feeds guide](https://developers.openai.com/ads/product-feeds#use-the-correct-feed-schema)
        {"."}
      

      

        Each table below groups fields by schema object and indicates whether a
        field is Required or Optional for a non-Ads feed, along with validation
        rules to help your engineering team build and maintain a compliant
        upload file.
      

      

        Supplying all required fields ensures your products can be displayed
        correctly. For Ads processing, also set 
        `is_ads_eligible` to `true` on each product that
        Ads should process.