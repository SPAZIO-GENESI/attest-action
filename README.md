# attest-action

GitHub Action that attests build artifacts with [Spazio Genesi](https://attestazione.spaziogenesi.org)'s
free proof-of-existence service, using the [`sg-attest`](https://github.com/SPAZIO-GENESI/attest-mcp#cli-sg-attest)
CLI (`@spazio-genesi/attest-mcp`).

**Full privacy**: the artifact's SHA-256 fingerprint is computed on the runner
(streamed from disk) — the artifact itself never leaves the runner. Only the
fingerprint and any metadata you declare (title/author/year/note) are sent.

## Usage

```yaml
- name: Attest the release artifact
  uses: SPAZIO-GENESI/attest-action@v1
  with:
    files: dist/release.zip
    api-key: ${{ secrets.SG_API_KEY }}
    title: "My Project v${{ github.ref_name }}"
```

Every attestation prints a fingerprint + verification link summary to the
job's `$GITHUB_STEP_SUMMARY`, and exposes them as outputs:

```yaml
- name: Attest
  id: attest
  uses: SPAZIO-GENESI/attest-action@v1
  with:
    files: dist/*.tar.gz
    api-key: ${{ secrets.SG_API_KEY }}
- run: echo "Certificate: ${{ steps.attest.outputs.certificate-url }}"
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `files` | yes | — | One or more paths/globs, whitespace- or newline-separated. |
| `api-key` | yes | — | An `sg_k_…` key. Pass it as a secret, never a literal value. |
| `title` / `author` / `year` / `note` | no | — | Declared metadata, bound to the attestation's signature. |
| `download-pdf` | no | `false` | Also mint the signed certificate PDF (saved as `<file>.certificato.pdf`). |
| `cli-version` | no | `0.3.1` | Pinned CLI version — never `latest` (a floating version could change behavior between runs without you noticing). |

## Outputs

| Output | Description |
|---|---|
| `sha256` | Fingerprint of the last attested file. |
| `certificate-url` | Its permanent verification URL (`/c/<hash>`). |

With multiple `files`, every file is attested and listed in the step
summary; the outputs reflect only the last one processed.

## Getting an API key

Self-service, free: [`attestazione.spaziogenesi.org/developer/keys/`](https://attestazione.spaziogenesi.org/developer/keys/)
(sign in with Google/Microsoft/LinkedIn, no credit card). 50 attestations/month
on the free tier. Store the key as a repository or organization secret
(`SG_API_KEY`) — never commit it, never pass it as a plain `with:` value.

## Cost of running this

Every run of this action consumes one unit of your key's monthly quota per
file attested. Don't wire it into workflows that run on every push if you
don't mean to spend quota that often — a release/tag-triggered workflow is
the common case.

## Why this, not a raw curl call

The CLI (and this action) hash the file locally and never upload it — the
service's `/api/hash` contract takes only a SHA-256 digest, timestamped and
signed server-side. A hand-rolled `curl` would need to compute that digest
correctly and handle the credential/error contract itself; this action does
that once, pinned and tested, so your workflow doesn't have to.

## Security

See [Spazio Genesi's responsible disclosure policy](https://attestazione.spaziogenesi.org/sicurezza/).
This repo has no runtime secrets of its own — it is a thin wrapper around the
pinned `sg-attest` CLI.

## License

MIT — see [LICENSE](LICENSE). Same license as [attest-mcp](https://github.com/SPAZIO-GENESI/attest-mcp),
which this action wraps; the attestation service itself
([imgauth](https://github.com/SPAZIO-GENESI/imgauth)) is AGPL-3.0.
