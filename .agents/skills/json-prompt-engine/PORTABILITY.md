# Json Prompt Engine Portability Notes

## Scope

This note documents the thin adapter layout for `json-prompt-engine`.
It does not redefine the skill workflow. The source workflow remains in `SKILL.md` and the schema, registry, and delivery rules remain in `references/`.

## Maintenance Source

Treat `.agents/skills/json-prompt-engine/` as the primary maintained package for this repository.
The Claude Code package under `.claude/skills/json-prompt-engine/` is an adapter mirror used for Claude Code discovery.

When the core workflow changes, update the primary package first, then mirror the relevant `SKILL.md` and `references/` files into the Claude Code adapter.
Do not copy `agents/openai.yaml` into the Claude Code adapter.

## Codex / OpenAI

The Codex package lives at:

```text
.agents/skills/json-prompt-engine/
```

`agents/openai.yaml` is a Codex / OpenAI companion metadata file.
It may describe display text, invocation policy, or tool dependencies for Codex / OpenAI hosts.
Do not assume Claude Code or Cursor will read this file.

Manual invocation may use the host-supported explicit skill mention syntax, such as `$json-prompt-engine` in Codex contexts that support it.
Implicit invocation depends on the host skill discovery mechanism and the `description` metadata.

## Claude Code

The Claude Code adapter lives at:

```text
.claude/skills/json-prompt-engine/
```

It contains its own `SKILL.md` and mirrored `references/` files so relative links remain valid inside the Claude Code skill package.
The adapter does not depend on `agents/openai.yaml`.

The Claude Code `SKILL.md` keeps the same skill name and core workflow, with a Claude-oriented `when_to_use` field to clarify relevant trigger situations.
This improves discoverability language, but it does not guarantee automatic invocation.
Automatic use still depends on Claude Code's own skill selection behavior and the user's request.

## Cursor

Cursor can use the existing package at:

```text
.agents/skills/json-prompt-engine/
```

No separate Cursor adapter is required for the current instruction-only skill package.
The package uses `SKILL.md` plus `references/`, which matches the supported open skill shape described in the upstream review.

Manual use in Cursor should rely on Cursor's Agent chat skill search or slash invocation behavior.
Automatic invocation depends on Cursor's own agent decision process and the skill metadata.

## Compatibility Boundaries

This repository provides adapter layers for discovery and supporting-file availability.
It does not claim fully equivalent runtime behavior across hosts.

Known non-portable element:

- `agents/openai.yaml` is retained for Codex / OpenAI only.

Known host-dependent behavior:

- automatic invocation
- skill ranking and trigger matching
- context retention after the skill is loaded
- exact handling of long instructions
