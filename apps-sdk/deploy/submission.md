# Prepare and maintain an app for plugin submission

Prepare an MCP-backed app for public review as part of a plugin.

Apps are now submitted and published as plugins. If your plugin contains an
  app, build the app's MCP server with the Apps SDK, then submit the plugin
  through the plugin submission portal. See
  <a href="/codex/submit-plugins">Submit plugins</a> for the source-of-truth
  submission flow and <a href="/codex/build-app">Build an app</a> for how apps
  fit into plugins.

## Prepare an app for plugin submission

Use this page for the app-specific requirements that apply when a plugin
contains an MCP-backed app: organization verification, app management
permissions, MCP server requirements, review snapshots, and version
maintenance.

When your app works in [Developer Mode](https://developers.openai.com/apps-sdk/deploy/testing), submit the
plugin that contains it for review in the
[plugin submission portal](https://platform.openai.com/plugins). This page
covers the app and MCP server requirements for that submission.

Only submit the plugin if you intend for it to be publicly available in the
countries you define during submission. For apps you intend to use privately or
only within your workspace, use
[developer mode](https://platform.openai.com/docs/guides/developer-mode)
instead.

Before submitting the plugin, review the [app guidelines](https://developers.openai.com/apps-sdk/app-guidelines)
for MCP server and app experience expectations, and see
[Submit plugins](https://developers.openai.com/codex/submit-plugins) for the full plugin submission,
approval, and publishing flow.

For the complete flow, including skills-only plugins, plugins that contain apps,
review, approval, and publishing, see [Submit plugins](https://developers.openai.com/codex/submit-plugins).

## Before you submit the plugin

### Organization verification

Before submitting a plugin that contains an app, complete identity verification
in the [OpenAI Platform Dashboard](https://platform.openai.com/settings/organization/general)
for the name you plan to publish under in the directory.

- **If you want to publish under your own name**, complete **individual verification**.
- **If you want to publish under a business name**, complete **business verification**.

This is enforced during review. Publishing under an unverified individual or
business name will result in rejection.

### App management permissions

To create plugin drafts that contain apps and submit them for review, you need
the `api.apps.write` permission. To view drafts and review status in the
Dashboard, you need the `api.apps.read` permission. Organization owners
automatically have both permissions, and can grant them to non-owners through
roles in the [OpenAI Platform Dashboard](https://platform.openai.com/settings/organization/roles).

### MCP server requirements

- Your MCP server is hosted on a publicly accessible domain
- You are not using a local or testing endpoint
- You defined a [content security policy (CSP)](https://developers.openai.com/apps-sdk/build/mcp-server#content-security-policy-csp) to allow the exact domains the app fetches from (this is required to submit a plugin that contains an app for security reasons)

### Template MCP server URLs

Most apps should submit a universal MCP server URL: a single hosted MCP endpoint that works for all users and organizations.

Choose **Template** only if your app uses workspace-specific MCP server URLs, such as when each customer has a separate tenant, workspace, or managed MCP endpoint. Template submissions require two URL values:

- **MCP Server URL:** A concrete, working MCP endpoint for review and automated checks.
- **Template MCP Server URL:** The URL pattern that describes which part of the MCP endpoint changes across customer workspaces.

The review MCP server URL must be a real endpoint that OpenAI can connect to during submission review. Don't enter a placeholder URL in the **MCP Server URL** field.

Use placeholders in the **Template MCP Server URL** for the parts that a workspace admin will configure later. Placeholders must use `{name}` syntax, start with a letter, and contain only letters, numbers, or underscores. Each placeholder name must be unique.

Make sure the concrete **MCP Server URL** matches the template pattern after replacing each placeholder with a real value.

For example:

```text
https://{workspace}.example.com/mcp
https://mcp.example.com/{tenant}/mcp
```

## Submit for review

If the prerequisites are met, you can submit the plugin that contains your app
for review from the [plugin submission portal](https://platform.openai.com/plugins).

### Start the review process

In the plugin submission portal:

1. Add your MCP server details (as well as OAuth credentials if OAuth is selected), and then select **Scan Tools**.
2. Complete the required fields in the submission form and check all confirmation boxes. You will need to provide your app name, logo, description, company and privacy policy URLs, MCP and tool information, test prompts and responses, and localization information. If your app has a UI, you may also provide optional screenshots. Don't provide screenshots for apps without a UI.
3. Select **Submit for review**.

### Metadata stored during tool scanning

When you select **Scan Tools**, the dashboard imports metadata advertised by your MCP endpoint into the draft. This includes tool names, titles, and descriptions; input and output schemas; security schemes; `_meta` fields; [tool annotations](https://developers.openai.com/apps-sdk/reference#annotations); linked UI resource metadata, including CSP settings; and MCP server `instructions`. The dashboard displays the annotation values provided by your server.

Your submission justifications should explain why those server-provided annotation values match each tool's behavior. They don't override the annotations. For example, if your server advertises `readOnlyHint: false`, describing the tool as “functionally read-only” in the justification doesn't make the tool read-only. If the tool is truly read-only, update its server annotation to `readOnlyHint: true`, deploy the change, select **Scan Tools** again, verify the updated value, and then submit.

Each organization can publish multiple unique plugins that contain apps. For
each MCP-backed app, only one version may be published at a time and only one
version may be in review at a time. If you need to make changes after
submitting, withdraw that submission by selecting **Cancel Review** and
resubmit the same version draft.

_Note that for now, projects with EU data residency cannot submit plugins that
contain apps for review. Please use a project with global data residency. If you
don't have one, you can create a new project in your current organization from
the OpenAI Dashboard._

## Review and approval

Once submitted, the plugin will enter the review queue. You can review the
status within the Dashboard and will receive an email notification informing
you of any status changes.

### Reviews and checks

We may perform automated scans or manual reviews to understand how your app works and whether it may conflict with our policies.

### Approval, rejection, and appeals

If your plugin is approved, we will notify you by email. Once approved, you can publish it from the plugin submission portal.

If your plugin is rejected or removed because of the app or MCP server, you will receive feedback on which checks were unsuccessful. After making the necessary changes, you may resubmit the plugin for re-review. Alternatively, if you wish to appeal the decision, you can respond back to the email you received. Make sure to include a clear rationale for the appeal along with any new information that will assist us in our review.

### Getting help

If you have questions before, during, or after submission, and if your question is not answered in the documentation, contact OpenAI support for further assistance. Include the app ID shown in the plugin submission portal so the support team can identify your app.

### Review and approval FAQs

**How long does review take?**

Review timelines may vary as we continue to build and scale our processes. Please do not contact support to request expedited review, as these requests cannot be accommodated.

**What are common rejection reasons and how can I resolve them?**

- **We're unable to connect to your MCP server using the MCP URL and/or test credentials we were given.**
  - For servers requiring authentication, our review team must be able to log into a demo account with no further configuration required.
  - Ensure that the provided URL and credentials are correct, do not feature MFA (including requiring SMS codes, login through systems that require SMS, email or other verification schemes).
  - Ensure that the provided credentials can be used to log in successfully (test them outside any company networks or LANs, or other internal networks).
  - Confirm that the credentials have not expired.
- **One or more of your test cases did not produce correct results.**
  - Review all test cases carefully and rerun each one. Ensure that outputs match the expected results. Verify that there are no errors in the UI (if applicable) - for example, issues with loading content, images, or other UI issues.
  - Ensure that the returned textual output closely adheres to the user's request, and does not offer extraneous information that is irrelevant to the request, including personal identifiers.
  - Ensure that all test cases pass on both ChatGPT web and mobile apps.
  - Compare actual outputs to clearly defined expected behavior for each tool and fix any mismatch so results are relevant to the user's input and the app “reliably does what it promises”.
  - If required, in your resubmission, modify your test cases and expected responses to be clear and unambiguous.
- **Your app returns user-related data types that are not disclosed in your privacy policy.**
  - Audit your MCP tool responses in developer mode by running a few realistic example requests and listing every user-related field your app returns (including nested fields and “debug” payloads). Ensure tools return only what's strictly necessary for the user's request and remove any unnecessary PII, telemetry/internal identifiers (e.g., session/trace/request IDs, timestamps, internal account IDs, logs) and/or any auth secrets (tokens/keys/passwords).
  - You may also consider updating your published privacy policy so it clearly discloses all categories of personal data you collect/process/return and why—if a field isn't truly needed, remove it rather than disclose it.
  - If a user identifier is truly necessary, make it explicitly requested and clearly tied to the user's intent (not “looked up and echoed” by default)
- **Tool hint annotations do not appear to match the tool's behavior:**
  - **readOnlyHint:** Set to `true` if it strictly fetches/looks up/lists/retrieves data and does not modify anything. Set to `false` if the tool can create/update/delete anything, trigger actions (send emails/messages, run jobs, enqueue tasks, write logs, start workflows), or otherwise change state.
  - **destructiveHint:** Set to `true` if it can cause irreversible outcomes (deleting, overwriting, sending messages/transactions you can't undo, revoking access, destructive admin actions, etc.), even in only select modes, via default parameters, or through indirect side effects. Ensure the justification provided clearly describes what is irreversible and under what conditions, including any safeguards like confirmation steps, dry-run options, or scoping constraints. Otherwise, set to `false`.
  - **openWorldHint:** Set to `true` if it can write to or change publicly visible internet state (e.g., posting to social media/blogs/forums, sending emails/SMS/messages to external recipients, creating public tickets/issues, publishing pages, pushing code/content to public endpoints, submitting forms to third parties, or otherwise affecting systems outside a private/first-party context). Set to `false` only if it operates entirely within closed/private systems (including internal writes) and cannot change the state of the publicly visible internet.

## Publication and distribution

### Publish the plugin

Once the plugin is approved, you can publish it from the [plugin submission portal](https://platform.openai.com/plugins) by selecting **Publish**.

### Discoverability

Once published, users can find your plugin by:

- Clicking a direct link to the plugin listing in the directory.
- Searching for the plugin by name.

Plugins that demonstrate strong real-world utility and high user satisfaction may be eligible for enhanced distribution opportunities—such as directory placement or proactive suggestions—but few plugins will receive enhanced distribution at publication. There is no process by which to request this at this time.

### Publication and Distribution FAQs

**What happens after the plugin is approved? Will it be listed in the plugin directory automatically?**

After the plugin is approved, you can choose to publish it from the [plugin submission portal](https://platform.openai.com/plugins). You must publish before it can appear in the universal plugin directory.

**Why can't I see my plugin in the directory?**

Plugins appear on the directory's main pages only if OpenAI selects them for enhanced distribution. To confirm that your plugin is published, search for it using the exact publication name or open its directory URL from the plugin submission portal.

**What should I do if I want to issue a press release or public announcement about my plugin?**

Before issuing any press releases or public announcements regarding the launch
of your plugin, please first reach out to
[press@openai.com](mailto:press@openai.com) to coordinate with our
communications team.

## Ongoing Maintenance

### How published app metadata versions work

Treat the metadata exposed by your MCP server as a versioned API contract for
the app inside the plugin. When you scan the app's MCP endpoint in the plugin
submission portal, OpenAI stores the discovered metadata with that draft
version. Submitting the version sends that stored snapshot for review. The
published app uses this metadata snapshot while tool calls and UI resources
continue to use your live MCP server.

Use this table to determine how to ship each change:

| Change                                                                                                                                                                                                   | Required action                                                                                                                                                              | When users see the change                                                                                   |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Tool list, names, titles, descriptions, input or output schemas, annotations, tool security schemes, tool `_meta` fields (including UI resource references and visibility), or MCP server `instructions` | Deploy the change, create or update a draft version, scan the endpoint, submit the version for review, and publish it after approval.                                        | After you publish the approved version. Until then, users continue to use the currently published snapshot. |
| UI resource URI or linked resource metadata, including content security policy (CSP) settings                                                                                                            | Deploy the change, create or update a draft version, scan the endpoint, submit the version for review, and publish it after approval.                                        | After you publish the approved version.                                                                     |
| Backward-compatible content update served from the same published UI resource URI                                                                                                                        | Deploy the content update. You don't need to scan, submit, or publish a new version if the URI and published contract remain compatible.                                     | After deployment. ChatGPT may continue serving cached resource contents for up to one hour.                 |
| Server-only fix or change to live tool results, including result `_meta`, or business data                                                                                                               | Deploy the server change. You don't need to scan, submit, or publish a new version if the change preserves the published contract.                                           | Through your live endpoint after deployment.                                                                |
| MCP server origin (`scheme`, `hostname`, or `port`)                                                                                                                                                      | To change the origin, create a new app, then complete its scan, submission, review, and publication flow. To change only the endpoint path, use the normal new-version flow. | After you publish the new app or approved version.                                                          |

Breaking changes to the app inside a published plugin aren't currently
supported. Removing or renaming a tool, making a schema incompatible, or
serving incompatible content at or removing content from a published UI
resource URI can break the current version as soon as the server change
deploys. Make backward-compatible updates instead:

1. Add new tools, fields, or UI resources while continuing to honor the published contracts.
2. Submit the updated metadata as a new version.
3. Publish the approved version and keep the old contracts available.

You can deploy server-only fixes without submitting a new version if they preserve the published contract. If a deployment breaks the published version, roll back the server change rather than waiting for a new version to complete review.

### Submitting new versions for review

Once your app is published, its submitted information and reviewed metadata snapshot are locked for safety. To update either, create a new draft version of your existing app and resubmit that version for review (do not create a new app). Each resubmission starts a new review. When submitting changes, include a clear description of what changed in the release notes section of the form.

The app's MCP server origin (`scheme`, `hostname`, or `port`) can't change
between versions. To use a different origin, submit a new plugin that contains
the app with the new MCP server origin. You can change the endpoint path in a
new version of the existing app.

We will review the updated app metadata again and inform you if the update was approved or rejected via email and in the [plugin submission portal](https://platform.openai.com/plugins). Similar to initial reviews, if rejected, you may update and resubmit or appeal the decision.

Once your resubmission is approved, you can publish the update, which will
replace the previous app version inside the plugin.

If you've made additional changes to your app between submission and approval and want to submit a new version for review, you can cancel the review from the plugin submission portal and resubmit.

### Changing published app metadata versions and removing the plugin

Once an app is published as part of a plugin, you can change the published
version from the [plugin submission portal](https://platform.openai.com/plugins)
by unpublishing the current version and publishing an approved replacement. You
can remove the plugin from public visibility by unpublishing the current version
and not publishing an alternative version.

To remove the plugin from your organization and ChatGPT entirely, delete it from
the plugin submission portal.

### Maintenance requirements

Plugins, including the apps they contain, may be removed if they are inactive,
unstable, or non-compliant. We may reject or remove any plugin from our services
at any time and for any reason without notice, such as for legal or security
concerns or policy violations.

### Ongoing Maintenance FAQs

**What happens if users report my plugin as harmful or misleading?**

OpenAI reviews user reports and may review or investigate your plugin, including
any app it contains. Plugins that are identified as violating our policies may
be restricted or removed. You may appeal a removal or other enforcement action
by following the appeals process described here. You should regularly review and
respond to feedback and update your plugin if issues are found.

**How long will updates take?**

Similar to new reviews, we are unable to offer estimated times for update
reviews.