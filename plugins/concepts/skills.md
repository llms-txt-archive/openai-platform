# Skills

Skills are folders of instructions and resources that teach ChatGPT and Codex
how to complete repeatable workflows. A plugin can include one skill or a group
of related skills.

Each skill has a `SKILL.md` file with:

- A name.
- A description that tells the model when to consider the skill.
- Instructions for completing the workflow.
- Optional references, scripts, templates, and other assets.

## When to use a skill

Use a skill when the model can complete a use case with instructions, examples,
bundled resources, and tools it already has.

For example, a skill can define how to:

- Turn meeting notes into a recap and follow-up.
- Apply an organization's writing or review standards.
- Generate a report from files the user provides.
- Follow a repeatable debugging or deployment procedure.

Add an [MCP server](https://developers.openai.com/plugins/concepts/mcp-server) when the workflow also needs
live data, authentication, controlled actions, or code that runs on
infrastructure you operate.

## How skills activate

The model first sees skill metadata, including the name and description. It
loads the complete instructions when the user's request matches the skill or
the user invokes it directly.

Write descriptions around the user goal and the conditions that should trigger
the workflow. Keep detailed steps and output requirements in the instruction
body.

## Skills in a plugin

Skills are the workflow layer of a plugin. They can:

- Work on their own.
- Guide the model through tools exposed by the plugin's MCP server.
- Package organization-specific procedures with reusable templates and
  references.

Skills and MCP tools should have clear, complementary roles. A skill explains
how to complete the workflow; an MCP server provides live information and
enforces controlled actions.

Continue with [Build skills](https://developers.openai.com/plugins/build/skills) to create, test, and
package a skill.