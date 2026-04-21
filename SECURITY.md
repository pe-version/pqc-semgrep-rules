# Security Policy

## Reporting a vulnerability

If you discover a security vulnerability in `pqc-semgrep-rules` (a false-negative that lets a risky algorithm slip past, or a false-positive pattern that produces unsafe migration advice), please report it through [GitHub's private vulnerability reporting](https://github.com/pe-version/pqc-semgrep-rules/security/advisories/new) rather than opening a public issue.

Reports will be acknowledged within seven days where possible.

## Scope

These rules are an *advisory inventory aid*, not a complete cryptographic-bill-of-materials. They will miss algorithms constructed via configuration, dependency-injection factories, or runtime-loaded provider chains. Use them as a fast first pass, not as a compliance attestation.

## Supported versions

| Version | Supported |
| --- | --- |
| 0.1.x | ✓ |
| < 0.1 | ✗ |
