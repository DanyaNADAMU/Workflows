# Trust Model of the Workflow Catalog

## Zero secrets on purpose

This catalog is public because GitHub reusable workflows can only be
consumed from a public repository (or the same one). Because public means
world-readable, the repository is designed so that a full leak is worthless:

- No tokens, no private keys, no host inventories are ever committed here.
- Every credential a workflow needs is declared by **name** in its
  `workflow_call` interface and must be provisioned in the **calling**
  repository's own secrets, entering via `secrets: inherit`.

## The right to act lives where the act happens

Each workflow is built around the principle that a credential never travels
further than the machine where it is used:

- **Mirror push key** — exists only in the caller's secrets; the matching
  public key on the git server is pinned to a forced command
  (`git-receive-pack '/home/git/<Repo>.git'`), so it can never read
  anything or reach a shell.
- **Deploy key** — the service user's own key; the public side is pinned to
  `deploy-service <service> <digest>` on the target host, so a leaked key
  can only ever roll one service to one image digest.
- **GetBlock key** — used only by the cross-check step of `ots.yml`, inside
  the caller's job, never materialized here.

## Verification gates

- Commits and tags are SSH-signed (`danya-git`); `.allowed_signers` is the
  trust anchor, OpenTimestamps proofs anchor the history into Bitcoin.
- Callers pin `@v1`, a signed annotated tag; a `v1` re-sign is the only way
  a workflow change can reach any caller.
- `git tag -v` is mandatory on every mirror checkout before a deploy: the
  operator verifies the signature chain before trusting a release.