# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [v1.1.3] - 2026-11-24

### Fixed
- `container-build.yml`: dropped `--skip-java-db-update` from the Trivy
  gate one release after introducing it — trivy hard-fails the very
  first image scan on a cold cache when the flag is set ("the first run
  cannot skip downloading the Java DB"), and every hosted runner is
  cold, so the gate never got as far as scanning. The buildx prune
  merged in v1.1.2 is what actually frees the runner disk the scan
  needs; the Java DB now downloads on every scan, as it must.

## [v1.1.2] - 2026-11-23

### Fixed
- `container-build.yml`: the Trivy gate intermittently failed on pushed
  refs because hosted runners run out of disk mid-scan — the layer cache
  rides on `cache-to: type=gha`, so the local buildx cache left after the
  push is dead weight that trivy's DB downloads land on top of; the scan
  step now prunes the local buildx cache first, and a red scan again means
  a real finding rather than runner disk exhaustion.
- `container-build.yml`: ~~the Trivy scan now passes
  `--skip-java-db-update`~~ — reverted in v1.1.3: the flag makes trivy
  hard-fail the first scan on a cold runner, which is every hosted
  runner; see v1.1.3 for the full story. The prune above is the part
  of this release that actually fixed the disk exhaustion.

## [v1.1.1] - 2026-11-23

### Fixed
- `release-precheck.yml`: the tag-signature gate failed on every
  tag-triggered run because `actions/checkout` rewrites the triggering
  tag ref to the peeled commit, leaving `git tag -v` with a plain
  commit object ("cannot verify a non-tag object"); the ref is now
  force re-fetched from origin so the check verifies the annotated,
  signed tag object.

## [v1.1.0] - 2026-11-22

### Added
- `container-build.yml`: reusable canonical GHCR image build — lowercase
  name normalization, GHA layer cache, provenance + SBOM attestations,
  and a severity-gated Trivy scan (default CRITICAL) on pushed refs.
- `release-precheck.yml`: reusable production-release gate — strict
  SemVer validation, CHANGELOG section check, SSH tag-signature
  verification against the caller's `.allowed_signers`, and ancestry
  proof that the tag descends from the production branch.

## [v1.0.0] - 2026-11-19

### Added
- `mirror.yml`: reusable full-ref push mirror to a bare git-server repository
  over a dedicated push-only, forced-command-pinned SSH key.
- `ots.yml`: reusable daily OpenTimestamps upgrade + verify with
  dual-oracle (bitcoinrpc.xyz + GetBlock) block-header cross-check; single
  oracle outage degrades to warning, divergence is a hard failure.
- `deploy.yml`: reusable digest-pinned rootless quadlet deploy delegating
  capture/rollback/self-check to the target host's `deploy-service`
  forced-command wrapper.
- Public zero-secrets repository baseline: README (EN/RU), SECURITY.md with
  OpenTimestamps verification instructions, AGENTS.md project rules,
  `.allowed_signers`, `.githooks/`, and bilingual `docs/` layout.
- Signed annotated tag `v1` as the stable pin for all callers.