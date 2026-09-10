# Agents API examples

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

Complete applications and sandbox integrations for the Agents API.

## Applications

| Example | What it does |
| --- | --- |
| [Incident response](https://developers.openai.com/cookbook/examples/agents_api/apps/sev_bot/README.md) | Investigate alerts and request approval for recovery actions. |
| [Slack bot](https://developers.openai.com/cookbook/examples/agents_api/apps/slack_bot/README.md) | Answer requests using conversation history and connected workplace tools. |
| [Data analyst](https://developers.openai.com/cookbook/examples/agents_api/apps/data_analyst/README.md) | Answer questions with read-only warehouse queries. |
| [GitHub issue investigator](https://developers.openai.com/cookbook/examples/agents_api/apps/github_issues/README.md) | Reproduce reported bugs and prepare findings for GitHub. |
| [Document reviewer](https://developers.openai.com/cookbook/examples/agents_api/apps/document_review/README.md) | Review invoices and contracts with policy skills and specialist agents. |

## Sandbox integrations

- [Application-managed](https://developers.openai.com/cookbook/examples/agents_api/sandboxes/application_managed/README.md): your application
  starts and stops the sandbox directly.
- [Webhook-managed](https://developers.openai.com/cookbook/examples/agents_api/sandboxes/webhook_managed/README.md): a deployed handler
  provisions the sandbox while a shared client calls the Agents API.

See the [sandbox overview](https://developers.openai.com/cookbook/examples/agents_api/sandboxes/README.md) to choose a provisioning mode.

## Run an example

Run commands from the Cookbook repository root. Each example's README covers
dependencies, credentials, sample inputs, and cleanup. Python entry points declare
their dependencies inline for [uv](https://docs.astral.sh/uv/).

The application examples require Python 3.14. Provider examples declare their
supported Python version in each script.

These examples use the official OpenAI Python SDK's `client.beta.agents`
and `sessions.stream()`, available in `openai>=3.13.0` on PyPI.

Create sessions with the standard client:

```python
from openai import OpenAI

client = OpenAI()

session = client.beta.agents.sessions.create(
    agent={"model": "gpt-5.6-sol"},
    environment={"type": "self_hosted", "workspace_directory": "/workspace"},
)
```

The sandbox examples then start `codex exec-server` with the returned
`session.environment.id` and `session.environment.remote_url`, stream a turn,
and clean up. Async applications use `AsyncOpenAI` with the same
`client.beta.agents` resources and `await` for requests.

Keep credentials in environment variables or the example's local `.env` file.
Use a separate restricted `OPENAI_EXECUTOR_API_KEY` for sandbox execution. Follow
the [executor authentication requirements](https://developers.openai.com/api/docs/guides/agents-api/environments/self-hosted#authentication).
The executor key needs `api.agents.environments.connect`, and its IP restrictions
must allow requests from your sandbox's outbound network.

## Documentation

- [Agents API overview](https://developers.openai.com/api/docs/guides/agents-api/overview)
- [Self-hosted sandboxes](https://developers.openai.com/api/docs/guides/agents-api/environments/self-hosted)
- [Sandbox lifecycle](https://developers.openai.com/api/docs/guides/agents-api/environments/lifecycle)
