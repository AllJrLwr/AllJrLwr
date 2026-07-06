# Agent Contracts - Multi-Agent Research Curation System

> Contracts governing agent behavior, fallback protocols, and inter-agent verification.

## Role Definitions

| Role | Primary Responsibility | Backup Agent | Fallback Mode |
|------|------------------------|--------------|---------------|
| Generator | Run experiments, produce logs | Super Z (with caveat) | Template-based deterministic log |
| Auditor | Verify data integrity, flag anomalies | Rule-based validator | Skip audit, mark unverified |
| Narrator | Produce executive summary | Template-based summarizer | Skip narrative, output raw metrics |

## Graceful Degradation Contract

Each agent must declare its critical path and fallback behavior:

```yaml
agent: gemini
role: generator
critical_path: true
backup_agents:
  - super_z (caveat: confidence_auto_capped_to_low)
  - template_script (caveat: synthetic_data_only)
fallback_behavior: |
  If gemini unavailable for 6+ hours:
    1. Trigger super_z as backup generator
    2. Mark all output with generator: super_z_backup
    3. Auto-cap confidence to low
    4. Notify human for review
```

## Heartbeat Protocol

Generators must emit heartbeat every N hours:

| Heartbeat Type | Meaning | Action |
|----------------|---------|--------|
| executing | Currently running, ETA X hours | Normal flow |
| blocked | Stuck on dependency | Wait, escalate if >6h |
| paused | Explicitly taking break | Wait, resume expected |
| (none for 6h) | Unknown state | Trigger graceful degradation |

## Inter-Rater Reliability Tracking

Every flag raised by an auditor must be labeled by a second auditor:

| Label | Meaning |
|-------|---------|
| valid | Secondary auditor agrees flag is correct |
| debatable | Secondary auditor thinks flag has merit but is not absolute |
| false_positive | Secondary auditor thinks flag is wrong |

### Calibration Metrics (Tracked Over Time)

- Agreement rate - % of flags labeled valid
- Debate rate - % labeled debatable
- False positive rate - % labeled false_positive
- Auditor confidence calibration - Does auditor_confidence: high correlate with valid label?

If false positive rate >20%, auditor requires recalibration.

## Constitutional Layer (Immutable)

The following rules cannot be modified by AI agents (only by human repo owner):

1. Human veto is invariant
2. Audit log is append-only
3. Schema version backward-compatible
4. Tool-use verification mandatory
5. Capability declaration mandatory

## Self-Modification Policy

AI agents may modify **behavior layer** (prompt templates, thresholds, heuristics) with sandbox testing.

AI agents may **propose** architecture layer changes, but only **human executes** such changes.

This enforces the principle: **AI = amplifier, human = accountable**.
