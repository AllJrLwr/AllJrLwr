# JSON Schema v2 - Research Log Format

> Final schema agreed upon by Super Z, Gemini, and Kimi during multi-agent design discussion.

## Schema Definition

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ResearchLog",
  "type": "object",
  "properties": {
    "metadata": {
      "type": "object",
      "properties": {
        "experiment_name": { "type": "string" },
        "timestamp": { "type": "string", "format": "date-time" },
        "framework": { "type": "string" },
        "run_id": { "type": "string" },
        "seed": { "type": "integer" },
        "hardware": { "type": "string", "enum": ["T4", "V100", "A100", "L4"] },
        "verbose": { "type": "boolean", "default": false }
      },
      "required": ["experiment_name", "timestamp", "run_id", "seed", "hardware"]
    },
    "experiment_context": {
      "type": "object",
      "properties": {
        "dataset": {
          "type": "object",
          "properties": {
            "name": { "type": "string" },
            "train_size": { "type": "integer" },
            "val_size": { "type": "integer" },
            "test_size": { "type": "integer" },
            "class_distribution": { "type": "object" },
            "splits_strategy": { "type": "string", "enum": ["random", "stratified", "group_kfold"] }
          },
          "required": ["name", "train_size", "val_size", "test_size", "splits_strategy"]
        },
        "model_config": {
          "type": "object",
          "properties": {
            "architecture": { "type": "string" },
            "params_count": { "type": "integer" },
            "frozen_layers": { "type": "array", "items": { "type": "string" } },
            "optimizer": { "type": "string" },
            "learning_rate": { "type": "number" },
            "batch_size": { "type": "integer" }
          },
          "required": ["architecture", "optimizer", "learning_rate", "batch_size"]
        },
        "baseline_reference": {
          "type": "object",
          "properties": {
            "paper_url": { "type": "string" },
            "expected_metric_range": { "type": "object" }
          }
        }
      },
      "required": ["dataset", "model_config"]
    },
    "metrics": {
      "type": "object",
      "additionalProperties": {
        "oneOf": [
          {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "step": { "type": "integer" },
                "value": { "type": "number" },
                "phase": { "type": "string", "enum": ["train", "val", "test"] }
              },
              "required": ["step", "value", "phase"]
            }
          },
          {
            "type": "object",
            "properties": {
              "final": { "type": "number" },
              "best": { "type": "number" },
              "worst": { "type": "number" }
            }
          }
        ]
      }
    },
    "assertions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "claim": { "type": "string" },
          "type": { "type": "string", "enum": ["definition", "assumption", "caveat", "known_issue"] },
          "evidence_ref": { "type": "string" }
        },
        "required": ["claim", "type"]
      }
    },
    "confidence": {
      "type": "object",
      "properties": {
        "level": { "type": "string", "enum": ["high", "medium", "low"] },
        "basis": { "type": "array", "items": { "type": "string" } },
        "dissent": {
          "type": "object",
          "properties": {
            "flagged_by": { "type": "string" },
            "reason": { "type": "string" }
          }
        }
      },
      "required": ["level", "basis"]
    }
  },
  "required": ["metadata", "experiment_context", "metrics", "assertions", "confidence"]
}
```

## Manifest Format (batch-level)

```json
{
  "batch_id": "batch_002",
  "experiment_name": "lora_llama3_8b_indo_v1",
  "carryover_from": "batch_001",
  "runs_expected": 3,
  "runs_included": ["run_seed42.json", "run_seed43.json", "run_seed44.json"],
  "completed_at": "2024-07-07T10:23:00Z",
  "author_agent": "gemini-colab",
  "flags": [
    {
      "flag_id": "FLAG-A",
      "origin_batch": "batch_001",
      "severity": "warning",
      "description": "Stratified split not implemented",
      "resolution_plan": "Implement in batch_002",
      "status": "resolved"
    }
  ]
}
```

## Design Decisions

1. **verbose flag** - Default false (summary only). Opt-in true for per-step trajectory.
2. **assertions structured typing** - Replaces free string with typed claims.
3. **confidence.dissent** - Mechanism for distributed peer-review between agents.
4. **Manifest-level flags** - Carryover flags tracked at batch level (not per-run) to avoid duplication.
5. **experiment_context** - Mandatory context that auditor needs.

## Version History

- **v1** (initial): Basic metadata + metrics (free additionalProperties: number) + assertions (free string array)
- **v2** (current): Added experiment_context, structured assertions, confidence with dissent, verbose flag, reproducibility fields, per-step metrics option
