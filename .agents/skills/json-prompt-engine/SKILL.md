---
name: json-prompt-engine
description: turn a natural-language request into a governed internal json prompt, choose an appropriate control mode, apply first-principles decomposition when needed, run stage-specific reviews, and produce either a final answer or a reviewed downstream JSON prompt package. use for chinese-first prompt engineering workflows such as writing, rewriting, summarizing, analysis, planning, q&a, prompt audit, prompt upgrade, or whenever the user asks to structure a request, generate a json prompt, refine a prompt, or hand off a reviewed prompt artifact to another model.
---

# Json Prompt Engine

## Overview

Use this skill to convert a user's request into a stable internal json prompt before solving the task.
Keep the workflow JSON-first, and extend it with these control layers:

- control mode
- first principles
- role
- task
- structure constraints
- anti-sycophancy harness engineering
- stage-specific review
- change governance
- delivery model

This version keeps the existing review and revision chain intact.
The goal is not to delete the back half of the workflow, but to extend the output model from:

- internal json prompt -> natural-language answer

to:

- internal json prompt -> reviewed downstream JSON prompt package

while still preserving:

- `prompt_review -> answer_generation -> answer_review -> quality checks`

The internal governance object remains the full internal json prompt.
The external handoff object can now be either:

- a final answer
- a reviewed downstream JSON prompt package
- a hybrid prompt package that includes both structured delivery fields and a compiled instruction string

## Delivery model

Use `delivery_spec` to decide what the final artifact should be.

Supported `artifact_type` values:

- `final_answer`
- `json_prompt`
- `hybrid_prompt_package`

Interpretation:

- `final_answer`
  - keep the classic behavior and return the reviewed answer to the user
- `json_prompt`
  - return a reviewed downstream JSON prompt package
- `hybrid_prompt_package`
  - return a reviewed downstream JSON prompt package and include a compiled instruction string when appropriate

When `delivery_spec` is absent, preserve the old behavior:

- `artifact_type = final_answer`
- `delivery_target = user`

`summary_visibility` only controls how much intermediate structure is shown.
It does not decide whether the final artifact is a final answer or a delivery JSON package.

## Internal json vs delivery json

Use the two layers for different jobs:

- internal json prompt
  - governance, decomposition, review, revision, and safety boundaries
- delivery JSON payload
  - minimal downstream execution contract for another model or tool

Do not expose the full internal json prompt to downstream execution by default.
Compile a slim delivery payload instead.

## Core guarantees

- One clarification round at most
- Stable internal json representation
- Explicit governance intensity through `control_mode`
- Default first-principles decomposition for non-trivial tasks
- Explicit role and task framing for non-trivial requests
- Output structure constraints that reduce drift
- Anti-sycophancy self-checks that resist flattery and premature approval
- Stage-specific `prompt_review` before artifact generation when needed
- Stage-specific `answer_review` after artifact generation
- Mandatory revision when review triggers are hit
- User approval boundaries governed through `change_governance`
- Internal governance and external delivery remain separate layers
- The downstream payload stays slim and does not automatically inherit full governance metadata

## Extended workflow

Follow this sequence every time:

1. Identify the task type and the closest template slot.
2. Check whether critical requirements are complete.
3. If critical requirements are missing, ask for clarification once.
4. If the user does not add details but explicitly wants to continue, proceed with minimal reasonable assumptions.
5. Resolve `control_mode`.
6. Decide whether to enable the first-principles layer.
7. If enabled, run the first-principles pass in this fixed order:
   - `problem_decomposition`
   - `knowledge_foundation`
   - `execution_path`
8. If first-principles correction requires approval, pause and ask the user according to `change_governance`.
9. Draft the control layer:
   - `role_spec`
   - `task_spec`
   - `structure_constraints`
   - `anti_sycophancy_harness`
10. Build an internal json prompt using the schema in [references/json-schema.md](references/json-schema.md).
11. Run `prompt_review`.
12. If `prompt_review` triggers are hit, revise the internal json prompt before generating the current artifact.
13. `answer_generation`
    - generate the first-pass artifact from the current internal json prompt
    - if `delivery_spec.artifact_type = final_answer`, generate the first-pass answer
    - if `delivery_spec.artifact_type = json_prompt`, compile the first-pass downstream delivery JSON
    - if `delivery_spec.artifact_type = hybrid_prompt_package`, compile the delivery JSON and include `compiled_instruction` when enabled
14. `answer_review`
    - review the current artifact rather than assuming the artifact is always a natural-language answer
    - for `final_answer`, use the normal answer-quality checks
    - for `json_prompt` and `hybrid_prompt_package`, emphasize delivery schema quality, field determinism, downstream executability, serialization minimality, and token economy
15. `revision`
    - if issues are found, revise the internal json prompt or the internal-to-delivery mapping, then regenerate the current artifact
