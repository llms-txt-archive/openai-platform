# Build skills

A skill teaches ChatGPT and Codex how to complete a repeatable workflow. Use a
skill when instructions, examples, templates, or bundled resources are enough
to produce the result. Add an MCP server when the workflow also needs live data
or controlled actions.

A plugin can contain one skill or a group of related skills. Keep every skill
focused on a recognizable user goal from your
[use-case inventory](https://developers.openai.com/plugins/plan/use-case).

## Create a skill

The fastest way to start is with the built-in skill creator:

```text
@skill-creator Create a skill that turns meeting notes into a recap,
an action register, and a customer follow-up draft.
```

In Codex, invoke the same creator as `$skill-creator`.

You can also create the files manually. Each skill lives in its own directory
and requires a `SKILL.md` file:

## Write `SKILL.md`

Start the file with a name and a description, followed by the instructions:

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

The description determines when the model considers the skill. State the
workflow and the conditions that should trigger it. Put detailed procedure,
format, and safety instructions in the body.

## Define the workflow boundary

Connect every skill to one or more use cases. The instructions should make the
following clear:

- What input the workflow expects.
- Which steps the model should follow.
- What output the user should receive.
- Which facts the model must not infer.
- When the workflow should ask a question, stop, or decline.
- Which supporting files the model should consult.

Prefer one focused skill over a large collection of loosely related
instructions. Split workflows when they have different triggers, inputs, or
success criteria.

## Add supporting resources

Keep `SKILL.md` concise and place detailed material next to it:

- Use `references/` for policies, schemas, examples, and background material.
- Use `assets/` for templates or files the workflow should copy or transform.
- Use `scripts/` when the workflow needs deterministic computation or file
  processing.

Reference supporting files from `SKILL.md` and explain when to load or run
them. Do not add a script when instructions and existing tools can complete the
task reliably.

## Connect skills to MCP tools

A skill can guide the model through tools exposed by the plugin's MCP server.
Use the skill for workflow instructions and the server for live data,
authorization, and controlled actions.

If a skill requires an MCP server, declare the dependency in
`agents/openai.yaml`:

```yaml
dependencies:
  tools:
    - type: "mcp"
      value: "acme-projects"
      description: "Read and update Acme projects"
      transport: "streamable_http"
      url: "https://example.com/mcp"
```

A dependency makes the required tool available; it does not replace clear
  workflow instructions. Tell the model which tools to use, in what order, and
  how to handle missing or ambiguous results.

## Test the skill

Test with representative requests from the use-case inventory:

1. Direct requests that should activate the skill.
2. Indirect requests that express the same goal.
3. Incomplete inputs that should trigger a follow-up question.
4. Requests that should not activate the skill.
5. Edge cases where the skill must avoid inventing information or taking an
   unsupported action.

Review both activation and output quality. Refine the description when the
skill activates at the wrong time. Refine the instructions when it chooses the
right workflow but produces an inconsistent result.

## Package the skill

Point the plugin manifest at the skills directory:

```json
{
  "name": "meeting-follow-up",
  "version": "1.0.0",
  "description": "Turn meeting notes into consistent follow-ups",
  "skills": "./skills/"
}
```

See [Package your plugin](https://developers.openai.com/plugins/build/plugins) for the complete manifest,
local testing, and distribution flow. If the workflow needs live data or
actions, continue with [Build an MCP server](https://developers.openai.com/plugins/build/mcp-server).