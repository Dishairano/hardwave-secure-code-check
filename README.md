# Hardwave Secure Code Check

The shared Secure Code Check for Hardwave's own software: the security checks every product runs on each
push, pull request and release, and the signed per-release report built from them. It holds the checker
only, no product code.

Moved here on 26 Nov 2026 from `Dishairano/hardwave-daw` (commit f99298ed), unchanged apart from the
repository name, so products no longer depend on the DAW repository.

## Use

Call it pinned to a commit, and pass the same commit as `tools-ref`, so the scripts that write the report
are the ones the workflow was tested with:

```yaml
jobs:
  secure-code-check:
    uses: Dishairano/hardwave-secure-code-check/.github/workflows/secure-code-check.yml@<sha>
    with:
      tools-ref: <sha>
    secrets:
      SECURE_CODE_CHECK_SIGNING_KEY: ${{ secrets.SECURE_CODE_CHECK_SIGNING_KEY }}

  evidence:
    needs: [secure-code-check, release]
    permissions:
      contents: write
    uses: Dishairano/hardwave-secure-code-check/.github/workflows/release-evidence.yml@<sha>
    with:
      tools-ref: <sha>
      tag: ${{ needs.release.outputs.tag }}
      needs-json: ${{ toJSON(needs) }}
    secrets:
      SECURE_CODE_CHECK_SIGNING_KEY: ${{ secrets.SECURE_CODE_CHECK_SIGNING_KEY }}
```

A calling repository with a restricted Actions allow list needs
`Dishairano/hardwave-secure-code-check/.github/workflows/*` on it.

Each product keeps its own `deny.toml`, `security/secure-code-check/config.toml` and its public signing keys
(`signing_key.pub`, `allowed_signers`) in its own repository; this repository has no keys.

The report scripts' own tests: `python3 -m unittest discover -s scripts/secure-code-check -p 'test_*.py'`.
