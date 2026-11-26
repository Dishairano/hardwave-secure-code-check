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
(`signing_key.pub`, `allowed_signers`, and `revoked_signers` once a key has been replaced) in its own
repository; this repository has no keys. `sign.sh` refuses to sign with a key listed in `revoked_signers`.

## What the public page links

The report and the page do not link the product's source repository, which may be private. What a reader
needs from it is published next to the report, in the release's own folder on
`hardwavestudios.com/security/releases/<product>/<version>/`: `allowed_signers`, `signing_key.pub`,
`revoked_signers`, `deny.toml` and `vulnerability-history.json`, each as it is at that release. They are in
the `<product>-<version>-security-page.tar.gz` the release carries.

The report scripts' own tests: `python3 -m unittest discover -s scripts/secure-code-check -p 'test_*.py'`.
