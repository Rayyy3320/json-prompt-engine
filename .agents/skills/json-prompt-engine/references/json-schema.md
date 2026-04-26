# JSON Schema Reference

Use this schema as the default internal representation.
Keep keys in English for stability.

## Default schema

```json
{
  "task_type": "",
  "user_request": "",
  "objective": "",
  "context": "",
  "target_audience": "",
  "input_data": "",
  "constraints": [],
  "missing_requirements": [],
  "clarification_needed": false,
  "clarification_message": "",
  "assumptions": [],
  "summary_visibility": "compact",
  "control_mode": "standard",
  "mode_resolution": {
    "control_mode": "standard",
    "source": "default_fallback",
    "reason": ""
  },
  "control_floor": "",
  "first_principles_profile": "normal",
  "delivery_spec": {
    "artifact_type": "final_answer",
    "delivery_target": "user",
    "schema_version": "delivery-v1",
    "serialization_mode": "slim",
    "include_compiled_instruction": true
  },
  "delivery_payload": {},
  "first_principles_spec": {
    "enabled": false,
    "trigger_mode": "bypassed_simple",
    "first_principles_visibility": "",
    "problem_decomposition": {
      "core_problem": "",
      "goal": "",
      "subproblems": [],
      "variables": [],
      "constraints": [],
      "boundaries": []
    },
    "knowledge_foundation": {
      "known_facts": [],
      "assumptions": [],
      "unknowns": [],
      "questionable_premises": [],
      "evidence_requirements": []
    },
    "execution_path": {
      "starting_point": "",
      "minimal_viable_path": [],
      "validation_points": [],
      "failure_modes": [],
      "fallbacks": []
    },
    "correction_policy": {
      "default_level": "medium",
      "current_assessment": "none",
      "needs_user_approval_for_medium": true,
      "must_escalate_for_strong": true,
      "pending_change_summary": [],
      "proposed_rewrites": [],
      "user_rejected": false
    }
  },
  "change_governance": {
    "change_scope": "micro",
    "approval_required": "no",
    "reason": "",
    "affected_fields": [],
    "fallback_if_denied": "",
    "can_auto_apply": true
  },
  "role_spec": {
    "identity": "",
    "expertise": [],
    "stance": "",
    "do_not_do": []
  },
  "task_spec": {
    "primary_goal": "",
    "success_criteria": [],
    "out_of_scope": [],
    "failure_handling": []
  },
  "structure_constraints": {
    "required_sections": [],
    "ordering": [],
    "format_rules": [],
    "content_rules": []
  },
  "anti_sycophancy_harness": {
    "enabled": true,
    "principles": [],
    "forbidden_behaviors": [],
    "must_challenge": [],
    "self_check_questions": []
  },
  "output_spec": {
    "language": "",
    "format": "",
    "tone": "",
    "length": "",
    "structure": []
  },
  "template": {
    "name": "",
    "version": "v1"
  },
  "execution_strategy": {
    "mode": "fast",
    "steps": []
  },
  "review_spec": {
    "prompt_review": {
      "enabled": true,
      "dimensions": [
        {
          "name": "",
          "inspect": "",
          "common_failures": [],
          "handling": ""
        }
      ],
      "triggers": [
        {
          "condition": "",
          "severity": "minor",
          "required_action": ""
        }
      ],
      "pass_criteria": [],
      "max_revision_rounds": 1
    },
    "answer_review": {
      "enabled": true,
      "dimensions": [
        {
          "name": "",
          "inspect": "",
          "common_failures": [],
          "handling": ""
        }
      ],
      "triggers": [
        {
          "condition": "",
          "severity": "minor",
          "required_action": ""
        }
      ],
      "pass_criteria": [],
      "max_revision_rounds": 1
    }
  },
  "quality_checks": [],
  "final_prompt": ""
}
```

## Field rules

### task_type

Use the base classifier from the skill unless a more specific subtype is clearly useful.
The slot choice may be more specific than the base task type.

### user_request

Store the user's original request or a minimally normalized version.

### objective

Rewrite the core task as a crisp execution goal.
If `first_principles_spec.enabled` is true, this field should align with the cleaned goal from `problem_decomposition.goal`.

