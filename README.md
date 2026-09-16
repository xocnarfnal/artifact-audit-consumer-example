# Artifact Audit Consumer Example

This repository is an official consumer smoke-test/example for Artifact Audit.
It is maintained by the same project owner and does not represent external adoption.

It validates the public GitHub Action release `v0.2.1` from a repository that is separate from the Artifact Audit source repository.

The workflow checks two synthetic scenarios:

1. **Clean validation:** the committed `artifacts/` bundle is verified against `seal.json` and must pass.
2. **Expected drift validation:** the workflow deliberately changes `artifacts/report.txt`, captures the Action's expected failure, and fails the job only if drift is not detected. The overall workflow therefore remains green when detection works correctly.

The exact Action reference under test is:

```yaml
uses: xocnarfnal/artifact-audit@v0.2.1
```

## Copy-paste workflow

```yaml
name: Artifact Audit consumer validation

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7

      - name: Verify the clean bundle
        uses: xocnarfnal/artifact-audit@v0.2.1
        with:
          path: ./artifacts
          manifest: ./seal.json

      - name: Introduce synthetic drift
        shell: bash
        run: printf '\nDeliberate synthetic drift.\n' >> artifacts/report.txt

      - name: Verify the drifted bundle
        id: expected-drift
        continue-on-error: true
        uses: xocnarfnal/artifact-audit@v0.2.1
        with:
          path: ./artifacts
          manifest: ./seal.json

      - name: Assert that drift was detected
        shell: bash
        env:
          ACTION_OUTCOME: ${{ steps.expected-drift.outcome }}
        run: |
          if [ "$ACTION_OUTCOME" != "failure" ]; then
            echo "::error::Expected drift was not detected."
            exit 1
          fi
```

## Project links

- [Artifact Audit repository](https://github.com/xocnarfnal/artifact-audit)
- [Artifact Audit on GitHub Marketplace](https://github.com/marketplace/actions/artifact-audit)
- [Artifact Audit on PyPI](https://pypi.org/project/artifact-audit/)

This example is maintainer-owned validation. It is not third-party adoption, independent validation, or evidence of community usage.
