# Security Policy

We take security seriously and value responsible disclosure. This policy explains how to report suspected security vulnerabilities affecting CitOmni projects, how reports are triaged, and how coordinated disclosure is handled.

## Scope and Applicability

This is CitOmni's organization-wide default security policy. It applies to repositories in the `citomni` GitHub organization that do not provide their own `SECURITY.md`. A repository-specific security policy takes precedence when present.

Security reports are appropriate for vulnerabilities affecting:

- CitOmni-maintained source code and packages
- Official CitOmni release artifacts and distribution mechanisms
- Authentication, authorization, request handling, configuration, persistence, CLI, or other framework components
- Supply-chain paths that could compromise CitOmni packages or their consumers

## Supported Versions

Security fixes target actively maintained release lines and branches. When a repository documents supported versions, that repository-specific information is authoritative.

As a default:

- The current maintained release line or branch receives priority
- Older maintained lines may receive fixes when severity, exploitability, and backport risk justify it
- End-of-life or otherwise unsupported versions may require upgrading to a maintained version

Exact package inventories and version support can change over time, so repository-specific release information should be used when available.

## Reporting a Vulnerability

Please do **not** open a public issue, discussion, pull request, or other public report for a suspected security vulnerability before coordinated disclosure.

Preferred reporting channels:

- Email: **security@citomni.com**
- If the affected repository offers GitHub's **Report a vulnerability** action under Security Advisories, you may use that private reporting channel instead

Include, as applicable:

- The affected repository or package and version, release, or commit
- A clear description of the vulnerability and potential impact
- Minimal, deterministic reproduction steps or proof of concept
- Relevant environment details, such as PHP version, OS, web server, and runtime mode
- Relevant configuration details with secrets removed
- Sanitized logs, stack traces, screenshots, or other supporting evidence
- Any known mitigations or conditions required for exploitation

For exposed secrets, credentials, or tokens, revoke or rotate them first where possible. Do not send active secrets unless they are essential to the report and a secure transfer method has been agreed in advance.

## Triage and Communication

We aim to:

- **Acknowledge** a report within **72 hours**
- Provide an **initial assessment** within **7 days**
- Communicate meaningful status changes while a valid issue is being investigated and remediated

These are response goals, not guarantees. Complex issues, incomplete reports, or coordination with downstream consumers may require additional time.

During triage, a report may be classified as valid, not reproducible, duplicate, out of scope, or requiring additional information.

## Severity and Remediation

We use **CVSS v4.0** where practical and may also provide a **CVSS v3.1** score when required for ecosystem or tooling compatibility.

CVSS is one input to prioritization. We also consider real-world exploitability, affected scope, default exposure, available mitigations, and downstream impact.

For valid vulnerabilities, remediation priority is based on risk. Where feasible, we may provide temporary configuration or operational mitigations before a patched release is available.

## Fix and Disclosure Process

1. Reproduce the issue and determine root cause and affected scope.
2. Identify affected and supported versions or branches.
3. Prepare and verify fixes for supported targets, including regression tests where practical.
4. Prepare remediation guidance and temporary mitigations where useful.
5. Coordinate disclosure with the reporter and affected downstream maintainers or distributors when necessary.
6. Release fixes before or alongside public disclosure whenever practical.
7. Publish an advisory and request a CVE when appropriate.

For public repositories where GitHub Security Advisories are available, a GHSA may be used to coordinate and publish the advisory.

## Coordinated Disclosure

We support coordinated disclosure. Please keep vulnerability details private until a fix or mitigation is available and disclosure timing has been coordinated.

Disclosure timing depends on severity, exploitability, fix complexity, and the time reasonably required for affected users or downstream maintainers to update. We may request a limited embargo when necessary to reduce risk while patches propagate.

We credit reporters by name or handle unless anonymity is requested.

## Responsible Testing

- Test only against systems, data, and accounts you own or are explicitly authorized to test
- Do not exfiltrate real data or access more data than is necessary to demonstrate the issue
- Do not intentionally disrupt services, degrade availability, or affect other users
- Respect rate limits and applicable legal boundaries
- Prefer local, isolated, or disposable test environments
- If you inadvertently access sensitive data, stop, retain only the minimum information needed to report the issue, and report it privately

## Safe Harbor

We will not pursue or support legal action for good-faith security research that:

- Follows this policy
- Avoids privacy violations and intentional service disruption
- Does not exploit a vulnerability beyond what is reasonably necessary to demonstrate it
- Reports findings privately and provides a reasonable opportunity to investigate and remediate them

This safe harbor does not authorize testing against third-party systems or data that you do not have permission to access.

## Contact

- Security reports: **security@citomni.com**
- Non-security bugs and support questions: Use the relevant repository's normal issue or support channel

_Thank you for helping keep the CitOmni ecosystem safe._