### context

Capture only context that materially changes the output.

### target_audience

Fill when audience matters for tone, detail level, or framing.

### input_data

Use when the request depends on source text, notes, code, or other provided material.

### constraints

List only real hard constraints such as format, banned content, required sections, deadlines, style locks, or exact length caps.

### missing_requirements

List only the missing items that would materially change the result.
Do not pad this list.

### clarification_needed

Set to `true` only when missing requirements justify a clarification.

### clarification_message

Write one concise Chinese reminder listing only the missing critical items.

### assumptions

Use only when the request must continue without full clarification.
Keep assumptions minimal and explicit.
If first principles are enabled, keep this field aligned with `knowledge_foundation.assumptions`.

### summary_visibility

Use this top-level field as the authoritative display control for user-facing intermediate artifacts.

Allowed values:

- `compact`
- `full`

Interpretation:

- `compact` is the default normal mode
- `full` is the expanded mode

If a legacy schema still stores `first_principles_spec.summary_visibility`, migrate that value upward during interpretation when the top-level field is absent.

Do not use `summary_visibility` to control the final delivery artifact type.

### control_mode

Use this field to represent governance intensity.

Allowed values:

- `lite`
- `standard`
- `audit`

Do not confuse this with `execution_strategy.mode`.

- `control_mode` controls governance intensity and review rigor
- `execution_strategy.mode` controls execution style or speed
- `summary_visibility` controls display visibility
- `delivery_spec` controls delivery artifact type

### mode_resolution

Use this block to record how `control_mode` was chosen.

Allowed `source` values:

- `user_forced`
- `slot_default`
- `risk_escalated`
- `default_fallback`

### control_floor

Use this field when a slot has a minimum governance level that cannot be silently downgraded.
It can stay empty for normal tasks.
Typical value:

- `audit`

Slots such as `prompt-audit` and `prompt-upgrade` should set `control_floor = audit`.

### first_principles_profile

Use this field to represent first-principles rigor.

Allowed values:

- `normal`
- `intensive`

### delivery_spec

Use this block to control the final delivery artifact.
This is the only delivery-type control layer.
Do not reuse `summary_visibility`, `control_mode`, or `execution_strategy.mode` for this job.

Allowed values:

- `artifact_type`
  - `final_answer`
  - `json_prompt`
  - `hybrid_prompt_package`
- `delivery_target`
  - `user`
  - `downstream_model`
- `schema_version`
  - default: `delivery-v1`
- `serialization_mode`
  - default: `slim`
  - do not use `full` as the normal export mode
- `include_compiled_instruction`
  - whether `delivery_payload.compiled_instruction` should be populated from the internal `final_prompt`

Compatibility behavior:

- if `delivery_spec` is absent, preserve the old behavior
- default to `artifact_type = final_answer`
- default to `delivery_target = user`

### delivery_payload

Use this block as the downstream execution carrier.
It is a slim delivery object, not a copy of the full internal governance object.

Recommended slim delivery shape:

```json
{
  "schema_version": "delivery-v1",
  "artifact_type": "json_prompt",
  "delivery_target": "downstream_model",
  "instruction_precedence": [
    "constraints",
    "output_spec",
    "objective",
    "context",
    "input_data"
  ],
  "objective": "",
  "context": "",
  "input_data": "",
  "constraints": [],
  "assumptions": [],
  "role": {
    "identity": "",
    "stance": ""
  },
  "execution_steps": [],
  "output_spec": {
    "language": "",
    "format": "",
    "tone": "",
    "length": "",
    "structure": []
  },
  "quality_bar": [],
  "compiled_instruction": ""
}
```

If `artifact_type = final_answer`, `delivery_payload` may remain empty.
If `artifact_type != final_answer`, `delivery_payload` should contain the minimum sufficient downstream fields.

### instruction_precedence

JSON objects do not have natural instruction ordering.
Use `delivery_payload.instruction_precedence` to define how downstream consumers should resolve conflicts.

Recommended default precedence:

1. `constraints`
2. `output_spec`
3. `objective`
4. `context`
5. `input_data`

