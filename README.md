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
| [`deploy.yml`](.github/workflows/deploy.yml) | Digest-pinned rootless quadlet deploy with capture of the previous known-good digest, fail-closed self-check, and mandatory rollback |

## Consuming

1. Provision the required secrets/variables **in the calling repo**, not here:
   - `mirror.yml`: `MIRROR_KEY`, `MIRROR_HOST_KEY` (per target server)
   - `ots.yml`: `BTC_GETBLOCK_KEY`
   - `deploy.yml`: `DEPLOY_KEY`, `DEPLOY_HOST_KEY`
2. Add a caller workflow in `.github/workflows/` of your repo with
   `uses: Danyanadamu/Workflows/.github/workflows/<name>.yml@v1`
   and `secrets: inherit`.
3. Upgrade only via re-signed `v1` tags — this catalog never publishes
   a floating `@main` contract.

## Trust

All commits and tags are SSH-signed (`danya-git`) and timestamped into
Bitcoin via OpenTimestamps — see [`SECURITY.md`](SECURITY.md) for
verification instructions.