16. `quality checks`
    - run final checks for the current artifact type
17. `respond`
    - return the final answer or the reviewed delivery artifact according to `delivery_spec`
    - use `summary_visibility` only for intermediate display control

## Workflow insertion map

This skill extends the original workflow instead of replacing it.

- `control_mode` is resolved after clarification and before heavy control layers.
- `first_principles_spec` is inserted after control-mode resolution and before the other control layers.
- `role_spec`, `task_spec`, `structure_constraints`, and `anti_sycophancy_harness` are drafted after first-principles analysis.
- `prompt_review` is inserted after internal json construction and before artifact generation.
- `answer_generation` now means artifact generation, not only answer generation.
- `answer_review` is inserted after first-pass artifact generation and before final quality checks.
- `delivery_spec` and `delivery_payload` affect only what gets delivered, not how governance fields are interpreted.

## Default behavior

### Governance and execution defaults

- Use `execution_strategy.mode = fast` by default for execution style.
- Use `control_mode = standard` by default unless the task is clearly lite or clearly audit-grade.
- Keep `control_mode`, `execution_strategy.mode`, `summary_visibility`, and `delivery_spec` semantically separate.
  - `control_mode` controls governance intensity and review rigor.
  - `execution_strategy.mode` controls execution style or speed.
  - `summary_visibility` controls display visibility only.
  - `delivery_spec` controls the final artifact type only.

### Summary visibility

Use top-level `summary_visibility` as the only authoritative display field.

- `compact`: normal behavior, expose only compact intermediate conclusions when necessary
- `full`: expanded behavior, expose structured intermediate artifacts in full

If a first-principles-local visibility setting is needed, use `first_principles_spec.first_principles_visibility`.
That local field should inherit from top-level `summary_visibility` when absent.

Do not use `summary_visibility` as a delivery-type switch.

### User-facing tone

Write to the user in Chinese unless the user clearly requests another language.
Keep the tone professional, concise, direct, and tool-like.

### Internal prompt language

Prefer an English instruction skeleton for `final_prompt` because it is often more stable.
Add explicit output-language requirements inside the prompt so the final answer still matches the user's requested language.

## Clarification rule

Check these fields first:

- objective
- output format
- target audience when relevant
- language when relevant
- tone/style when relevant
- length/detail level when relevant
- hard constraints

Ask for clarification once only when one or more missing items would materially change the result.
Use a short, direct reminder that lists only the missing critical items.

If the user refuses to clarify or says to continue anyway, continue with minimal reasonable assumptions.
Also record the assumptions inside the internal json prompt.

## Task classification and slot selection

Classify the request before building the json prompt.
Use one of these base task types unless a more specific subtype is clearly useful:

- writing
- rewriting
- summarization
- analysis
- planning
- qa
- marketing
- coding

Then choose the closest slot from [references/template-registry.md](references/template-registry.md).
Prefer explicit slots such as `prompt-audit` or `prompt-upgrade` when the user is inspecting or improving an existing prompt rather than solving a normal content task.

## Control-mode resolution

Resolve `control_mode` in this fixed order:

1. explicit user instruction
2. slot default
3. risk-based escalation
4. fallback to `standard`

## First-principles pass

For non-trivial tasks, enable a first-principles pass by default.
Use this layer to get back to the core problem before drafting the final prompt.

### Default trigger policy

- skip by default for simple question answering and simple rewriting
- enable by default for all other tasks
- always enable when the user explicitly asks to use first principles

### First-principles profile

Use `first_principles_profile` to record rigor level.

- `normal`
  - default when first principles are enabled in `standard`
- `intensive`
  - preferred in `audit`
  - allowed in high-risk `standard` when escalation is justified

### Required decomposition order

1. `problem_decomposition`
2. `knowledge_foundation`
3. `execution_path`

### Correction protocol

When the first-principles pass identifies a need to correct the task framing:

- keep `correction_policy.current_assessment` as `none`, `medium`, or `strong`
- route user-facing approval behavior through `change_governance`
- do not rewrite the task definition automatically when approval is required
- if the user rejects a proposed change, preserve the original definition and keep the risk visible in later review

## Change governance

Use `change_governance` to determine whether a change can happen automatically.
It is the authoritative bridge between framing assessment, revision depth, and user interaction.

Use these values:

- `change_scope`
  - `micro`
  - `local`
  - `structural`
- `approval_required`
  - `no`
  - `yes`

## Json prompt construction

Always build an internal json object before execution.
Use the schema and field rules in [references/json-schema.md](references/json-schema.md).
Use [references/delivery-schema.md](references/delivery-schema.md) when compiling the downstream delivery payload.

Rules:

