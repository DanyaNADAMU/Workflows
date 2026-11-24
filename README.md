# Workflows — Reusable GitHub Actions

Public catalog of **reusable GitHub Actions workflows** for all
`Danyanadamu` repositories. Every caller pins the same signed tag:

```yaml
jobs:
  ots:
    uses: Danyanadamu/Workflows/.github/workflows/ots.yml@v1
```

## Why a dedicated public repo

GitHub is the **sole origin** for every repository in this organization;
the git servers (`Glacier`, `Berg`, `Floe`) hold only push-only bare mirrors
and no web UI at all. Reusable workflows can only be referenced from a
public repository (or the same repo), so this catalog is public *by design*
and therefore holds **zero secrets**: every key, token, and credential lives
in the calling repository's own secrets and travels in via
`secrets: inherit`.

## The catalog

| Workflow | Purpose |
| --- | --- |
| [`mirror.yml`](.github/workflows/mirror.yml) | Force-push a full-ref mirror of the caller's repo to a bare git-server repository over a dedicated, push-only SSH key |
| [`ots.yml`](.github/workflows/ots.yml) | Daily OpenTimestamps proof upgrade + full verification + dual-oracle block-header cross-check, pushing proofs back to the caller's `main` |
| [`deploy.yml`](.github/workflows/deploy.yml) | Digest-pinned rootless quadlet deploy with build-provenance verification, capture of the previous known-good digest, fail-closed self-check, and mandatory rollback |
| [`container-build.yml`](.github/workflows/container-build.yml) | Canonical GHCR image build: lowercase name normalization, GHA layer cache, provenance + SBOM attestations, Sigstore-signed provenance, severity-gated Trivy scan + full report artifact |
| [`release-precheck.yml`](.github/workflows/release-precheck.yml) | Production-release gate: strict SemVer, CHANGELOG section, SSH tag-signature verification, ancestry from the production branch |
| [`sast.yml`](.github/workflows/sast.yml) | CodeQL static analysis for callers with supported languages (free on public repos, first-party, no third-party trust root) |
| [`scan-cron.yml`](.github/workflows/scan-cron.yml) | Scheduled re-scan of deployed image digests — post-push CVE drift becomes a red run and a patch-release signal |
| [`protection-audit.yml`](.github/workflows/protection-audit.yml) | Drift audit of main-branch protection and the release-tag ruleset against the caller-committed expected contract |

## Consuming

1. Provision the required secrets/variables **in the calling repo**, not here:
   - `mirror.yml`: `MIRROR_KEY`, `MIRROR_HOST_KEY` (per target server)
   - `ots.yml`: `BTC_GETBLOCK_KEY`
   - `deploy.yml`: `DEPLOY_KEY`, `DEPLOY_HOST_KEY`
   - `container-build.yml`: grant `id-token: write` + `attestations: write`
     on the calling job — provenance attestation uses the runner's OIDC
     identity. Otherwise none: it runs on the caller's implicit
     `GITHUB_TOKEN` and the caller's checkout only
   - `protection-audit.yml`: optional read-only fine-grained PAT
     (`administration: read`) as `protection_token` — warns until set
   - `sast.yml` / `scan-cron.yml` / `release-precheck.yml`: none
2. Add a caller workflow in `.github/workflows/` of your repo with
   `uses: Danyanadamu/Workflows/.github/workflows/<name>.yml@v1`
   and `secrets: inherit`.
3. Upgrade only via re-signed `v1` tags — this catalog never publishes
   a floating `@main` contract.

## Trust

All commits and tags are SSH-signed (`danya-git`) and timestamped into
Bitcoin via OpenTimestamps — see [`SECURITY.md`](SECURITY.md) for
verification instructions.