# Workflows — Project Agent Rules

## What This Project Is

`Danyanadamu/Workflows` is the **public, single source of truth for reusable
GitHub Actions workflows** shared by every repository in the organization:
`mirror.yml` (bare git-server mirroring), `ots.yml` (daily OpenTimestamps
anchoring with dual-oracle cross-check), `deploy.yml` (digest-pinned
rootless quadlet deploys with mandatory rollback), `container-build.yml`
(canonical GHCR image build with cache, attestations, and Trivy scan), and
`release-precheck.yml` (signed-SemVer/CHANGELOG/ancestry release gate).
Callers pin these workflows with a signed annotated tag (`@v1`); nothing in
this repo is ever called by branch.

## Non-Negotiables

- **Zero secrets, ever**: this repository is public by design; every key and
  credential lives in the *calling* repository's own secrets
  (`secrets: inherit`), never here. The right to act lives where the act
  happens, and not one step further.
- **Pinned by signed tag only**: new workflow changes must be validated by
  running workflows on `main` first, then released by re-signing `v1`
  (annotated, signed). Never expose callers to untested `@main`.
- **Public is a release surface**: anything visible here is treated as
  world-readable documentation of our infrastructure; comments must never
  leak host topology that Infra does not already publish.

## Module Boundaries

- `mirror.yml` — pushes a full-ref mirror to a bare git repo over a
  push-only, forced-command-pinned SSH key. Reads nothing but git refs.
- `ots.yml` — upgrades and verifies OTS proofs in the *caller's* checkout,
  pushes back to the caller's `main`, cross-checks attested block headers
  across two independent RPC providers.
- `deploy.yml` — resolves an image to a digest, then hands exactly one
  argument to the target host's forced-command wrapper
  (`deploy-service`), which owns capture/rollback/self-check logic.
- `container-build.yml` — single canonical image build: lowercase GHCR
  name normalization, GHA layer cache, provenance/SBOM attestations, and
  a Trivy severity gate on pushed refs; returns `image_ref` to the caller
  for `deploy.yml`.
- `release-precheck.yml` — pure verification of the triggering tag
  (strict SemVer, CHANGELOG section, SSH signature against the caller's
  `.allowed_signers`, ancestry from the production branch); it performs
  no side effects beyond reading the caller's checkout.
- No workflow here may import another; each is a self-contained unit that
  receives all inputs via `workflow_call`.

## Conventions

- Commits: Conventional Commits, English, SSH-signed (`danya-git`).
- Docs: bilingual `docs/en/` + `docs/ru/` for anything user-facing.
- Tags: annotated + signed (`git tag -a`), SemVer for the repo itself;
  callers consume only `v1`-style stable major tags.
- CI changes are validated on `test` callers before a `v1` re-sign reaches
  any production caller.