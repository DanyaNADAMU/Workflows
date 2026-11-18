# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in this project, please send an
email to git@nada.mu.

## Commit Signatures & Verification

All official commits and releases in this repository are cryptographically
signed using SSH keys and timestamped into the Bitcoin blockchain via
OpenTimestamps.

To verify signatures locally:

```bash
git config gpg.ssh.allowedSignersFile .allowed_signers
git log --show-signature
git tag -v v1
```

## Zero-Secrets Design

This repository is **public by design** — GitHub reusable workflows can only
be consumed from a public repository. It therefore deliberately contains
**no secrets of any kind**: no tokens, no private keys, no host inventories.
Every credential referenced by a workflow is documented by *name only* and
must be provisioned in the calling repository's own secrets. A leak of this
repository grants nothing but the workflows' source code.

Official Project Domain: https://nada.mu