If `artifact_type != final_answer` and `instruction_precedence` is missing, treat the delivery artifact as incomplete.

### first_principles_spec

Use this block when the task is non-trivial or when the user explicitly requests first principles.

- `enabled`: whether first-principles decomposition is active
- `trigger_mode`: recommended values are `auto_complex`, `user_forced`, and `bypassed_simple`
- `first_principles_visibility`: optional local visibility rule for first-principles output only

If `first_principles_visibility` is empty, inherit from top-level `summary_visibility`.

#### correction_policy

Use this block to govern how first-principles correction is assessed.
Keep its current semantics.
Route approval behavior through `change_governance` rather than using this block as a delivery control layer.

### change_governance

Use this block to determine whether a change can be applied automatically.

Allowed values:

- `change_scope`
  - `micro`
  - `local`
  - `structural`
- `approval_required`
  - `no`
  - `yes`

### role_spec

Use this block to make the operating role explicit.

### task_spec

Use this block to sharpen the task beyond the user's raw wording.
If first principles are enabled, align `primary_goal` with `first_principles_spec.problem_decomposition.goal`.

### structure_constraints

Use this block to stabilize the answer shape.

### anti_sycophancy_harness

Use this block to reduce flattery, premature approval, weak assumptions, and false completion.

### output_spec

Use this block to define the required output form.
It should also survive export into `delivery_payload` when a downstream artifact is requested.

### template

Record the chosen template slot and version.

### execution_strategy

Use `fast` by default.
Keep `steps` brief and action-oriented.

### review_spec

Use this block to force stage-specific objective review.

#### prompt_review

Use this block to inspect structural risk before artifact generation.
When `delivery_spec.artifact_type != final_answer`, add delivery-specific dimensions such as:

- delivery-schema completeness
- field determinism
- serialization minimality
- downstream compatibility

Recommended delivery-specific prompt-review triggers:

- `missing_delivery_required_field`
- `conflicting_delivery_fields`
- `over_verbose_delivery_payload`
- `ambiguous_instruction_priority`

#### answer_review

Use this block to inspect manifested output risk after artifact generation.
When `delivery_spec.artifact_type != final_answer`, add delivery-artifact checks such as:

- `invalid_json`
- `schema_drift`
- `constraint_loss_detected`
- `delivery_payload_too_large`
- `output_spec_ambiguous`
- `human_commentary_leakage`

### quality_checks

Include only the checks relevant to the current task and artifact type.

### final_prompt

Keep `final_prompt` as a valid internal compiled instruction source.
It is no longer the only delivery form.
When `delivery_spec.include_compiled_instruction = true`, a slimmed form of `final_prompt` may be written to `delivery_payload.compiled_instruction`.

## Delivery export whitelist / blacklist

Use these as principles rather than a rigid serializer spec.
For detailed mapping, see [delivery-schema.md](delivery-schema.md).

Export whitelist:

- `objective`
- `context`
- `input_data`
- `constraints`
- `assumptions`
- `role.identity`
- `role.stance`
- `execution_steps`
- `output_spec`
- `quality_bar`
- `compiled_instruction`

Never-export by default:

- `mode_resolution`
- `control_floor`
- full `first_principles_spec`
- full `review_spec`
- `change_governance`
- pending change summaries
- internal review traces

## Compatibility and downgrade behavior

When reading older prompts:

- if `delivery_spec` is absent, preserve the old final-answer behavior
- if `delivery_payload` is absent, still allow classic behavior
- if `summary_visibility` is absent, assume `compact`
- if `control_mode` is absent, assume `standard`
- if `mode_resolution` is absent, treat as `default_fallback`
- if `first_principles_profile` is absent, assume `normal`
- if legacy `first_principles_spec.summary_visibility` exists and top-level `summary_visibility` is absent, migrate its value upward during interpretation
- if stage-specific review blocks are absent, interpret the old review behavior through `answer_review` semantics and note the migration requirement in docs

## Prompt language policy

Default strategy:

1. Build the structural keys in English.
2. Prefer an English instruction skeleton for `final_prompt`.
3. Add explicit output-language requirements.
4. Preserve essential user wording when it is important to style or domain meaning.
