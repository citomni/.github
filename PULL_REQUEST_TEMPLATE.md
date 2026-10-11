<!--
Organization-wide CitOmni pull request template.
Keep answers concise but reproducible. Use "Not applicable" for irrelevant sections.
Follow any additional requirements documented by the target repository.
Do not disclose vulnerabilities, credentials, personal data, or other secrets in a public pull request.
-->

## Why

<!-- Describe the problem, requirement, or maintenance issue. Link a relevant issue if one exists. -->

## What changed

<!-- Summarize the important code, configuration, documentation, and behavioral changes. Keep unrelated changes out of this pull request. -->

## How it was verified

<!-- Give the exact commands, PHP version, relevant extensions/environment, results, and any checks you could not run. "Not run" is not "passed". Documentation-only changes do not need artificial runtime tests. -->

- **Environment:**
- **Commands and results:**
- **Not run / limitations:**

### Verification checklist

<!-- Check what applies; explain deliberate exceptions above. Run the test commands that actually exist in this repository. -->

- [ ] Relevant focused and aggregate tests pass, or omitted tests and their reasons are documented.
- [ ] Regression coverage is included where the change fixes a defect, or the limitation is explained.
- [ ] Changed PHP is checked against the package's supported PHP and extension requirements.
- [ ] Changed examples, configuration, and documentation have been checked for accuracy.

For packages using the [CitOmni Testing Guide](https://github.com/citomni/docs/blob/main/contribute/testing-guide.md), the aggregate entry point is `php tests/run.php`. Other repositories may expose `composer test` or other scripts. Do not assume PHPUnit, PHPStan, or an identical test command exists everywhere.

## Public contracts and deployment

<!-- State "No change" when appropriate. Identify downstream impacts explicitly. -->

- **Public API / service IDs / Registry contributions:**
- **Configuration / HTTP routes / CLI commands:**
- **Database schema / migrations / state contracts:**
- **Caches / scaffold / deployment / rollback considerations:**
- **Compatibility or migration guidance:**

- [ ] I have checked whether public behavior or compatibility is affected and documented any required consumer action.

## Performance and Green by Design

<!-- For changes to boot, routing, dispatch, cache handling, database I/O, serialization, or other hot paths, describe the performance effect. Include representative before/after measurements when practical. For changes with no meaningful runtime effect, write "Not applicable". -->

- **Workload and environment:**
- **CPU / wall time / memory / I/O impact:**
- **Measurements, tradeoffs, or reason not measured:**

Do not present estimated throughput, historical benchmarks, or unmeasured resource savings as current guarantees. See [Coding & Documentation Conventions](https://github.com/citomni/docs/blob/main/contribute/CONVENTIONS.md).

## Security, dependencies, and licensing

- [ ] No credentials, private keys, tokens, personal data, or other secrets are added to source, tests, logs, or documentation.
- [ ] Security-sensitive changes, new dependencies, and bundled third-party material have been reviewed, or are not applicable.
- [ ] New or modified third-party notices and SPDX/license metadata are consistent with the target repository's `LICENSE`, or are not applicable.
- [ ] I have the right to submit this contribution under the target repository's applicable license.

Use the target repository's `LICENSE`; do not infer its license from the CitOmni organization name. See the [Licensing Overview](https://github.com/citomni/.github/blob/main/LICENSING.md). Report suspected vulnerabilities **privately**, following the [Security Policy](https://github.com/citomni/.github/blob/main/SECURITY.md), not through a public pull request.

## Contributor sign-off

- [ ] Each commit includes a Developer Certificate of Origin (DCO) `Signed-off-by` trailer identifying its author.

Use `git commit -s` to add a DCO trailer. This is different from cryptographic commit signing with `git commit -S`. See [Contributing to CitOmni](https://github.com/citomni/.github/blob/main/CONTRIBUTING.md) for expectations and guidance.

## Review notes

<!-- Optional: Affected consumers, areas needing extra attention, follow-up work, or links to supporting design discussions. Do not turn the pull request into a TODO list for unrelated work. -->
