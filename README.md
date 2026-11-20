# Shengcun

**An offline-first security scanner for code, containers, infrastructure and the supply chain.**

Shengcun reads a repository, a container manifest, a Kubernetes bundle, an IaC tree or a model file,
and writes findings as SARIF 2.1.0 or a CycloneDX SBOM. Static scanning performs no network I/O: it
reads bytes on disk and nothing else.

This repository is the public showcase. The source is private; this page describes what the scanner
covers and how fast it is, not how it works internally.

---

## Coverage

**Application code**
- Vulnerability patterns across Python, JavaScript/TypeScript, Go, Rust, Ruby, PHP, Java, Kotlin, C#,
  Swift, Scala and C/C++, read from a parsed syntax tree rather than from regular expressions.
- Cross-file data flow: a source in one file reaching a sink in another is a finding, not a miss.
- An unresolved call is reported as unresolved rather than silently dropped.

**Secrets**
- Signature families for cloud keys, platform tokens, private keys and connection strings, plus
  entropy analysis over high-entropy strings.
- Optional, read-only verification that a credential is live.
- Git history scanning for credentials already committed.

**Infrastructure as code**
- Dockerfile, Compose, Kubernetes, Terraform, CloudFormation-style templates, GitHub Actions and RBAC
  YAML, each with its own rule domain and line-anchored findings.

**Supply chain**
- Dependency advisories resolved against the versions you actually pin.
- Exploit-likelihood scoring, so a high-CVSS issue nobody exploits does not outrank a live one.
- CycloneDX 1.5 SBOM using the same coordinates the findings cite.

**Containers**
- Image and manifest posture, base-image drift, exposed runtime settings.

**CI/CD**
- Workflow permissions, unpinned third-party actions, script injection through expression contexts.

**Models and agents**
- Model-weight deserialization risks and unsafe loaders.
- Agent and MCP configuration: over-broad tool grants, shell escapes, prompt-injection surfaces.

**Compliance mapping**
- Findings carry their control mappings for SOC 2, PCI DSS, HIPAA and the CIS benchmarks.

**External surface, authorization required**
- Attack-surface discovery, port probing and form auditing behind an explicit authorization flag and a
  scope check that fails closed. Nothing runs against a host you have not declared in scope.

## Review and remediation

- **Change-scoped scanning** — scan only what a branch touches, plus its call-graph neighbourhood, and
  flag code that becomes newly reachable.
- **Pull-request review** — one review comment per PR with a computed verdict, not a wall of findings.
- **Remediation suggestions** — a suggested fix attached to the finding, and an optional assisted fix
  that can be run against a single finding with your own model provider and key.
- **Pre-push gate** — block the push that introduces a finding, without blocking legacy debt.
- **Signed findings** — findings and reports can be signed so a client can prove when something was
  first seen and that a report has not been edited.
- **Impact tracing** — follow a leaked credential to where it actually goes and how to revoke it.
- **Prioritisation** — dependency findings are ranked by whether the vulnerable code is reachable and
  how likely the exploit is in the wild.
- **History and metrics** — findings are tracked over time with mean-time-to-remediation by severity,
  rule and directory.
- **Policy as code** — per-client rules over findings, in a small declarative file.
- **Expiring suppressions** — every exception carries a reason and an expiry, and CI fails the day one
  lapses.
- **Baselines** — acknowledge what exists so the next run reports only what changed.

## Outputs

- SARIF 2.1.0 for GitHub code scanning, Azure DevOps or any SIEM that speaks it.
- CycloneDX 1.5 SBOM.
- JSON, a console report, and a self-contained HTML report that can be white-labelled per client.
- Exit codes that distinguish clean, findings at or above threshold, usage error, target that could not
  be evaluated, and isolation unavailable — a broken scan never looks like a clean one.

## Benchmarks

Measured on an 8-core laptop against the Django 5.1.4 source tree (2,788 files, 71 MB), warm cache,
median of two runs:

| Run | Time |
|---|---|
| Full rule set, one process | 22.2 s |
| Full rule set, four processes | 13.0 s |
| Full rule set, seven processes | 8.6 s |
| A 67-file service tree | 1.4 s |

For reference on the same tree, a Python-only AST linter takes about 17 s and a multi-language
pattern-matching engine about 115 s doing narrower work.

## Deployment

- **CLI** — one command, one exit code, usable in a pre-commit hook or a build step.
- **Air-gapped** — standard-library Python, no third-party dependencies, no package index required.
- **Isolation tiers** — untrusted samples and dynamic analysis run confined; if confinement cannot
  start, the scan refuses instead of running unbounded.
- **Batch mode** — scan many client directories in one run, each with its own report and identity.

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

## Access

Shengcun is available under a commercial licence. Open an issue on this repository to request a demo
against a codebase of your choosing.

## Security

Please do not open public issues for vulnerabilities. Use the private reporting channel in the
repository security policy.
