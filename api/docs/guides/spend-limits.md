# Spend limits

Spend limits help you control API costs by setting monthly budgets for your organization or for individual projects. You can use them as a soft budget for monitoring spend, or enforce a hard limit so API responses fail after spend reaches the limit.

## How spend limits work

Spend limits are monthly limits measured in U.S. dollars. They can apply at two levels:

- **Organization spend limits** apply across the organization.
- **Project spend limits** apply to a single project.

You can also enforce either level as a hard limit, which makes API responses fail after spend reaches the monthly limit.

## Using hard limits

A hard limit causes API responses to fail after the organization or project reaches its monthly spend limit. After spend reaches a hard limit, API responses return a `429` error until you increase the spend limit or the limit resets.

Use hard limits when you need to prevent unexpected spend, such as for experiments, development projects, or customer-specific projects with fixed budgets.

## Configuring spend limits



<div data-content-switcher-pane data-value="organization">
    <div class="hidden">Organization</div>

1. Go to [Organization limits](https://platform.openai.com/settings/organization/limits).
2. In **Spend**, select **Edit spend limit**.
3. Enter the **Monthly spend limit**.
4. To make API responses fail after the organization reaches the limit, turn on **Enforce a hard limit**.
5. Select **Save**.

  </div>
  <div data-content-switcher-pane data-value="project" hidden>
    <div class="hidden">Project</div>

1. Go to [Project settings](https://platform.openai.com/settings/).
2. Select **Limits**.
3. In **Spend**, select **Edit spend limit**.
4. Enter the **Monthly spend limit**.
5. To make API responses fail after the project reaches the limit, turn on **Enforce a hard limit**.
6. Select **Save**.

  </div>



## Spend alerts

Use spend alerts to get notified before spend reaches the monthly limit. Add alerts at thresholds that give your team enough time to adjust usage, increase the limit, or investigate unexpected traffic.

For broader usage analysis, review the [usage dashboard](https://platform.openai.com/settings/organization/usage). For request-rate constraints, see the [rate limits guide](https://developers.openai.com/api/docs/guides/rate-limits).