# Products

<div data-product-feed-version-container>
    <div data-product-feed-version-fragment>
      <h2 id="feed-reference">Feed Reference</h2>
      <p>
        This reference defines the shared flat-file schema that OpenAI ingests
        and indexes. For a non-Ads product feed, follow the Required and
        Optional labels on this page. Ads product feeds use the same base schema
        plus the additional eligibility requirement in the 
        <a href="/ads/product-feeds#use-the-correct-feed-schema">
          Ads product feeds guide
        </a>
        {"."}
      </p>
      <p>
        Each table below groups fields by schema object and indicates whether a
        field is Required or Optional for a non-Ads feed, along with validation
        rules to help your engineering team build and maintain a compliant
        upload file.
      </p>
      <p>
        Supplying all required fields ensures your products can be displayed
        correctly. For Ads processing, also set 
        <code>is_ads_eligible</code> to <code>true</code> on each product that
        Ads should process.
      </p>
      </div>

  </div>