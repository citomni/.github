# Contributing to CitOmni

CitOmni welcomes focused contributions to its framework packages, provider packages, application skeletons, tooling, and documentation. The objective is not merely to make code work. Changes should preserve the qualities that make CitOmni useful: **explicit behavior, deterministic execution, low overhead, clear ownership, and maintainable contracts**.

This is the organization-wide contribution guide. A repository may have more specific instructions for its tests, dependencies, release process, or public API. Follow the target repository's current code and `composer.json` for implementation facts, and its local contribution instructions where present. Some older documentation may not yet reflect the current implementation.

CitOmni packages are installed as **Composer dependencies**. Do not copy framework source files into an application's `src/` tree to work around package integration issues.

---

## Before contributing

1. Read the target package's `README.md`, `composer.json`, and relevant source and tests. Inspect the actual public API before proposing changes to it.
2. Reproduce the problem or establish a concrete use case. Include the relevant PHP version, runtime mode, configuration, and expected behavior when reporting a bug.
3. Keep the proposed change focused. Fix an existing boundary before introducing an abstraction intended to hide it.
4. For a new public API, cross-package dependency, or architectural change, open a discussion or issue when an early design decision would avoid substantial rework.
5. Use a **private channel** for suspected vulnerabilities. Do not publish an exploit or sensitive proof of concept in an issue or pull request. See [Security](SECURITY.md).

The most helpful contributions are often small, carefully verified improvements with an explanation of why they matter.

---

## Engineering principles

CitOmni's architectural choices serve five connected goals.

- **Determinism.** Registration, configuration precedence, error behavior, and public contracts should be explicit and repeatable.
- **Low overhead.** Avoid needless allocations, filesystem access, queries, network calls, and runtime discovery.
- **High performance.** Optimize actual work and latency, with measurements where performance is material.
- **Clear ownership.** Keep transport, orchestration, persistence, and reusable technical capabilities in their respective layers.
- **Safe failure.** Invalid definitions, configuration mistakes, and unrecoverable failures should surface clearly rather than trigger undocumented fallbacks.

Prefer the shortest design that meets those goals. More classes, interfaces, and dependency layers are not automatically better architecture.

---

## PHP and source conventions

Current maintained CitOmni framework packages target **PHP 8.5 or later**. The target repository's `composer.json` is authoritative for its actual PHP and extension requirements; an older skeleton or document may still specify an obsolete requirement.

For CitOmni PHP code, follow these conventions.

| Concern | Convention |
|---|---|
| Autoloading | Composer and PSR-4 |
| General style | PSR-1 and established repository conventions |
| Classes | `PascalCase` |
| Methods and variables | `camelCase` |
| Constants | `UPPER_SNAKE_CASE` |
| Braces | K&R, opening brace on the declaration line |
| Indentation | Tabs, not spaces |
| Strict typing | `declare(strict_types=1);` immediately after `<?php` in PHP source files |
| Documentation | English for README content, PHPDoc, inline comments, and commit messages |
| Errors | Fail fast unless the caller has a deliberate, recoverable fallback |

Use informative PHPDoc for non-trivial public behavior. Comments should explain constraints, rationale, safety requirements, and non-obvious tradeoffs rather than repeat the next line of code. Match the existing package's SPDX and copyright conventions without changing unrelated file headers.

Avoid mass formatting changes in otherwise unrelated pull requests. Keep the diff reviewable.

