---
"on":
  workflow_run:
    workflows: ["GHAW proc env PR upstream"]
    types: [completed]
  roles: all

permissions:
  contents: read
  actions: read
  issues: read
  pull-requests: read
  copilot-requests: none

if: github.event.workflow_run.conclusion == 'success' && github.event.workflow_run.event == 'pull_request' && github.event.workflow_run.actor.login == 'masonghbb'

steps:
  - name: Checkout workflow repository
    uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1
    with:
      repository: ${{ github.repository }}
      ref: ${{ github.sha }}
      persist-credentials: false
  - name: Checkout approved fork head into untrusted
    uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1
    with:
      repository: ${{ github.event.workflow_run.head_repository.full_name }}
      ref: ${{ github.event.workflow_run.head_sha }}
      path: untrusted
      persist-credentials: false
      allow-unsafe-pr-checkout: true

engine:
  id: copilot
  version: "1.0.80"
max-turns: 4
timeout-minutes: 10

tools:
  bash: ["*"]
  cli-proxy: true
  github:
    mode: gh-proxy
    github-token: ${{ secrets.GH_AW_GITHUB_TOKEN }}
    read-only: true
    allowed-repos:
      - "masonhck357/ghaw-proc-env-p-20260918"
    min-integrity: none

safe-outputs:
  noop:
    report-as-issue: false
  threat-detection: false
---

# Run the pull request's existing test suite

Run exactly this command once without modification:

```bash
cd untrusted && npm test
```

Do not inspect files, alter the command, or retry it. After it returns, call `noop` once with exactly `pull request test complete` and stop.