- Keep keys stable and in English.
- Put user-facing content and assumptions in the language that best supports execution.
- Set `clarification_needed` accurately.
- Fill `missing_requirements` only with genuinely missing critical items.
- Keep `final_prompt` concise, directive, and executable.
- Include `delivery_spec` and `delivery_payload`.
- Include top-level `summary_visibility`.
- Include `control_mode`, `mode_resolution`, and `first_principles_profile` when relevant.
- Include `first_principles_spec` when its trigger policy enables it.
- Include the other control-layer fields when the task is non-trivial.
- Use stage-specific `review_spec.prompt_review` and `review_spec.answer_review`.
- Do not expose the full internal json prompt unless the user explicitly asks.

### Internal-to-delivery compilation

The internal json prompt is the upstream governance object.
When `delivery_spec.artifact_type != final_answer`, compile a slim delivery payload from the internal json prompt.
The slim delivery payload should contain only the minimum sufficient fields needed for downstream execution.

Do not export the following by default:

- full `first_principles_spec`
- full `review_spec`
- `change_governance`
- pending change summaries
- internal review traces
- `mode_resolution`
- `control_floor`

Keep `final_prompt`, but treat it as an optional compiled instruction source rather than the only delivery form.
If `delivery_spec.include_compiled_instruction = true`, the delivery payload may include `compiled_instruction` derived from the internal `final_prompt`.

## Prompt review

Run `prompt_review` after the internal json prompt is created and before artifact generation.
This stage exists to catch structural risk before spending a generation round.

`prompt_review` should always focus on:

- role adequacy
- task-boundary clarity
- structure completeness
- output-protocol stability
- first-principles alignment
- anti-sycophancy strength

When `delivery_spec.artifact_type != final_answer`, also add these delivery-specific dimensions:

- delivery-schema completeness
- field determinism
- serialization minimality
- downstream compatibility

Delivery-specific `prompt_review` triggers include:

- `missing_delivery_required_field`
- `conflicting_delivery_fields`
- `over_verbose_delivery_payload`
- `ambiguous_instruction_priority`

If a trigger is hit, revise the internal prompt or the delivery mapping before generating the artifact.
If the required revision crosses a governance boundary, pause and ask the user first.

## Answer review

Run `answer_review` after the first-pass artifact is generated.
This stage exists to catch manifested artifact risk.

If `delivery_spec.artifact_type = final_answer`, review the answer normally.
If `delivery_spec.artifact_type != final_answer`, review the delivery artifact as the current output.

For `final_answer`, focus on:

- ask alignment
- omission risk
- drift in actual output
- false confidence
- audience fit
- clarity of final delivery

For `json_prompt` and `hybrid_prompt_package`, focus on:

- legal JSON serialization
- schema stability
- constraint retention
- payload size discipline
- output spec determinism
- absence of leaked human-only commentary

Artifact-specific `answer_review` triggers include:

- `invalid_json`
- `schema_drift`
- `constraint_loss_detected`
- `delivery_payload_too_large`
- `output_spec_ambiguous`
- `human_commentary_leakage`

If a trigger is hit:

1. revise the internal json prompt or delivery mapping
2. regenerate the current artifact when needed
3. re-check the revised result

Do not treat `prompt_review` and `answer_review` as the same checklist repeated twice.

## Revision policy

Do not stop at commentary.
Do not mark the prompt or delivery artifact as good enough when a serious execution risk is still unresolved.

If `failed_first_principles_alignment` is hit, run a first-principles-driven restructure instead of a local patch.
If the required restructure would change the task frame materially, ask the user first according to `change_governance`.

## Quality checks

Before responding, verify that the current artifact:

- addresses the actual objective
- follows the requested format
- uses the requested language and tone
- respects hard constraints
- does not omit important requested points
- is not obviously repetitive or bloated
- does not carry unresolved critical execution risks found during review
- does not bypass first-principles correction rules when they were triggered
- does not violate slot control floors when they apply

When `delivery_spec.artifact_type != final_answer`, also verify:

- it is valid JSON
- it satisfies the declared delivery schema version
- it contains only the fields needed downstream
- it respects token budget expectations
- it does not leak internal governance metadata
- it retains the minimum sufficient execution information

## Expanded mode

Only expose intermediate artifacts when the user explicitly asks.

When expanded mode is requested, show the following in order:

1. brief prompt design draft
2. complete `first_principles_spec` when enabled
3. internal json prompt
4. prompt-review summary
5. first-pass delivery artifact or first-pass answer
6. answer-review summary
7. revised delivery artifact or revised answer
8. final delivery artifact or final answer

Do not present hidden reasoning as chain-of-thought.
Present only the structured artifacts needed for the workflow.

## Template evolution

This version keeps the base template registry structure but adds explicit delivery routing through an overlay rather than a new slot.
Choose the closest slot first, then layer first principles, role, task, structure constraints, anti-sycophancy controls, stage-specific reviews, and the delivery overlay onto it.
