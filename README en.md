# ez prompt skill

Turn one-off prompts into prompt packages that are reviewable, governable, and reusable.

`ez prompt skill` is a prompt governance engine for AI Agent / Skill workflows. It helps you take complex, easy-to-drift natural-language requests, organize them first into an internal JSON prompt that is reviewable, reusable, and deliverable, and then generate either the final answer or a downstream prompt package.

It is not a standalone CLI, and it is not a collection of prompt templates.

It is closer to a layer of prompt governance workflow: first break down complex requirements clearly and review them clearly, then decide whether to generate a final answer or a prompt package that can be handed off to a downstream model.

<p align="center">
  <strong>Language</strong>:
  <a href="./README.md">简体中文</a> |
  <a href="./README%20en.md">English</a>
</p>

## Quick Navigation

- [Overview](#overview)
- [Why It Is Needed](#why-it-is-needed)
- [Who It Is For](#who-it-is-for)
- [What It Can Produce](#what-it-can-produce)
- [Example Outcome](#example-outcome)
- [Core Capabilities](#core-capabilities)
- [How It Differs from Ordinary Prompt Templates](#how-it-differs-from-ordinary-prompt-templates)
- [Current Integration Method](#current-integration-method)
- [How It Works](#how-it-works)
- [Why It Is Credible](#why-it-is-credible)
- [Repository Structure](#repository-structure)
- [Boundary Notes](#boundary-notes)
- [License](#license)

## Overview

What this repository currently provides is a description-driven skill, rather than an independently executable program.

Its core purpose is not to answer questions directly, but to first convert a user request into an internal task-control object, namely an internal JSON prompt. You can think of it as: **a structured task specification used inside the model**. This object carries goals, constraints, assumptions, roles, review rules, and delivery requirements, and then determines the final output:

- `final_answer`: the final answer shown directly to the user
- `json_prompt`: a structured prompt package for downstream models
- `hybrid_prompt_package`: preserves both structured fields and the compiled instruction string

If your prompt needs to be reused, audited, handed to downstream models, or solidified into a skill, this governance layer is where it starts to become genuinely valuable.

## Why It Is Needed

When a prompt is just a one-off question, writing it clearly is usually enough.

But when a prompt becomes a reusable workflow, the problem is often not that “the wording is not long enough,” but that:

- after five rounds of edits to one prompt, no one can clearly explain which parts are hard constraints
- the same requirement produces unstable output structures when sent to different models
- you want to turn a one-off requirement into a skill, but you lack a review and delivery structure

- task boundaries tend to sprawl over time
- constraints are scattered across natural language and are hard to inspect
- prompt changes lack governance rules
- there is no `prompt_review` before generation
- there is no `answer_review` after generation
- downstream models receive too much information, making it both redundant and unstable

The goal of `ez prompt skill` is not to make prompts sound more polished, but to make this hidden control logic explicit: structure first, then generate; review first, then deliver.

## Who It Is For

If you are maintaining reusable prompt / skill workflows, this project is best suited to you.

It is also suitable for:

- users who often need to audit, upgrade, and reuse prompts
- people maintaining AI Agent / Skill workflows
- people who need to deliver natural-language requirements to downstream models or teams
- people who want the prompt generation process to be inspectable, traceable, and constrained

It is less suitable for:

- simple tasks where you only want a one-off chat reply
- lightweight Q&A that does not require structured delivery
- people who only want a ready-made prompt and do not care about the governance process

## What It Can Produce

It currently supports three kinds of deliverables:

| Deliverable | Suitable Scenario | Description | Illustrative Output Form |
|---|---|---|---|
| `final_answer` | Direct reply to the user | Natural-language delivery retained after review | A final response paragraph |
| `json_prompt` | Handed to a downstream model for continued execution | Returns a reviewed downstream JSON prompt package | Structured JSON fields |
| `hybrid_prompt_package` | When both structured fields and instruction compatibility are needed | Returns the delivery payload and, when needed, includes `compiled_instruction` | JSON fields + instruction string |

## Example Outcome

The example below uses only workflows and fields that actually exist in the repository, without introducing unimplemented capabilities.

User input:

> Help me optimize this prompt so it is better suited for generating a product proposal document, and output a reusable hybrid prompt package.

`ez prompt skill` will not start polishing immediately. Instead, it will first determine:

- this is not ordinary writing, but an upgrade task for an existing prompt
- it should preferably be routed to the formal `prompt-upgrade` slot category
- this type of task usually requires stricter governance intensity
- complex prompt upgrades should enable first-principles decomposition
- the final delivery should include not only revision suggestions, but also a reusable prompt package

So it is more likely to do these things first:

1. Check whether the goal, format, audience, and constraints are missing anything
2. Analyze how strict the control method for this task needs to be, namely `control_mode`
3. Draft `role_spec`, `task_spec`, and `structure_constraints`
4. Run `prompt_review`
5. Generate the `hybrid_prompt_package`
6. Then run `answer_review`

What you ultimately receive is not just “a rewritten prompt,” but:

- revision suggestions for the user
- structured delivery fields for a downstream model
- a compiled instruction string when necessary

### Illustrative Output Snippet

For the request above, the structured portion of the delivery result might look like this:

```json
{
  "control_mode": "audit",
  "first_principles_profile": "intensive",
  "template": {
    "name": "prompt-upgrade",
    "version": "v1"
  },
  "delivery_spec": {
    "artifact_type": "hybrid_prompt_package",
    "delivery_target": "downstream_model",
    "schema_version": "delivery-v1",
    "include_compiled_instruction": true
  },
  "delivery_payload": {
    "objective": "Upgrade the prompt for product proposal generation.",
    "constraints": ["Preserve the original goal.", "Return reusable output."],
    "role": {
      "identity": "prompt engineer",
      "stance": "structured and review-driven"
    },
    "compiled_instruction": "..."
  }
}
```

The point of this snippet is not to “make the prompt longer,” but to turn the task type, control intensity, delivery target, and downstream-executable fields into inspectable objects.

## Core Capabilities

The core of `ez prompt skill` is not “making prompts look better,” but making the prompt generation process more stable.

- **Task classification**: identify the task type first, then decide which slot it should enter
- **Governance intensity analysis**: use `control_mode` to determine how strict the control method for this task should be
- **First-principles decomposition**: break down complex tasks before writing the prompt
- **Structured control layer**: explicitly draft `role_spec`, `task_spec`, and `structure_constraints`
- **Pre-generation review**: use `prompt_review` to check whether the prompt itself is clear enough and stable enough
- **Post-generation review**: use `answer_review` to check whether the result has drifted, omitted something, or become falsely confident
- **Lean delivery**: by default, deliver only the necessary fields downstream, i.e. a slim payload, rather than exposing the entire internal governance object

## How It Differs from Ordinary Prompt Templates

Ordinary prompt templates usually focus on “how to write the wording well.”

`ez prompt skill` focuses more on “how to make the prompt generation process governable”:

| Comparison Item | Ordinary Prompt Templates | ez prompt skill |
|---|---|---|
| Core goal | Write a better prompt paragraph | Generate a prompt package that is reviewable and reusable |
| Control method | Rely on natural-language constraints | Carry constraints in structured fields |
| Review mechanism | Usually none | `prompt_review` + `answer_review` |
| Delivery method | A paragraph of text | `final_answer` / `json_prompt` / `hybrid_prompt_package` |
| Suitable scenarios | One-off tasks | prompt audit, prompt upgrade, Agent skill workflows |

## Current Integration Method

This repository is not yet a standalone CLI tool, and it does not yet provide package-manager installation entry points or Marketplace publishing instructions.

If you only want to try it out, the minimum viable approach at the current stage is: first obtain a copy of the repository, then use the corresponding skill directory according to the host type.

### 1. Obtain a Repository Copy

On the GitHub repository page, click `Code`, copy the repository URL, and clone it locally.

`(Insufficient information: the current README does not include the actual repository URL, so no clone command is shown directly here.)`

### 2. Use the Corresponding Directory by Host

#### Codex / Cursor

Use:

```text
.agents/skills/ez prompt skill/
```

This directory contains:

- `SKILL.md`
- `references/`
- `agents/openai.yaml`

#### Claude Code

Use:

```text
.claude/skills/ez prompt skill/
```

This directory contains:

- `SKILL.md`
- `references/`

### 3. Minimum Validation Method

After integration, you can first use a request like the following to verify whether the host recognizes the skill:

> Please upgrade this natural-language requirement into a governed JSON prompt, and output a hybrid_prompt_package.

If the host can recognize `ez prompt skill`, it should not immediately give you only a single finished prompt. It should first show:

- task classification or slot recognition
- governance intensity judgment
- first-principles or control-layer drafting
- review before delivering the result

### 4. Known Boundaries at the Current Stage

- whether automatic triggering happens depends on the host's own skill discovery and matching mechanism
- manual invocation syntax varies by host, so the host-supported skill invocation method should take precedence
- `agents/openai.yaml` is companion metadata specific to Codex / OpenAI, and it should not be assumed that Claude Code or Cursor will read it

## How It Works

The core idea of `ez prompt skill` is: **govern first, then generate; structure first, then deliver.**

You can understand several key terms like this first:

- `control_mode`: how strict the control method for this task needs to be
- `prompt_review`: checks whether the prompt itself is qualified before generation
- `answer_review`: checks whether the answer satisfies the objective after generation
- `slim payload`: give downstream models only the necessary fields, reducing redundancy and leakage

### Main Flow

1. Identify the task type and slot
2. Check whether any key information is missing that would materially change the result
3. Perform clarification only once; if the user asks to continue, record the minimum reasonable assumptions
4. Parse `control_mode`
5. Enable first principles for non-simple tasks
6. Draft `role_spec`, `task_spec`, `structure_constraints`, and `anti_sycophancy_harness`
7. Construct the internal JSON prompt
8. Run `prompt_review`
9. Generate the current target artifact
10. Run `answer_review`
11. Revise and perform a final check
12. Decide user-visible content according to `summary_visibility`

### Why Internal JSON Prompt Comes First

Because the core of this skill is not the final answer, but the internal governance object.

The value of this internal object mainly comes from three things:

- **Reviewability**: review can center on the structured object rather than only staring at the final prose
- **Reusability**: the prompt is no longer just a temporary text block, but a reviewable artifact
- **Safe delivery**: downstream models receive only the necessary fields by default, rather than the full governance metadata

## Why It Is Credible

The credibility of this project mainly comes from the structured materials that are already public in the repository:

| File | Description |
|---|---|
| `.agents/skills/ez prompt skill/SKILL.md` | proves that the main workflow, control layer, and review / revision rules exist |
| `.agents/skills/ez prompt skill/references/json-schema.md` | proves that the internal JSON prompt has structured constraints |
| `.agents/skills/ez prompt skill/references/template-registry.md` | proves that task type, slot, and overlay rules exist |
| `.agents/skills/ez prompt skill/references/delivery-schema.md` | proves that mapping rules exist from the internal object to the delivery payload |
| `1.0/mermaid-diagram.png` | proves that the workflow can be visually reviewed |
| `1.0/ez prompt skill_workflow_analysis.pdf` | proves that independent workflow reconstruction and review materials already exist |

Taken together, these files show that this is not just verbal methodology, but a structured skill that can be read, inspected, and reviewed.

## Repository Structure

The core content that actually exists in the current repository is as follows:

```text
.
├── .agents/
│   └── skills/
│       └── ez prompt skill/
│           ├── SKILL.md
│           ├── PORTABILITY.md
│           ├── agents/
│           │   └── openai.yaml
│           └── references/
│               ├── json-schema.md
│               ├── template-registry.md
│               └── delivery-schema.md
│── .claude/
│   └── skills/
│       └── ez prompt skill/
│           ├── SKILL.md
│           └── references/
│               ├── json-schema.md
│               ├── template-registry.md
│               └── delivery-schema.md
│
│── LICENSE
└── README.md

```

Key file notes:

- `SKILL.md`: main workflow, control layer, and review / revision rules
- `references/json-schema.md`: internal governance JSON schema
- `references/template-registry.md`: task type, slot, and overlay rules
- `references/delivery-schema.md`: mapping rules from the internal object to the delivery payload
- `PORTABILITY.md`: compatibility boundaries for Codex / Cursor / Claude Code
- `1.0/`: workflow analysis materials and diagrams prepared for public release

## Boundary Notes

`ez prompt skill` is currently more suitable as a governance layer inside an AI Agent / Skill host environment than as independently running software.

Current known boundaries:

- no runtime code or orchestration scripts have been found in the repository
- the currently visible main body consists of skill documents, schema, delivery mapping, and the adaptation layer
- automatic triggering, approval pause/resume, and host-level execution details still depend on external host implementations

If your task is only a one-off lightweight Q&A, writing the prompt directly may be lighter.


## Star History

<a href="https://www.star-history.com/?repos=Rayyy3320%2Fez prompt skill&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=Rayyy3320/ez prompt skill&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=Rayyy3320/ez prompt skill&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=Rayyy3320/ez prompt skill&type=date&legend=top-left" />
 </picture>
</a>

## License

MIT License
