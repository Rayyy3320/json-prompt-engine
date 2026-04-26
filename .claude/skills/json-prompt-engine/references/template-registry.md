# Template Registry

This file defines the starter template slots for vNext.
The registry stays objective-first: choose the dominant output objective first, then add governance, review, and delivery overlays.
Do not invent a new base slot for every control or delivery variation.

## Base slots

### writing-basic

Use for drafting original content.
Emphasize objective, audience, tone, length, and structure.
For non-trivial writing tasks, add explicit role framing, first-principles decomposition, and structure constraints.

### rewrite-basic

Use for rewriting, polishing, compressing, expanding, or restyling existing text.
Emphasize source preservation, change target, tone shift, forbidden losses, and drift control.
Bypass first principles only when the rewrite is clearly simple.

### summary-basic

Use for summarization.
Emphasize source scope, compression level, audience, output format, and omission control.
Use first principles when the summary task includes constraints, analysis, or output shaping beyond straightforward compression.

### analysis-basic

Use for analysis, evaluation, comparison, diagnosis, and reasoning-heavy responses.
Emphasize criteria, evidence, tradeoffs, conclusion style, anti-sycophancy challenge behavior, and first-principles alignment.

### plan-basic

Use for plans, outlines, roadmaps, procedures, and step-by-step recommendations.
Emphasize sequencing, dependencies, actionability, boundary handling, and first-principles decomposition of the real problem.

### qa-basic

Use for direct question answering.
Emphasize precision, completeness, explanation depth, uncertainty handling, and whether examples are needed.
Bypass first principles only when the question is clearly simple.

## Formal prompt slots

### prompt-audit

Use when the dominant objective is to inspect an existing prompt for ambiguity, hallucination risk, execution drift, governance weakness, or missing constraints.

### prompt-upgrade

Use when the dominant objective is to improve an existing prompt so it becomes more executable, stable, reviewable, and governable.

## Cross-cutting overlays

For non-trivial tasks, every slot should be overlaid with the following control layers:

### First-principles overlay

Define the real problem before drafting the prompt.
Use the fixed sequence:

1. `problem_decomposition`
2. `knowledge_foundation`
3. `execution_path`

Use this overlay by default for all non-trivial tasks and whenever the user explicitly requests first principles.
Use `first_principles_profile = intensive` for audit-grade prompt work or other explicitly high-rigor work.

### Role overlay

Define who the model is acting as, what expertise matters, and what stance improves judgment.
Do not use decorative roles that do not change execution.

### Task overlay

Define the primary goal, success criteria, out-of-scope items, and fallback behavior.
This overlay is where vague user wording gets sharpened into an executable task.
When first principles are enabled, align the task goal with the cleaned first-principles goal.

### Structure-constraints overlay

Define required sections, ordering, format rules, and content rules when output stability matters.

### Anti-sycophancy overlay

Define principles, forbidden behaviors, must-challenge conditions, and self-check questions.
This is especially important for analysis, review, prompt design, and advisory tasks.

### Stage-review overlay

Use stage-specific review instead of a single vague review pass.

- `prompt_review` should inspect structural preparedness before artifact generation
- `answer_review` should inspect manifested output quality after artifact generation

If `failed_first_principles_alignment` fires, use restructure rather than cosmetic patching.

### Change-governance overlay

Use `change_governance` to determine whether the revision can be auto-applied.
Do not rely on review severity alone to decide whether to pause for user approval.

### Delivery overlay

Use this overlay to decide whether the final artifact should be a final answer or a downstream JSON prompt package.
This overlay:

- does not change base-slot selection
- does not change the meaning of `control_mode`
- does not change the meaning of `summary_visibility`
- does not create a new base slot

When `delivery_spec.artifact_type != final_answer`, the delivery overlay should:

- compile a slim delivery payload
- add delivery-specific prompt-review dimensions
- add delivery-specific answer-review dimensions
- run artifact-level quality checks

## First-principles default trigger policy

Apply first principles with these defaults:

- `analysis`, `planning`, diagnosis-like tasks, prompt design, and multi-constraint writing: enable by default
- simple QA and simple rewriting: bypass by default
- explicit user request for first principles: always enable

## Task-type and slot mapping

- writing -> `writing-basic`
- rewriting -> `rewrite-basic`
- summarization -> `summary-basic`
- analysis -> `analysis-basic`
- planning -> `plan-basic`
- qa -> `qa-basic`
- marketing -> `writing-basic` unless a more specific marketing template is added later
- coding -> `qa-basic` unless a code-focused template is added later

Special routing:

- if the user is primarily inspecting an existing prompt, choose `prompt-audit`
- if the user is primarily improving an existing prompt, choose `prompt-upgrade`

## Selection rule

Pick the closest slot first.
If the request spans multiple task types, choose the dominant output objective rather than blending multiple slots unless blending is clearly necessary.

Then add the required overlays rather than inventing a new slot for every request.
If the final artifact type changes, apply the delivery overlay instead of creating a new slot.

## Review profiles by slot

Use the slot's minimal profile as the baseline.
Then adapt it by control mode.

When the delivery overlay is active and `artifact_type != final_answer`, extend `prompt_review` and `answer_review` with delivery-artifact checks.

## Visibility rule

Use top-level `summary_visibility` to control global display expansion.
Do not use `summary_visibility` to control delivery artifact type.
If a slot needs richer first-principles display, rely on expanded mode or local `first_principles_visibility`.

## Future expansion points

Planned additions can still include:

- marketing-launch
- marketing-ad-copy
- coding-explain
- coding-generate
- business-plan
- research-summary
- bilingual-output

Keep delivery as an overlay concept rather than splitting it into a dedicated base slot.
Keep `first-principles-intensive` as a profile concept, not a future slot.
