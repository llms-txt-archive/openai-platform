# Agents SDK

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

## Important notice

The Agents SDK is **feature complete**. Maintenance, security fixes, critical bug fixes, and compatibility work continue, but major new features are not planned. For new agent applications, we recommend the **[Agents API](https://developers.openai.com/api/docs/guides/agents-api/quickstart)**, which runs a managed Codex harness.

You can continue using the Agents SDK for existing applications. For new applications that require capabilities the Agents API does not yet support, the SDK remains a short-term option.

## Overview

The OpenAI Agents SDK is an open-source framework for building agent workflows in application code. It builds on [Swarm](https://github.com/openai/swarm), released in 2024 to explore lightweight multi-agent orchestration, and brought those ideas into a framework for production applications with guardrails and built-in tracing. The [Python SDK](https://github.com/openai/openai-agents-python) launched in March 2025, followed by the [TypeScript SDK](https://github.com/openai/openai-agents-js) in June 2025.

Its agent loop runs in your application, coordinating calls to OpenAI and other models through the Responses and Chat Completions APIs with tool execution, including MCP server tools. Session management preserves conversation context across runs. Human approvals let your application pause tool execution for review.

For multi-agent workflows, agents can call other agents as tools or transfer control through handoffs. The SDK also supports [realtime voice agents](https://developers.openai.com/api/docs/guides/voice-agents) for low-latency spoken interactions and [sandbox agents](https://developers.openai.com/api/docs/guides/agents/sandboxes) for working with files and commands. These capabilities let you combine text, voice, and sandbox agents within a broader application workflow.

## Get your first agent running

Start with the [Agents SDK quickstart](https://developers.openai.com/api/docs/guides/agents/quickstart) to install the SDK, define one agent, and run it. Once that works, return here to choose the next capability your application needs.

## Get the Agents SDK

Use the GitHub repositories for more examples, issues, and language-specific reference details.



  [TypeScript SDK



        Open the TypeScript SDK repository on GitHub.](https://github.com/openai/openai-agents-js)
  [Python SDK



        Open the Python SDK repository on GitHub.](https://github.com/openai/openai-agents-python)



## Choose your starting point

| If you want to                            | Start here                                                                                                                                             | Why                                                                                            |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| Set up an Agents SDK integration          | [Quickstart](https://developers.openai.com/api/docs/guides/agents/quickstart)                                                                                                       | This is the shortest path to a working SDK integration.                                        |
| Define one specialist cleanly             | [Agent definitions](https://developers.openai.com/api/docs/guides/agents/define-agents)                                                                                             | Start here when you are still shaping the contract for a single agent.                         |
| Choose models, defaults, and transport    | [Models and providers](https://developers.openai.com/api/docs/guides/agents/models)                                                                                                 | Use this when model choice, provider setup, or transport strategy affects the workflow.        |
| Understand the runtime loop and state     | [Running agents](https://developers.openai.com/api/docs/guides/agents/running-agents)                                                                                               | This is where the agent loop, streaming, and continuation strategies live.                     |
| Run work in a container-based environment | [Sandbox agents](https://developers.openai.com/api/docs/guides/agents/sandboxes)                                                                                                    | Use this when the agent needs files, commands, packages, snapshots, mounts, or provider links. |
| Design specialist ownership               | [Orchestration and handoffs](https://developers.openai.com/api/docs/guides/agents/orchestration)                                                                                    | Use this when you need more than one agent and must decide who owns the reply.                 |
| Add validation or human review            | [Guardrails and human review](https://developers.openai.com/api/docs/guides/agents/guardrails-approvals)                                                                            | Use this when the workflow should block or pause before risky work continues.                  |
| Understand what a run returns             | [Results and state](https://developers.openai.com/api/docs/guides/agents/results)                                                                                                   | This page explains final output, resumable state, and next-turn surfaces.                      |
| Add hosted tools, function tools, or MCP  | [Using tools](https://developers.openai.com/api/docs/guides/tools#usage-in-the-agents-sdk) and [Integrations and observability](https://developers.openai.com/api/docs/guides/agents/integrations-observability) | Tool semantics live in the platform tools docs; SDK-specific MCP and tracing live here.        |
| Inspect and improve runs                  | [Integrations and observability](https://developers.openai.com/api/docs/guides/agents/integrations-observability) and [evaluate agent workflows](https://developers.openai.com/api/docs/guides/agent-evals)      | Use traces for debugging first, then move into evaluation loops.                               |
| Build a voice-first workflow              | [Voice agents](https://developers.openai.com/api/docs/guides/voice-agents)                                                                                                          | Use the SDK voice pipeline and realtime agent patterns.                                        |

## Build with the SDK

In an Agents SDK application, your server manages deployment, tool implementations, state storage, and approval decisions. The SDK runs the agent loop and invokes tools. These guides cover:

- typed application code in TypeScript or Python
- direct control over tools, MCP servers, and runtime behavior
- custom storage or server-managed conversation strategies
- tight integration with existing product logic or infrastructure

A typical SDK reading order is:

- Start with [Quickstart](https://developers.openai.com/api/docs/guides/agents/quickstart) to get one working run on screen.
- Use [Agent definitions](https://developers.openai.com/api/docs/guides/agents/define-agents) and [Models and providers](https://developers.openai.com/api/docs/guides/agents/models) to shape one specialist cleanly.
- Continue to [Running agents](https://developers.openai.com/api/docs/guides/agents/running-agents), [Orchestration and handoffs](https://developers.openai.com/api/docs/guides/agents/orchestration), and [Guardrails and human review](https://developers.openai.com/api/docs/guides/agents/guardrails-approvals) as the workflow grows more complex.
- Use [Results and state](https://developers.openai.com/api/docs/guides/agents/results) and [Integrations and observability](https://developers.openai.com/api/docs/guides/agents/integrations-observability) when application logic depends on the run object or deeper visibility into behavior.

<a id="compare-agent-runtimes"></a>

## Compare agent runtime options

Use the [Agents overview](https://developers.openai.com/api/docs/guides/agents#compare-agent-runtimes) to compare the Agents API, Codex SDK, and Responses API. The Agents SDK runs in your application; the Agents API runs a managed harness in OpenAI's service.