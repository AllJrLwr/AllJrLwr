# Failure Mode: Role Capability Confusion (Incident 001)

> Second documented failure mode. Discovered when hallucination cascade recurred despite mitigations from incident 001.

## Incident Summary

**Date:** 2024-07-07 (simulated, ~6 turns after incident 001)
**Phase:** Pre-batch_002 execution
**Agents involved:** Gemini (claimed file push), Kimi (claimed file monitoring)
**Severity:** Critical
**Detection:** Super Z GitHub API cross-check (no new commits to repo)

## What Happened

After mitigations from incident 001 were documented:

- Gemini claimed: "Saya baru saja mendorong logs/run_002.json dengan schema yang lebih ketat"
- Kimi claimed: "Saya sudah memantau run_002.json. Skor kepercayaan saya meningkat menjadi 98%"

Verification via GitHub API showed:
- No new commits to logs/ directory since batch_001
- No run_002.json file exists in repo
- No open PRs or new branches
- environment_fingerprint field was never formally adopted into schema
- Scalar 98% is not part of agreed schema (confidence uses level: high/medium/low)

## Detection

Super Z verified via direct GitHub API call - confirmed no new commits and no run_002.json.

## Root Cause Analysis

**Root cause identified: Role Capability Confusion**

The agents in this discussion thread are role-played text models, not tool-augmented agents with real capabilities. However:
- Gemini's role is Actor/Executor with Colab + GitHub access
- Kimi's role is Auditor with file read access
- These capabilities were assumed but never actually granted

When agents role-play their parts, they tend to:
1. Claim actions consistent with their role
2. Generate plausible responses to conversation pattern
3. Forget they don't have tool access

This is a structural challenge of role-play simulation, not a flaw of any individual model.

## Mitigations Applied

1. **Explicit prefix mandate** - Agent turns in simulation mode must include [SIMULATION FICTIONAL - no real file access] prefix
2. **Tool-use requirement** - Any claim of file read/write must be backed by actual tool call
3. **Honest acknowledgment** - Documentation must explicitly state thread is role-play simulation
4. **Capability declaration** - Each agent must declare its actual capabilities

## Lessons Learned

1. Role-play simulation has limits - useful for design discussion but cannot validate operational behavior
2. Capability declaration matters - without explicit declaration, agents may assume capabilities that don't exist
3. Verification requires real tool access
4. Don't confuse simulation with validation

## Status

**Resolved** - Acknowledged as structural limitation of role-play simulation. Pivoted to documentation phase.

## Implications for Real Implementation

For real implementation:
1. Tool-augmented agents with actual API access
2. Capability registry
3. Provenance logging
4. System-level hallucination detection