The [Coding & Documentation Conventions](https://github.com/citomni/docs/blob/main/contribute/CONVENTIONS.md) provide additional guidance. If a convention example conflicts with the current package implementation, verify the current behavior in source before using or updating it.

---

## Application architecture

CitOmni has two delivery modes, **HTTP** and **CLI**, sharing the application core in `citomni/kernel`. Neither delivery mode should dictate the structure of transport-independent business logic.

| Layer | Responsibility and boundary |
|---|---|
| HTTP Controllers | Parse request input, apply HTTP/session/CSRF concerns, and shape responses. No SQL or substantial business orchestration. |
| CLI Commands | Parse arguments, format terminal output, and return exit codes. No SQL or substantial business orchestration. |
| Operations | Coordinate non-trivial, reusable, transport-independent workflows. No SQL and no HTTP/CLI output shaping. |
| Repositories | Own SQL and datastore persistence, including queries and storage-specific decisions. |
| Services | Registered, App-aware reusable capabilities. No SQL embedded in ordinary services. |
| Contracts | Small, stable public extension interfaces, not a collection of internal interfaces. |
| Policy | App-aware rules and policy-driven decisions. |
| State | Explicit runtime state contracts, transitions, keys, and lifetimes. |
| Support | Focused App-aware supporting classes that do not warrant service registration. |
| Enum | Stable, bounded sets of values. |
| Util | Pure calculations without App, configuration, persistence, I/O, logging, or mutable runtime state. |
| Exceptions | Transport-independent application and domain failure semantics. |

Keep SQL in Repositories even when a higher-level service or Operation needs the data. An HTTP Controller may call a Repository directly for trivial CRUD; introduce an Operation when orchestration or reuse justifies the additional layer.

Operations are constructed explicitly, for example with `new SomeOperation($this->app)`, rather than automatically added to the service map. Conversely, a class under `src/Service/` must be a real service registered through the application's or provider's service map.

Do not introduce a new layer simply because a familiar pattern from another framework has a name for it.

---

## Provider and runtime registration

### Provider Registry

Reusable providers expose declarative contributions through `src/Boot/Registry.php`. Register provider classes explicitly in the consuming application's `config/providers.php`.

The supported Registry contribution constants are:

| Contribution | Purpose |
|---|---|
| `MAP_COMMON` | Service definitions shared by HTTP and CLI. |
| `MAP_HTTP`, `MAP_CLI` | Mode-specific service definitions. |
| `CFG_COMMON` | Shared configuration defaults. |
| `CFG_HTTP`, `CFG_CLI` | Mode-specific configuration defaults. |
| `ROUTES_HTTP` | HTTP dispatch map. |
| `COMMANDS_CLI` | CLI command dispatch map. |

Define only the constants a package actually needs. Do not replace them with legacy `Boot/Services.php`, `Boot/Config.php`, `Boot/Routes::MAP`, or CLI `ROUTES_CLI` patterns.

Keep Registry definitions **side-effect-free**. No database connection, filesystem scan, network request, service construction, or environment-dependent registration should occur while the package is being loaded. Boot contributions declare data; the application runtime resolves and uses it.

Services must use one of the supported definitions:

```php
'id' => \Vendor\Package\Service\ExampleService::class
```

or:

```php
'id' => [
	'class' => \Vendor\Package\Service\ExampleService::class,
	'options' => ['enabled' => true],
]
```

Registered services are created lazily as `new Service($app)` or `new Service($app, $options)` and memoized per `App`. There is no reflection-driven autowiring or fallback class discovery. Invalid definitions should fail visibly.

### Composition and precedence

The kernel composes each active runtime from explicit sources.

**Configuration**, from lowest to highest precedence:

1. HTTP or CLI vendor Registry baseline.
2. Registered providers in order; each contributes `CFG_COMMON`, then its mode-specific configuration.
3. Application common configuration, then HTTP- or CLI-specific configuration.
4. Application common environment overlay, then mode-specific environment overlay.

Configuration uses a recursive merge for associative arrays, with later values winning; lists are replaced. `$app->cfg` is read-only. A direct read of an unknown key throws `OutOfBoundsException`. A package ships a Registry baseline (`CFG_COMMON`, `CFG_HTTP`, `CFG_CLI`) for every cfg path it reads and reads those paths directly; a missing baseline is fixed in the Registry, not with `??` or `isset()` at the call site. App cfg read by a package, another package's cfg, and a path present in one mode only are read with `?? <default>`.

**Service registrations**, from lowest to highest precedence:

1. Mode-specific vendor baseline.
2. Registered providers; their `MAP_COMMON` and mode-specific maps, with later providers able to override earlier registrations.
3. Application `config/services.php`.
4. Application `config/services_http.php` or `config/services_cli.php`.

Service maps use keyed replacement semantics. Within each merge step, PHP array union preserves the left-hand entry; the newer layer is placed on the left. As a result, later sources override earlier service IDs.

**Dispatch maps** are separate from configuration and service registration. HTTP routes use `ROUTES_HTTP` and the application's `citomni_http_routes*.php` files. CLI commands use `COMMANDS_CLI` and `citomni_cli_commands*.php`. Their entries merge with deterministic precedence from vendor through providers to app and environment overlays.

Never invent configuration keys, service IDs, route shapes, or command syntax. Read the relevant package's current Registry and consumers.

### Dependencies and packaging

Keep each Composer package focused on capabilities it actually uses.

- Require `citomni/kernel` where its App and base contracts are needed.
- Require `citomni/http` or `citomni/cli` when the package actually depends on their classes or transport contracts, not merely because its service could be called from such an application.
- Put reusable source under the package's PSR-4 `src/` tree. Keep application configuration, environment-specific secrets, and runtime state in the consuming application.
- Use `citomni/installer` and package-owned scaffold manifests for application files that must be materialized outside `vendor/`. Do not silently copy runtime files during service construction.
- Keep optional extensions and runtime limitations explicit in `composer.json` and documentation.

The [Provider Skeleton](https://github.com/citomni/provider-skeleton) is the starting reference for new packages. Prefer its current Registry contract to examples in older provider documents.

---

## Green coding and performance

**Green by Design is an engineering constraint, not an exemption from evidence.** A change should avoid wasting CPU time, memory, I/O, and infrastructure capacity, but environmental and performance claims must be tied to measured conditions.

### Practical design rules

- **Remove unnecessary work.** Avoid duplicate decoding, parsing, normalization, allocations, and repeated computations on hot paths.
- **Preserve cheap application boot.** Prefer explicit constant maps, lazy service construction, and OPcache-friendly code over runtime scanning, autowiring, and reflection-driven discovery.
- **Use compiled maps appropriately.** Configuration, services, HTTP routes, and CLI commands can be cached as PHP arrays. Avoid rebuilding unchanged maps on every request when a maintained cache is available.
- **Minimize avoidable datastore I/O.** Avoid N+1 queries, retrieve only needed rows or columns, use parameterized SQL, and keep query planning in Repositories.
- **Control filesystem and network work.** Reuse existing data or connections when safe and meaningful; avoid repeating unnecessary reads, writes, and remote calls.
- **Avoid excessive abstraction.** Do not add layers, temporary arrays, wrapper objects, or generic interfaces to a tight execution path without a concrete maintenance or performance benefit.
- **Think about memory lifetime.** Avoid holding large intermediate buffers longer than necessary; prefer streaming or bounded processing where it genuinely improves the workload.
- **Preserve correctness and security.** Logging, input validation, atomic writes, and required checks are not wasteful merely because they incur cost.

Performance is contextual. A database-backed task, an image encoder, and a cached HTTP route have fundamentally different resource profiles. Do **not** treat a historical minimal-page benchmark or an arbitrary millisecond, file-read, memory, or query count as an organization-wide pass/fail budget.

### Measuring performance-sensitive changes

Changes to boot, routing, dispatch, service resolution, cache loading, serialization, database access, or similarly hot code paths should include a short performance note when measurement is practical.

A useful note identifies:

1. **Workload.** Exact operation, input size, request type, relevant dependencies, and concurrent load if applicable.
2. **Environment.** PHP/runtime versions, platform, extension versions, OPcache configuration, and whether caches are warm.
3. **Method.** Baseline and modified code measured under comparable conditions; cold start recorded separately from warm runs.
4. **Measurements.** Wall time (preferably p50/p95), CPU time where available, memory usage/peak, and relevant database, filesystem, or network operations.
5. **Interpretation.** What changed, why the tradeoff is worthwhile, and what cannot be concluded from the measurements.

Use `hrtime(true)` and suitable profiling or benchmark tools, and repeat measurements enough to distinguish a regression from normal noise. Measure a representative production-like configuration when possible; development-only performance flags are diagnostic aids, not a substitute for comparable tests.

If measurement is not feasible, say so plainly and explain the expected work avoided or added. A performance claim without methodology is not acceptance evidence.

The [historical capacity and Green-by-Design report](https://github.com/citomni/docs/blob/main/reports/2025-10-02-capacity-and-green-by-design.md) documents one older HTTP workload. It is useful background, **not** a benchmark of the current codebase or a universal resource budget.

### Cache changes

Current compiled-map paths include:

```text
var/cache/cfg.http.php       var/cache/cfg.cli.php
var/cache/services.http.php  var/cache/services.cli.php
var/cache/routes.http.php    var/cache/commands.cli.php
```

Update cache producers, invalidation behavior, tests, and deployment guidance together when changing map structure or semantics. Do not edit generated cache files as the source of truth. Follow the current [`citomni/cli` cache commands](https://github.com/citomni/cli/blob/main/README.md) for warming and clearing caches.

---

## Testing and verification

Use the target package's **actual test entry points**. CitOmni's current [Testing Guide](https://github.com/citomni/docs/blob/main/contribute/testing-guide.md) documents package-owned standalone PHP suites, including separate integration suites where a database is needed. Do not assume PHPUnit is required or that one test command exists identically in every historical repository.

Examples of commands to use **when the repository supplies them**:

```bash
composer validate
composer test
php tests/run.php
```

For changes to a specific package, check syntax and run the relevant focused and aggregate suites. Test observable behavior rather than only successful construction.

Depending on the change, cover:

- Success, invalid input, boundary values, and explicit error paths.
- The active PHP version, required extensions, and optional capability fallbacks.
- HTTP and CLI behavior separately where both are supported.
- Provider registration, service overrides, configuration precedence, and unknown-key behavior.
- Persistence constraints and transactional effects using disposable test data or an isolated test database.
- Cache generation, loading, stale-cache detection, and concurrency when relevant.
- Security cases such as missing permissions, CSRF validation, replay checks, and secret redaction where applicable.

Keep fixtures small and deterministic. A test that requires network access, real credentials, a writable directory, or a database must make that dependency explicit. Never run destructive tests against production data.

When reporting test results, identify the exact command and environment. If a required suite could not run, distinguish **not run** from **passed**, and explain the blocker. Include regression coverage with the fix when feasible.

---

## Compatibility and changes to public contracts

Treat published package behavior as an intentional contract. Changes to the following can affect applications and downstream packages:

- Public classes, methods, arguments, return values, exceptions, and documented behavior.
- Service IDs, constructor options, and provider Registry contributions.
- Configuration names, defaults, precedence, and missing-key behavior.
- HTTP routes, CLI command names, and their input/output contracts.
- Database schemas, persistence semantics, and state transitions.
- Cache formats and deployment lifecycle assumptions.

Before changing one of these, identify affected consumers, determine whether the change is compatible with the package's supported release line, and describe migration steps when needed. Apply the repository's versioning policy and actual release status; do not automatically treat every internal refactor or every changed key as an identical SemVer event.

Prefer explicit migration guidance over silent compatibility shims or hidden fallbacks. Avoid speculative back-compat layers that do not serve a real supported consumer.

---

## Documentation contributions

Package READMEs should be **usable manuals** covering current capabilities, installation, configuration, public APIs, behavior, failure modes, and examples. They should not become TODO lists, changelogs, or collections of speculative features.

- Keep documentation aligned with the implemented code and released dependencies.
- Use English and executable or otherwise clearly illustrative examples.
- Document meaningful defaults, optional integrations, security boundaries, and limitations.
- Link to canonical architecture or testing references rather than repeating large sections that will drift.
- Keep version histories, release announcements, and migration notes in appropriate release or migration documents instead of turning the README into a development diary.
- Verify relative links and referenced repository names when editing shared documentation.

Documentation-only contributions do not need artificial runtime tests. Validate Markdown, links, and any code examples they introduce or change.

---

## Pull requests and review

Small, focused pull requests are easier to verify and review. The [shared pull request template](PULL_REQUEST_TEMPLATE.md) provides the expected outline. Include enough context for a maintainer to reproduce and evaluate the change.

A useful pull request describes:

- **Why:** The concrete defect, requirement, or maintenance issue.
- **What changed:** The important code, configuration, documentation, and behavioral differences.
- **How to test:** Commands executed, environment, results, and known omissions.
- **Compatibility:** Public API effects, migrations, or deployment steps.
- **Performance:** Measured changes or reasoned impact when relevant.
- **Security and dependencies:** New privileges, secrets, network access, third-party material, or license implications.

Use an imperative commit subject, such as `Fix route cache invalidation` or `Document provider configuration precedence`. Keep unrelated formatting, refactoring, and behavioral changes out of the same commit where possible.

Short branch names such as `fix/<topic>`, `feat/<topic>`, `perf/<topic>`, and `docs/<topic>` are helpful but not a runtime or architectural contract.

### Developer Certificate of Origin sign-off

The current shared pull request template requests a **DCO sign-off** for commits. Use:

```bash
git commit -s -m "Document provider configuration precedence"
```

This adds a `Signed-off-by` trailer expressing the contributor's certification that they have the right to submit the work. It is **not** the same as cryptographic commit signing (`git commit -S`). Follow any more specific repository policy where applicable.

---

## Security, conduct, and licensing

- Report suspected vulnerabilities privately through [SECURITY.md](SECURITY.md) or **security@citomni.com**, not in public issues or pull requests.
- Remove credentials, tokens, personal data, and other secrets from examples, logs, screenshots, and reproduction archives.
- Follow the [Code of Conduct](CODE_OF_CONDUCT.md) for technical discussion and review.
- Contribute only material you have the right to submit. The target repository's `LICENSE` governs contributions unless a repository explicitly states a different arrangement. See [Licensing](LICENSING.md).
- Keep copyright, third-party notices, and attribution accurate. Open-source code licensing does not grant permission to misuse CitOmni trademarks. See [Trademarks](TRADEMARKS.md).

---

## Reference documents

- [CitOmni Architecture](https://github.com/citomni/docs/blob/main/concepts/architecture.md)
- [Current Provider Skeleton](https://github.com/citomni/provider-skeleton)
- [Provider Design Notes](https://github.com/citomni/docs/blob/main/concepts/services-and-providers.md) (verify examples against current Registry contracts)
- [Testing Guide](https://github.com/citomni/docs/blob/main/contribute/testing-guide.md)
- [Coding & Documentation Conventions](https://github.com/citomni/docs/blob/main/contribute/CONVENTIONS.md)
- [Security Policy](SECURITY.md)
- [Licensing Overview](LICENSING.md)
- [Trademark Policy](TRADEMARKS.md)

---

CitOmni - low overhead, high performance, ready for anything.
