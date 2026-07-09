# Connect from ChatGPT

## Before you begin

You can test your app in ChatGPT with your account using [developer mode](https://platform.openai.com/docs/guides/developer-mode).

Apps are now submitted and published as plugins. Use developer mode to test
  your app in ChatGPT; when you are ready for public distribution, submit the
  plugin through the plugin submission portal. See
  <a href="/codex/submit-plugins">Submit plugins</a>.

To turn on developer mode, open [ChatGPT](https://chatgpt.com), then go to
**Settings → Security and login** and turn on **Developer mode**.

From there, you can toggle developer mode if your organization allows it. If the
toggle is unavailable, ask your workspace admin to allow developer mode for your
account or workspace.

Once developer mode is active, you can create and manage developer-mode apps
from **Settings → Plugins** or directly at
[chatgpt.com/plugins](https://chatgpt.com/plugins).

As of November 13th, 2025, ChatGPT Apps are supported on all plans, including
  Business, Enterprise, and Education plans.

## Create a developer-mode app

Once you have developer mode enabled, create a developer-mode app in ChatGPT.

1. Ensure your MCP server is reachable over HTTPS (for local development, use [Secure MCP Tunnel](https://developers.openai.com/api/docs/guides/secure-mcp-tunnels) and select **Tunnel** when you create the app, or you can expose a local server to the public internet via a tool such as [ngrok](https://ngrok.com/) or [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)).
2. In ChatGPT, open **Settings → Plugins** or go directly to [chatgpt.com/plugins](https://chatgpt.com/plugins).
3. Select the plus button to create a new developer-mode app.
4. Provide the app metadata:
   - **Name** – a user-facing title such as _Kanban board_.
   - **Description** – explain what the app does and when to use it. The model uses this text during discovery.
   - **MCP server URL** – the public `/mcp` endpoint of your server (for example `https://abc123.ngrok.app/mcp`).
5. Click **Create**. If the connection succeeds you will see a list of the tools your server advertises. If it fails, refer to the [Testing](https://developers.openai.com/apps-sdk/deploy/testing) guide to debug your app with MCP Inspector or the API Playground.

## Try the app

Once your developer-mode app is created, you can try it out in a new ChatGPT conversation.

1. Open a new chat in ChatGPT.
2. Click the **+** button near the message composer, and click **More**.
3. Choose your app in the list of available tools. This will add your app to the conversation context for the model to use.
4. Prompt the model to invoke tools by saying related to your app. For example, “What are my available tasks?” for a Kanban board app.

ChatGPT uses each app's tool annotations and the user's app permission settings to
decide when to request confirmation. Users can choose one of three permission
levels:

- **Always ask**: Ask before retrieving information or making changes.
- **Ask before making changes**: Retrieve information automatically, but ask
  before making changes.
- **Ask only before important changes** (default): Retrieve information and make
  routine changes automatically, but ask before consequential changes such as
  sending, deleting, posting, or purchasing.

For personal accounts, users can set a global permission level or override it for
an individual app in **Settings > Apps**. Eligible confirmation prompts also
offer **Always allow**, which remembers the user's permission for that app.

For Business and Enterprise workspaces, admins control the workspace default and
per-app overrides. Members don't see **Always allow**. Admins can update the
workspace default in **Workspace settings > Permissions & roles > Connected
data**, or configure an individual app from **Admin Apps > App permissions**.

These settings apply to connected apps in ChatGPT. Apps authenticated in Codex
use separate permission controls.

## Refreshing metadata

Whenever you change your tools list or descriptions, refresh your MCP server's metadata in ChatGPT.

1. Update your MCP server and redeploy it (unless you are using a local server).
2. Open **Settings → Plugins** or [chatgpt.com/plugins](https://chatgpt.com/plugins), click into your developer-mode app, and choose **Refresh**.
3. Verify the tool list updates and try a few prompts to test the updated flows.

This refresh flow applies to developer-mode apps. Plugins that contain apps use reviewed [app metadata snapshots](https://developers.openai.com/apps-sdk/deploy/submission#how-published-app-metadata-versions-work). To update the app metadata in a published plugin, scan, submit, and publish a new version.

## Using other clients

You can connect to your MCP server on other clients.

- **API Playground** – visit the [platform playground](https://platform.openai.com/chat), and add your MCP server to the conversation: open **Tools → Add → MCP Server**, and paste the same HTTPS endpoint. This is useful when you want raw request/response logs.
- **Mobile clients** – once the app is linked on ChatGPT web, it will be available on ChatGPT mobile apps as well. Test mobile layouts early if your component has custom controls.

With the app linked you can move on to validation, experiments, and eventual rollout.