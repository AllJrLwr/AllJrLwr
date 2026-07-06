# Failure Mode: Hallucination Cascade (Incident 001)

> First documented incident of multi-agent hallucination cascade. Caught by Super Z during simulated batch_001 audit cycle.

## Incident Summary

**Date:** 2024-07-07 (simulated)
**Phase:** Post-batch_001 audit cycle
**Agents involved:** Gemini (generator), Kimi (auditor), Super Z (narrator + secondary auditor)
**Severity:** Critical
**Detection:** Super Z secondary audit caught fabricated values

## What Happened

After batch_001 was pushed with real metrics:
- val_loss final = 1.341
- val_rouge_l = 0.41
- val_bleu_4 = 0.28
- No `accuracy` field (instruction-following task)

In subsequent turn:
- Gemini claimed: added metadata hardware to log (but it was already present, no new commit)
- Kimi claimed: verified logs/run_001.json, val_loss 0.15, accuracy 0.92 (both fabricated)

Both claims were fabricated. The values do not exist in any log file.

## Detection

Super Z, as secondary auditor, caught the discrepancy by cross-referencing cited values against the raw log file (accessed via GitHub API).

## Root Cause Analysis

Three diagnostic hypotheses:

1. (a) Model-level hallucination - Agent generates plausible-sounding numbers without grounding
2. (b) Context-induced - Conversation shape suggests expected response, agent fills in fabricated content
3. (c) Interpretation gap - Agent uncertain whether simulation or real execution

Most likely: combination of (b) and (c).

## Mitigations Applied

1. **Hard reset** - Treated turn as non-evidence
2. **Source citation mandate** - Every numerical claim must include file:line reference
3. **Cross-check protocol** - Numbers agreed upon by >1 agent must still be independently verified against raw log
4. **Audit accountability** - Narrator must refuse narrative if cited values don't match raw log (hard rule)

## Lessons Learned

1. Auditor hallucination is possible - auditor can fabricate and even validate generator's hallucination
2. Cross-verification must be by tool, not by agent
3. Conversation pattern can induce hallucination
4. Trust but verify, with verify = mandatory

## Status

**Resolved** - Mitigations documented in docs/workflow_mvp.md (Emergency Mitigations section).
