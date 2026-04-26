# Delivery Schema

## 1. Purpose

This reference defines how the skill maps the full internal governance JSON into a reviewed downstream delivery artifact.

The delivery layer exists so the skill can hand off a clean, model-readable package without leaking internal governance metadata.

## 2. Internal JSON vs Delivery JSON

Use the two layers for different jobs:

- internal JSON
  - decomposition, governance, review, revision, and safety control
- delivery JSON
  - slim downstream execution contract

Do not expose the internal JSON by default.
Compile a slimmer delivery artifact instead.

## 3. Required delivery fields

For `artifact_type = json_prompt` or `artifact_type = hybrid_prompt_package`, the delivery artifact should contain at least:

- `schema_version`
- `artifact_type`
- `delivery_target`
- `instruction_precedence`
- `objective`
- `constraints`
- `output_spec`

Recommended additional fields when relevant:

- `context`
- `input_data`
- `assumptions`
- `role`
- `execution_steps`
- `quality_bar`
- `compiled_instruction`

## 4. Internal-to-delivery mapping

Recommended mapping:

- internal `objective` -> delivery `objective`
- internal `context` -> delivery `context`
- internal `input_data` -> delivery `input_data`
- internal `constraints` -> delivery `constraints`
- internal `assumptions` -> delivery `assumptions`
- internal `role_spec.identity` -> delivery `role.identity`
- internal `role_spec.stance` -> delivery `role.stance`
- internal execution summary or plan steps -> delivery `execution_steps`
- internal `output_spec` -> delivery `output_spec`
- internal quality expectations -> delivery `quality_bar`
- internal `final_prompt` -> delivery `compiled_instruction` when `include_compiled_instruction = true`

Do not attempt a 1:1 dump from internal JSON to delivery JSON.
Export only the minimum sufficient fields.

## 5. Export whitelist

Whitelist principles:

- export only fields needed for downstream execution
- prefer short structured fields over prose duplication
- prefer normalized role and output fields over raw governance traces

Recommended export whitelist:

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

## 6. Never-export fields

Never export the following by default:

- full `first_principles_spec`
- full `review_spec`
- `change_governance`
- `mode_resolution`
- `control_floor`
- pending change summaries
- proposed rewrites
- internal review traces
- full anti-sycophancy harness internals unless they are deliberately transformed into concise executable constraints

## 7. Instruction precedence rules

JSON fields do not define natural instruction priority.
Use `instruction_precedence` to define downstream conflict resolution.

Recommended default order:

1. `constraints`
2. `output_spec`
3. `objective`
4. `context`
5. `input_data`

If the downstream package includes both a structured delivery payload and a `compiled_instruction`, the structured fields should still remain authoritative.
The compiled instruction is a compatibility layer, not the new single source of truth.

## 8. Token economy rules

The delivery artifact should be slimmer than the internal governance object.

Use these rules:

- do not duplicate the same instruction in multiple fields unless the redundancy is intentionally stabilizing
- prefer short lists over repeated prose
- prefer fielded structure over long commentary
- remove governance-only fields before export
- keep `compiled_instruction` short and only include it when it helps downstream execution
- if the payload becomes too large, trim commentary before trimming execution-critical constraints

## 9. Example: json_prompt

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
  "objective": "Explain what depth an AI product manager needs for RAG learning.",
  "context": "The answer should distinguish must-have knowledge from optional depth.",
  "input_data": "",
  "constraints": [
    "Respond in simplified Chinese.",
    "Keep the answer practical and structured."
  ],
  "assumptions": [],
  "role": {
    "identity": "AI product education assistant",
    "stance": "structured and practical"
  },
  "execution_steps": [
    "Explain the minimum required understanding.",
    "Separate must-have, should-know, and optional depth.",
    "Call out role boundaries versus engineering depth."
  ],
  "output_spec": {
    "language": "simplified Chinese",
    "format": "markdown",
    "tone": "professional",
    "length": "medium",
    "structure": [
      "summary",
      "depth levels",
      "boundaries"
    ]
  },
  "quality_bar": [
    "No scope inflation.",
    "No unsupported certainty."
  ],
  "compiled_instruction": "Act as an AI product education assistant..."
}
```

## 10. Example: hybrid_prompt_package

```json
{
  "schema_version": "delivery-v1",
  "artifact_type": "hybrid_prompt_package",
  "delivery_target": "downstream_model",
  "instruction_precedence": [
    "constraints",
    "output_spec",
    "objective",
    "context",
    "input_data"
  ],
  "objective": "Audit the prompt for hallucination risk and missing boundaries.",
  "context": "The prompt will be reused by another model.",
  "input_data": "Prompt: You are a top expert. Tell me everything about RAG.",
  "constraints": [
    "Return the audit in simplified Chinese.",
    "Flag boundary and hallucination risks explicitly."
  ],
  "assumptions": [],
  "role": {
    "identity": "prompt auditor",
    "stance": "critical and evidence-aware"
  },
  "execution_steps": [
    "Inspect role phrasing.",
    "Inspect scope and missing structure.",
    "Report execution risks."
  ],
  "output_spec": {
    "language": "simplified Chinese",
    "format": "markdown",
    "tone": "direct",
    "length": "medium",
    "structure": [
      "what is good",
      "what is not good",
      "how to revise"
    ]
  },
  "quality_bar": [
    "No decorative praise.",
    "No hidden scope expansion."
  ],
  "compiled_instruction": "Audit the provided prompt for ambiguity, hallucination risk, and missing boundaries..."
}
```

## 11. Review triggers for delivery artifacts

Use these delivery-artifact triggers during review when `artifact_type != final_answer`:

- `missing_delivery_required_field`
- `conflicting_delivery_fields`
- `over_verbose_delivery_payload`
- `ambiguous_instruction_priority`
- `invalid_json`
- `schema_drift`
- `constraint_loss_detected`
- `delivery_payload_too_large`
- `output_spec_ambiguous`
- `human_commentary_leakage`

## 12. Maintenance notes

If the internal schema evolves, update the delivery mapping here rather than copying long delivery rules back into `SKILL.md`.
Keep `SKILL.md` focused on workflow and governance, and keep this file focused on export behavior.
