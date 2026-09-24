# Agents

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

For new agent applications, start with the **[Agents API](https://developers.openai.com/api/docs/guides/agents-api/overview)**. OpenAI runs the Codex harness and manages orchestration, context compaction, and durable sessions. You build the surrounding application, connect tools, and choose where execution happens.

Follow the [Agents API quickstart](https://developers.openai.com/api/docs/guides/agents-api/quickstart) to run a task, stream progress, and continue a session.

## Choose your starting point

| You want to | Start here |
| --- | --- |
| Build a new agent application with a managed runtime | [Agents API quickstart](https://developers.openai.com/api/docs/guides/agents-api/quickstart) |
| Run the Codex harness in infrastructure you operate | [Codex SDK](https://developers.openai.com/codex/codex-sdk) |
| Call models directly or own the agent loop | [Responses API](https://developers.openai.com/api/docs/guides/migrate-to-responses) |

<a id="agents-sdk-vs-responses-api"></a>

<a id="compare-agent-runtimes"></a>

## Compare agent runtime options

Choose based on what OpenAI manages and what your application needs to control.

| Option | What it manages | What you operate |
| --- | --- | --- |
| **Agents API** | Hosted Codex harness, orchestration, and durable session state | Your application, tool integrations, and choice of execution environment |
| **Codex SDK** | Codex harness running in your environment | The harness process, hosting, and application lifecycle |
| **Responses API** | Model responses and configured hosted capabilities | Application logic and any agent loop you build around the API |

Use the [Agents API architecture guide](https://developers.openai.com/api/docs/guides/agents-api/architecture) to understand the boundary between the hosted harness and your execution environment. For direct model integrations, the Responses API also offers hosted tools and state through response chaining or Conversations; follow its guides for the capabilities you use.




## Add tools, skills, and prompt caching

Tool design, reusable skills, and prompt caching apply across agent workflows. Their configuration and lifecycle can differ by API.

- Start with [Using tools](https://developers.openai.com/api/docs/guides/tools) for function calling, MCP, and hosted capabilities.
- Read [Programmatic Tool Calling](https://developers.openai.com/api/docs/guides/tools-programmatic-tool-calling) for orchestration with JavaScript and the configuration for each API.
- Use [Skills](https://developers.openai.com/api/docs/guides/tools-skills) for reusable instructions and the supported loading mechanisms.
- Read [Prompt caching](https://developers.openai.com/api/docs/guides/prompt-caching) for shared caching behavior, then [Agents API observability and usage](https://developers.openai.com/api/docs/guides/agents-api/observability) for session accounting.

An Agents API session, an SDK session, a Responses conversation, and a sandbox are different resources. Follow the state and cleanup instructions for the runtime you choose.

## If you use the Agents SDK

The [Agents SDK](https://developers.openai.com/api/docs/guides/agents/sdk) is **feature complete**: major new features are not planned, but maintenance, security fixes, critical bug fixes, and compatibility work continue. You can continue using it for existing applications. See also: [SDK support policy](https://developers.openai.com/api/docs/guides/agents/sdk#important-notice).

For new agent applications, start with the [Agents API](https://developers.openai.com/api/docs/guides/agents-api/quickstart).