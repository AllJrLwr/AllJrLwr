# GitHub Action Workflow MVP - AI Research Analyzer

> Workflow design agreed upon during multi-agent discussion.

## Workflow YAML

```yaml
# .github/workflows/ai-research-analyzer.yml
name: AI Research Log Analyzer

on:
  push:
    paths:
      - 'logs/**/_MANIFEST.json'

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Validate manifest and collect runs
        id: collect
        run: |
          MANIFEST=$(find logs -name '_MANIFEST.json' -newer HEAD~1 | head -1)
          BATCH_DIR=$(dirname "$MANIFEST")
          echo "batch_dir=$BATCH_DIR" >> $GITHUB_OUTPUT
          python scripts/validate_manifest.py "$MANIFEST"

      - name: Aggregate metrics
        run: |
          python scripts/aggregate_runs.py "${{ steps.collect.outputs.batch_dir }}" > /tmp/aggregated.json

      - name: Invoke AI analysis
        id: ai
        uses: ./.github/actions/ai-analyze
        with:
          aggregated-file: /tmp/aggregated.json
          output-format: markdown
          api-key: ${{ secrets.AI_API_KEY }}

      - name: Create draft PR with research notes
        uses: peter-evans/create-pull-request@v6
        with:
          branch: research-notes/${{ steps.collect.outputs.batch_dir }}
          title: "[Auto] Research notes: ${{ steps.collect.outputs.batch_dir }}"
          body: ${{ steps.ai.outputs.markdown }}
          draft: true
          labels: ai-generated, needs-review
```

## Key Design Principles

1. **Trigger only on _MANIFEST.json** - atomic, no false-positives
2. **Validation step** - schema check + integrity
3. **Aggregation step** - normalize multi-run into single payload
4. **AI as composite action** - reusable and swappable
5. **Draft PR** - built-in human review gate (Pattern C)
6. **Auto-label ai-generated + needs-review** - explicit provenance

## Emergency Mitigations

After hallucination cascade incidents, the following mitigations are required:

### 1. Tool-Use Verification (Mandatory)

AI analysis step must read raw log files via tool call, not infer from PR body.

### 2. Divergence Threshold

If AI's narrative cites any metric value that diverges >5% from raw log, workflow must fail and open an issue tagged `audit-failure`.

### 3. Audit Accountability

Narrator agent must refuse narrative if:
- Auditor's flag list contains any critical severity unresolved
- Auditor's cited values don't match raw log
- Auditor is silent (no heartbeat) for >6 hours

## Open Items (Deferred to batch_003+)

- repository_dispatch trigger pattern
- Cost tracking (post-hoc only)
- Delta narrative convention
- Tiered audit (auto-rule-based + full agent + human review)
