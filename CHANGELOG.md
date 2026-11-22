# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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