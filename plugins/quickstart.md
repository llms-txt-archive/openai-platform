# Quickstart

Plugins extend and customize ChatGPT and Codex. They can add capabilities,
connect to external services, or both. A plugin can include skills that provide
instructions and resources, an MCP server that exposes tools, or both.

ChatGPT and Codex share one universal plugin directory. Public plugins are
published once and become discoverable from supported surfaces in both
products.

This tutorial creates a small skills-only plugin. It is the fastest way to
learn the plugin package structure before you add an MCP server or custom
UI.

By the end, you will have a plugin that teaches ChatGPT and Codex how to turn
meeting notes into a consistent follow-up.

## Create the plugin

The fastest way to start is to ask the built-in plugin creator:

```text
@plugin-creator Create a plugin named meeting-follow-up.
Add a skill that turns meeting notes into a concise recap with decisions,
action items, owners, and a customer follow-up draft.
```

In Codex, invoke the same skill as `$plugin-creator`.

The generated plugin should have this structure:

## Review the manifest

The plugin manifest is stored at `.codex-plugin/plugin.json`. For this
skills-only plugin, it points to the bundled skills directory:

```json
{
  "name": "meeting-follow-up",
  "version": "1.0.0",
  "description": "Turn meeting notes into consistent follow-ups",
  "skills": "./skills/"
}
```

Use a stable, kebab-case `name`. Keep the description focused on the outcome
the plugin helps someone achieve.

## Review the skill

Each skill lives in its own directory and includes a `SKILL.md` file. A minimal
version looks like this:

```md
---
name: meeting-follow-up
description: Turn meeting notes into a recap, action register, and customer follow-up draft.
---

Use this skill when the user provides meeting notes or a transcript and asks
for a follow-up.

1. Summarize the outcome in two or three sentences.
2. List decisions that were made.
3. List action items with an owner when the source identifies one.
4. Draft a concise customer follow-up email.

Do not invent owners, deadlines, decisions, or commitments that are missing
from the source.
```

Adjust the instructions to match the workflow you want to make repeatable.
Keep inputs, required sections, and boundaries explicit.

## Test the plugin

Ask the plugin creator to add the plugin to a personal marketplace for local
testing. Restart the ChatGPT desktop app, open the Plugins Directory, select
your local source, and install the plugin. This personal marketplace is a local
testing source; it does not publish the plugin to the universal directory.

Start a new chat with the plugin enabled and provide representative meeting
notes:

```text
Use the meeting follow-up skill on these notes. Keep unknown owners and dates
unassigned.

We agreed to move the launch to the first week of August. Priya will update the
rollout checklist. The customer wants a revised security summary before the
next review.
```

Verify that the response includes the decision, assigns only the known action
owner, and does not invent an owner or date for the security summary.

Test several realistic inputs, including incomplete notes and requests that
  should not use the skill. Refine the skill description and instructions when
  the plugin activates at the wrong time or produces inconsistent results.

## Add an MCP server when you need tools

This plugin does not need an MCP server because its workflow can use
information already provided to the model. Add an MCP server when the plugin needs to
connect to a service, authenticate users, expose controlled tools, or run code
on infrastructure you operate.

Continue with the
[MCP server and UI quickstart](https://developers.openai.com/plugins/build/app-quickstart), or read
[Plugin architecture](https://developers.openai.com/plugins/concepts/plugins) to compare plugin shapes.
Custom UI remains optional even when a plugin contains an MCP server.

## Publish the plugin

When the plugin is ready for other people, review the complete [plugin build
guide](https://developers.openai.com/plugins/build/plugins). To publish it publicly, use the [plugin
submission portal](https://developers.openai.com/plugins/deploy/submission).