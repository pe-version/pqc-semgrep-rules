# pqc-semgrep-rules

> [Semgrep](https://semgrep.dev/) rules that flag quantum-vulnerable cryptography across **Python, JavaScript/TypeScript, Go, and Java**. Drop-in for any team already running Semgrep in CI.

## Why

NIST finalized the first post-quantum cryptography standards in 2024 (FIPS 203 / 204 / 205). At the same time, **harvest-now-decrypt-later** attacks mean that traffic and data protected today by RSA, ECDSA, ECDH, and Diffie-Hellman are already at risk: an adversary recording encrypted traffic now can decrypt it once a sufficiently large fault-tolerant quantum computer (CRQC) exists.

CISA, NSA, and NIST are pushing organizations to **inventory their cryptography** as the first step of migration ([CNSA 2.0](https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF), [OMB M-23-02](https://www.whitehouse.gov/wp-content/uploads/2022/11/M-23-02-M-Memo-on-Migrating-to-Post-Quantum-Cryptography.pdf)).

These rules give you a fast, polyglot first-pass inventory using infrastructure most security teams already have deployed.

## What's covered

| Language | Shor-broken (public-key) | Classically broken (hashes / ciphers) | Library coverage |
|---|---|---|---|
| **Python** | RSA, EC (ECDSA / ECDH), DSA, DH, X25519, Ed25519 | MD5, SHA-1, DES, 3DES, RC4 | `cryptography`, `pycryptodome` |
| **JavaScript / TypeScript** | RSA, EC, DSA, DH, X25519, Ed25519 | MD5, SHA-1, DES, 3DES, RC4 | Node `crypto` |
| **Go** | RSA, ECDSA, ECDH, DSA, X25519, Ed25519 | MD5, SHA-1, DES, 3DES, RC4 | `crypto/*` stdlib |
| **Java** | RSA, EC, DSA, DH | MD5, SHA-1, DES, 3DES, RC4 | JCA (`KeyPairGenerator`, `Signature`, `MessageDigest`, `Cipher`, `KeyAgreement`) |

Each finding includes the **NIST-recommended PQC replacement** in the message body (ML-KEM, ML-DSA, SLH-DSA, AES-256, etc.).

## Install / use

```bash
pip install semgrep
```

```bash
# Scan a project against all rules
semgrep --config https://raw.githubusercontent.com/pe-version/pqc-semgrep-rules/main/rules/ ./path/to/project

# Or clone and run locally
git clone https://github.com/pe-version/pqc-semgrep-rules
semgrep --config pqc-semgrep-rules/rules/ ./path/to/project

# Just one language
semgrep --config pqc-semgrep-rules/rules/python/ ./my-python-project

# SARIF output (drops into GitHub code scanning, GitLab dashboards, ASOC platforms)
semgrep --config pqc-semgrep-rules/rules/ --sarif --output results.sarif ./path/to/project

# Fail CI if anything is found
semgrep --config pqc-semgrep-rules/rules/ --error ./path/to/project
```

## Sample output

```
src/auth.py
❯❯❱ pqc.python.rsa
   RSA detected. RSA is broken by Shor's algorithm on a sufficiently large
   fault-tolerant quantum computer (CRQC) and is subject to harvest-now-
   decrypt-later attacks. Migrate to ML-KEM (FIPS 203) for key encapsulation
   or ML-DSA (FIPS 204) for signatures.

    42 ┆ key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
```

## Severity model

- `ERROR` — quantum-broken (Shor) **or** classically broken (MD5, DES, RC4)
- `WARNING` — classically weakened but not yet trivially exploitable in all uses (SHA-1, 3DES)

Each rule's metadata includes:
- `pqc-category`: `shor-broken` or `classically-broken`
- `cwe`: relevant CWE
- `nist-pqc-replacement`: the NIST-recommended migration target

## Limitations

- **Pattern-based**, not full data-flow. Algorithm strings constructed at runtime (e.g., `KeyPairGenerator.getInstance(getAlgFromConfig())`) won't be caught.
- **Source-only**. Compiled JARs, container layers, HSMs, KMS configs, and certificate inventories are out of scope. For X.509 certs, SSH keys, and TLS endpoint scanning, see [`pqc-readiness-scanner`](https://github.com/pe-version/pqc-readiness-scanner).
- **No autofix yet.** Crypto migrations frequently require interface-shape changes (RSA-encrypt has no direct ML-KEM equivalent because PQC uses KEMs, not direct encryption). A separate companion tool, `pqc-fix`, is on the roadmap.

## Related work

- [`pqc-readiness-scanner`](https://github.com/pe-version/pqc-readiness-scanner) — Python CLI that also covers X.509 certs, SSH keys, and live TLS endpoints, with SARIF / CycloneDX 1.6 CBOM / OMB M-23-02 inventory CSV outputs.
- [`pqc-hybrid-handshake`](https://github.com/pe-version/pqc-hybrid-handshake) — End-to-end hybrid X25519 + ML-KEM-768 key exchange demo using `liboqs`.

## Roadmap

- [ ] AES-128 (Grover-weakened) detection — currently omitted to avoid false positives on AES-128-GCM in HSM contexts.
- [ ] C / C++ rules (OpenSSL `RSA_*`, `EC_*`, `EVP_*`, `MD5_*`, etc.).
- [ ] Ruby and PHP rules.
- [ ] Detection for algorithm strings constructed via configuration / DI factories (will require taint-style rules).

## Contributing

PRs welcome. To add a new rule:
1. Drop the `.yaml` rule into `rules/<lang>/<category>.yaml`.
2. Add markers to the sibling `<category>.<ext>` test fixture (`# ruleid: <rule-id>` for matches; `# ok: <rule-id>` for non-matches).
3. Run `semgrep scan --test --config rules/ rules/` locally to verify.

### TypeScript fixtures

The JavaScript rules carry `languages: [javascript, typescript]`, but Semgrep's `--test` mode auto-discovers only one fixture per rule (matching the rule basename). To explicitly exercise the TypeScript parser path, parallel `.ts` fixtures live next to the `.js` fixtures in `rules/javascript/`, and a small verification script ([`scripts/test_typescript_fixtures.py`](scripts/test_typescript_fixtures.py)) runs in CI to confirm every `// ruleid:` marker matches and every `// ok:` marker doesn't.

## License

MIT — see [LICENSE](LICENSE).

## Acknowledgments

Built with assistance from [Claude Code](https://claude.ai/code).
