# Shengcun

**An offline-first security scanner for code, containers, infrastructure and the supply chain.**

Shengcun reads a repository, container manifest, Kubernetes bundle, IaC tree or model file, and writes
findings as SARIF 2.1.0 or a CycloneDX SBOM. Static scanning performs no network I/O: it reads bytes on
disk and nothing else.

This repository is the public showcase. The source is private; the numbers and coverage below are what
it does, not how it does it. [Request a demo](#request-a-demo) to see it run against your codebase.

---

## What it covers

- **Application code** — vulnerability patterns, dangerous sinks and cross-file data flow, read from a
  parsed AST plus a per-repository call graph rather than from regular expressions.
- **Secrets** — signature families for cloud keys, tokens, private keys and connection strings, plus
  entropy analysis, with optional read-only verification that proves a credential is live without
  changing anything.
- **Infrastructure as code** — Dockerfile, Compose, Kubernetes, Terraform, GitHub Actions and RBAC YAML.
- **Supply chain** — dependency advisories by resolved version, exploit-likelihood scoring, and a
  CycloneDX SBOM using the same coordinates the findings cite.
- **Containers** — image and manifest posture, base-image drift.
- **CI/CD** — workflow permissions, unpinned actions, script injection.
- **Model and agent surfaces** — model-weight deserialization risks, MCP and agent configuration,
  prompt-injection surfaces.
- **Live targets, only with authorization** — external attack-surface discovery, port probing and form
  auditing behind an explicit authorization flag and a scope check that fails closed.

## What it produces

- SARIF 2.1.0, ingestible by GitHub code scanning, Azure DevOps or any SIEM that speaks it.
- CycloneDX 1.5 SBOM.
- JSON and a self-contained HTML report, optionally white-labelled per client.
- Exit codes that separate "clean", "findings at or above the threshold", "usage error",
  "could not evaluate the target" and "isolation unavailable" — so a pipeline never confuses a broken
  scan with a clean one.

## Why teams use it

- **It runs where the others can't.** Standard-library Python, zero third-party dependencies, no
  package index needed. It installs on an air-gapped host and keeps working.
- **One run, one report.** Code, secrets, IaC, dependencies, containers and CI in a single pass instead
  of six tools with six config files and six formats.
- **A baseline that means something.** The second run shows what changed, not everything again.
- **Isolation tiers.** Static scans are local. Untrusted samples and dynamic analysis run inside a
  confinement tier, and if the tier cannot start the scan refuses rather than running unbounded.

## Speed

Measured on an 8-core laptop, Django 5.1.4 source tree (2,788 files, 71 MB), warm cache, median of two
runs:

| Run | Time |
|---|---|
| Full rule set, one process | 22.2 s |
| Full rule set, four processes | 13.0 s |
| Full rule set, seven processes | 8.6 s |
| A 67-file service tree | 1.4 s |

## Output example

```text
$ shengcun scan .

[CRITICAL] SEC-001 [CWE-798]: AWS Access Key ID Detected
  --> server/config.py:42:22
  Fix:  Store AWS credentials in IAM roles or environment variables.

[CRITICAL] AST-001 [CWE-78]: Command injection: subprocess with shell=True
  --> pipeline/run.py:115:5
  Fix:  Set shell=False and pass an argument list.

[CRITICAL] IAC-TF-001 [CWE-284]: Security group ingress open to 0.0.0.0/0 on port 22
  --> infra/main.tf:18:5
  Fix:  Restrict cidr_blocks or move behind a bastion.

Scan Summary: 3 CRITICAL | 0 HIGH | 0 MEDIUM | 0 LOW
```

## Interfaces

- **CLI** — one command, one exit code, works in a pre-commit hook or a build step.
- **CI** — SARIF upload for native code-scanning views, plus workflow annotations.
- **Review and remediation** — change-scoped scanning, a PR review verdict, and remediation
  suggestions, each optional.
- **Operations** — history database, dashboards, per-client batch scanning, baselines and policy rules.

## Editions

A lite edition is being cut for release under AGPL-3.0. The full edition is available as a commercial
licence, with the isolation tiers, the review and remediation surfaces, compliance mapping and support.

## Request a demo

Open an issue on this repository or use the contact address in the repository profile. Demo requests
are answered with a run against a codebase of your choosing.

## Security

Please do not open public issues for vulnerabilities. Use the private reporting channel in the
repository security policy.

## License

This showcase repository is documentation only. The product is licensed separately; see the edition
notes above.
