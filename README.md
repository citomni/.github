# CitOmni

**Low overhead. High performance. Ready for anything.**

CitOmni is a modular PHP framework for HTTP and CLI applications. Its current core packages target **PHP 8.5+** and are delivered as Composer dependencies rather than copied into application source trees.

CitOmni is built around a straightforward premise. Framework code should do as little unnecessary work as possible while making the important decisions explicit. That means deterministic application boot, lean execution paths, lazy services, clearly owned side effects, and reusable capabilities that an application installs only when it needs them.

**Green by Design** is part of this engineering philosophy, not a separate marketing feature. Reducing avoidable CPU work, memory allocations, file I/O, and dependency overhead can improve application response times, hosting efficiency, and resource utilization. The measurable effects depend on the actual workload and deployment environment.

[Website](https://www.citomni.com) | [GitHub organization](https://github.com/citomni) | [All public repositories](https://github.com/orgs/citomni/repositories) | [Documentation](https://github.com/citomni/docs)

---

## Highlights

- **Deterministic boot.** Configuration, services, HTTP routes, and CLI commands come from explicit sources with defined precedence rules.
- **No runtime discovery theater.** No classpath scanning, reflection-driven autowiring, or implicit controller registration as part of normal dispatch.
- **Small, independently useful packages.** The shared kernel does not force an HTTP stack into a CLI application or a database layer into an image-processing job.
- **Lazy, memoized services.** Registered services are constructed when first accessed and reused within the same `App` instance.
- **Compiled, OPcache-friendly maps.** Configuration, services, routes, and commands can be prepared ahead of request handling.
- **Explicit architectural ownership.** Adapters handle transport, Operations orchestrate when needed, Repositories own SQL and datastore I/O, and Services provide registered, reusable capabilities.
- **Fail-fast contracts.** Missing service registrations and invalid definitions surface as errors instead of silently changing behavior.
- **Performance and sustainability as design constraints.** Avoid unnecessary CPU cycles, allocations, network calls, and filesystem operations; measure the result rather than assuming it.
- **Works with ordinary PHP tooling.** Composer, PSR-4, PHP CLI, supported PHP extensions, and conventional web servers remain the foundation.

The objective is not the smallest amount of source code at any cost. It is the smallest sensible amount of *work* and abstraction for the job.

---

## What CitOmni is

CitOmni provides a shared application model, separate HTTP and CLI runtimes, and focused packages for reusable technical capabilities.

A typical application has its own source code and configuration, while framework implementations remain installed under `vendor/` through Composer. Applications can combine both runtime modes, sharing business Operations, Repositories, configuration, and application services while keeping HTTP and CLI adapters separate.

CitOmni does not impose an ORM, a reflection-based dependency-injection container, a particular database schema, or a mandatory business-domain hierarchy. It also does not require every application to install every CitOmni package.

The framework's job is to make the application easier to structure and cheaper to run, not to conceal what the application is doing.

---

## Architecture at a glance

```text
Application (PSR-4 source, config, templates, state)
|
+-- HTTP delivery                         citomni/http
|   +-- Routes, controllers, requests, responses
|   +-- Templates, HTTP sessions, CSRF, cookies
|
+-- CLI delivery                          citomni/cli
|   +-- Commands, arguments, output, exit codes
|
+-- Shared application behavior
|   +-- Operations                       orchestration when justified
|   +-- Repositories                     SQL and persistence
|   +-- Registered Services              reusable application-aware tools
|   +-- Policies, State, Support, Utils  focused responsibilities
|
+-- Reusable provider packages           installed only as needed
|   +-- Infrastructure, image, upload, jobs, AI, data integrations
|
+-- Shared application core              citomni/kernel
    +-- App, Cfg, service resolution, provider composition, cache mechanics
```

This is a responsibility map, not a mandatory call chain. A simple CRUD controller can call a Repository directly. More involved transport-independent workflows can use an Operation. Introducing a layer is useful only when it establishes a meaningful boundary or removes duplication.

### Application layers

| Layer | Responsibility |
|---|---|
| **Controllers** | HTTP adapters for input parsing, session/CSRF handling, and response shaping. They do not own substantial business orchestration or SQL. |
| **Commands** | CLI adapters for arguments, terminal output, and exit codes. They share the same business boundaries as controllers. |
| **Operations** | Optional, explicitly instantiated coordination and decision logic independent of HTTP or CLI. They do not own SQL or transport responses. |
| **Repositories** | All SQL and persistence access, behind one identifiable application boundary. |
| **Services** | Reusable, App-aware tools with explicit service-map registrations. |
| **Contracts, Policy, State, Support, Enum** | Focused extension interfaces, rules, lifecycle state, helpers, and bounded vocabulary. |
| **Utils** | Pure calculations without App, configuration, external I/O, or mutable runtime state. |
| **Exceptions** | Transport-independent failure semantics that an HTTP or CLI adapter can translate if appropriate. |

These boundaries are intended to keep responsibilities clear without building a ceremony-heavy abstraction framework around a simple application.

### Explicit registration and lazy resolution

A CitOmni `App` resolves registered services through the service map and memoizes them per application instance.

```php
$log = $this->app->log;

if ($this->app->hasService('db')) {
	// The database service is registered; constructing it is still lazy.
}
```

Service definitions may specify a class or an explicit class-and-options array. The supported constructors are `new Service($app)` and `new Service($app, $options)`. Unknown or invalid definitions fail rather than triggering implicit discovery.

Provider classes are registered through the application's `config/providers.php`. Package Registries contribute constant configuration and service maps, and may contribute HTTP routes or CLI commands. A package does not need to be copied into the application or invoked through a custom bootstrap mechanism.

### Configuration and dispatch

Each execution mode has its own vendor baseline. The kernel composes provider contributions and application-owned configuration with deterministic precedence.

For configuration, the effective sequence is:

1. Mode-specific vendor baseline.
2. Registered providers in configured order, using each provider's common defaults and then its mode-specific defaults.
3. App common configuration, followed by mode-specific configuration.
4. App common environment overlay, followed by the mode-specific environment overlay.

Associative configuration values are merged recursively, and lists are replaced. Later configuration layers win at conflicting leaves. `$app->cfg` is a deep, read-only `Cfg` wrapper. A direct read of a missing key throws `OutOfBoundsException`. Cfg that a package's Registry ships, or that the app sets in its own config files, is read directly by its owner; app cfg read by a package, another package's cfg, and a path present in one mode only are read with `?? <default>`.

Service registrations use their own precedence rules. Application maps override provider registrations, provider registrations override the mode baseline, and mode-specific application services override common application services. HTTP routes and CLI commands are separate dispatch maps, not configuration subtrees.

See the [`citomni/kernel` manual](https://github.com/citomni/kernel/blob/main/README.md) and [architecture reference](https://github.com/citomni/docs/blob/main/concepts/architecture.md) for the detailed contracts.

### Compiled runtime maps

CitOmni can load prepared PHP array maps instead of rebuilding the same registrations on every request.

| HTTP mode | CLI mode |
|---|---|
| `var/cache/cfg.http.php` | `var/cache/cfg.cli.php` |
| `var/cache/services.http.php` | `var/cache/services.cli.php` |
| `var/cache/routes.http.php` | `var/cache/commands.cli.php` |

Maps are plain PHP data structures suited to OPcache. They still need appropriate cache warming or invalidation when provider registrations, configuration, routes, commands, or installed package versions change. Cache freshness is a deployment responsibility, not a license to ignore changes in source configuration.

The [`citomni/cli` manual](https://github.com/citomni/cli/blob/main/README.md) documents the supported `cache:warm` and `cache:clear` commands. `citomni/http` also has its own separate template-compilation cache.

---

## CitOmni ecosystem

CitOmni is an ecosystem of libraries, providers, project skeletons, and supporting repositories. The tables below describe the **publicly visible GitHub repositories**, not a guarantee that every repository is a published, production-ready Composer package or that every access-controlled CitOmni package is publicly visible.

### Core, runtimes, and reusable capabilities

| Area | Repository | Responsibility |
|---|---|---|
| Application core | [`citomni/kernel`](https://github.com/citomni/kernel) | Shared `App`, read-only `Cfg`, deterministic composition, base classes, and compiled-map mechanics. |
| HTTP delivery | [`citomni/http`](https://github.com/citomni/http) | HTTP kernel, routing, request/response, templates, sessions, cookies, CSRF, and HTTP-specific utilities. |
| CLI delivery | [`citomni/cli`](https://github.com/citomni/cli) | CLI kernel, command dispatch, diagnostics, terminal output, and cache-management commands. |
| Shared infrastructure | [`citomni/infrastructure`](https://github.com/citomni/infrastructure) | MySQL/MariaDB access, value conversion, JSONL logs, secrets, translations, outbound HTTP, email, and brute-force protection. |
| Image processing | [`citomni/image`](https://github.com/citomni/image) | Image inspection, deterministic transformations, encoding, output verification, and captcha-image rendering. |
| File intake and storage | [`citomni/upload`](https://github.com/citomni/upload) | Validated file intake, public/private local storage, and image outputs delegated to `citomni/image`. |
| Long-running jobs | [`citomni/jobrunner`](https://github.com/citomni/jobrunner) | DB-backed jobs executed through fresh CLI processes, with job state, logs, progress, heartbeat, and cancellation. |
| Conversational AI | [`citomni/helloai`](https://github.com/citomni/helloai) | Normalized chat requests/responses, provider adapters, profiles, and optional DB-backed caching. |
| Vector embeddings | [`citomni/vectorembedding`](https://github.com/citomni/vectorembedding) | Normalized embedding requests/responses and provider-specific adapters. |
| Knowledge retrieval | [`citomni/knowledgebase`](https://github.com/citomni/knowledgebase) | Document ingest, chunking, hybrid retrieval, search, and knowledge-grounded question answering. |
| Danish public data | [`citomni/danish-public-sector-data`](https://github.com/citomni/danish-public-sector-data) | Reusable clients for selected Danish public-sector registries and data sources. |
| Scaffold lifecycle | [`citomni/installer`](https://github.com/citomni/installer) | Discovers and materializes package-owned application files, with environment selection, repair, and synchronization. |

### Project starting points

| Repository | Responsibility |
|---|---|
| [`citomni/app-skeleton`](https://github.com/citomni/app-skeleton) | Current mode-neutral project skeleton for new HTTP, CLI, or combined applications. |
| [`citomni/provider-skeleton`](https://github.com/citomni/provider-skeleton) | Mode-neutral starting point for new CitOmni provider packages. |
| [`citomni/http-skeleton`](https://github.com/citomni/http-skeleton) | Older HTTP-specific application skeleton. Its documented PHP 8.2-era setup does **not** represent the current recommended starting point. |

### Documentation, governance, and repositories in preparation

| Repository | Responsibility or status |
|---|---|
| [`citomni/docs`](https://github.com/citomni/docs) | Architecture references, conventions, usage guides, and technical reports. |
| [`citomni/.github`](https://github.com/citomni/.github) | Organization profile, shared GitHub templates, contribution policies, and security reporting. |
| [`citomni/pdf`](https://github.com/citomni/pdf) | Public repository currently empty; **not** presented here as an installable or implemented PDF capability. |

Additional access-restricted or application-specific projects can exist outside this public index. For example, internal CitOmni work may involve `citomni/authenticate`, `citomni/admin`, `citomni/devkit`, or `citomni/testing`. Their current distribution, access requirements, and licenses must be verified individually; their names here are **not** a promise of publicly accessible Composer packages.

For a live inventory, use [the organization's repository list](https://github.com/orgs/citomni/repositories). For any particular dependency, its **current source, `composer.json`, README, and license** take precedence over this overview.

---

## Get started

### Requirements

- PHP **8.5+** for the current core and recommended application skeleton.
- Composer for application and dependency management.
- The PHP extensions required by the packages you choose, as listed in their `composer.json` files.
- A compatible web server for HTTP applications or PHP CLI for command-line applications.
- OPcache is strongly recommended for production workloads.

Older repositories may still contain different or outdated requirements. Check the actual package being installed instead of assuming one rule covers all historical skeletons.

### Create a new application

Start with the mode-neutral application skeleton:

```bash
composer create-project citomni/app-skeleton my-app
cd my-app
```

This creates the application container and provides CitOmni installer tooling, without assuming an HTTP or CLI runtime.

### Add HTTP delivery

```bash
composer require citomni/http
vendor/bin/citomni-installer install --environment=dev
```

The installer materializes package-owned HTTP scaffold, including the front controller under `public/`. For a local development server:

```bash
php -S 127.0.0.1:8000 -t public
```

Point production web-server document roots at `public/`, not the application root. Select the correct environment when materializing deployed entrypoints. The [`citomni/http` manual](https://github.com/citomni/http/blob/main/README.md) covers routing, configuration, controllers, the TemplateEngine, and security features.

### Add CLI delivery

For a CLI application, or to add CLI support to an existing CitOmni app:

```bash
composer require citomni/cli
vendor/bin/citomni-installer install --environment=dev
php bin/citomni list
```

The same application may install HTTP and CLI together. Each mode owns its delivery adapters and entrypoint, while the shared application model and registered cross-mode capabilities remain reusable. The [`citomni/cli` manual](https://github.com/citomni/cli/blob/main/README.md) covers command registration, options, diagnostics, and caching.

### Add infrastructure or another provider

For shared database, logging, mail, translation, and related technical services:

```bash
composer require citomni/infrastructure
```

Enable the provider in the application's `config/providers.php`, alongside any other required providers:

```php
<?php
declare(strict_types=1);

return [
	\CitOmni\Infrastructure\Boot\Registry::class,
];
```

Add other provider packages only when the application needs their functionality. See each package's README for its actual registration, configuration, and dependencies. For package-owned scaffold and upgrades, follow the [`citomni/installer` manual](https://github.com/citomni/installer/blob/main/README.md) rather than copying runtime files out of `vendor/` by hand.

---

## Green by Design

Resource efficiency is a first-class architectural objective in CitOmni. It shapes how the framework boots, resolves services, handles dispatch, creates caches, and divides responsibilities. The practical goal is simple. Use CPU time, memory, disk operations, and network activity for useful application work rather than avoidable framework machinery.

This is both a performance concern and a sustainability concern. At sufficient scale, avoiding repeated work can reduce server time and the infrastructure capacity required for a workload. But lower latency, lower memory usage, lower electricity consumption, and lower emissions are **different measurements**. None should be used as an automatic substitute for another.

### The technical foundations

| Engineering choice | Why it matters |
|---|---|
| **Explicit boot and registration** | Avoids directory scans, reflection-based service discovery, and incidental initialization on ordinary request paths. |
| **Small mode-specific runtimes** | CLI work does not need to initialize HTTP delivery; HTTP requests do not need a command dispatcher. |
| **Lazy service construction** | A service unused by a request need not construct objects or trigger its first-use dependencies. |
| **Memoized service instances** | Repeated access to the same App service ID avoids unnecessary reconstruction within the request/process. |
| **Compiled configuration and dispatch maps** | Moves merge and normalization work away from hot paths where prepared arrays can be reused. |
| **OPcache-friendly PHP** | Lets PHP reuse compiled source and map bytecode, reducing repeated parse/compile work. |
| **Explicit persistence boundaries** | Makes repeated queries, unnecessary round trips, and avoidable datastore I/O easier to identify. |
| **Transport-independent Operations** | Lets the same workflow serve HTTP and CLI without duplicating substantial business logic. |
| **Bounded, fit-for-purpose APIs** | Avoids carrying unused abstractions and adapters into every execution path. |
| **Predictable failure semantics** | Reduces hidden fallback work and makes expensive recovery paths visible and testable. |

These are architectural mechanisms, not a claim that every possible application workload becomes fast merely by using CitOmni. Database queries, external APIs, template complexity, large files, image processing, and application decisions often dominate total response cost.

### Practical green-coding discipline

The framework encourages the same performance questions in application code that it applies to itself.

**Do only the necessary work.** Keep request and command hot paths short. Avoid constructing objects, fetching records, rendering content, or loading resources that will not be used. Minimize repeated normalization and string conversion, especially inside high-frequency loops.

**Keep I/O intentional.** Filesystem access, database round trips, and remote HTTP requests usually cost far more than a small local conditional. Query only the required records and columns, avoid N+1 access patterns, batch work where it preserves clarity, and prevent unexpected network calls during application boot.

**Cache deliberately.** Compiled application maps and OPcache are useful because their input sources are explicit and stable. Application-data caches need equally clear ownership, invalidation rules, memory limits, and correctness guarantees. A cache that is expensive to populate or frequently invalid is not necessarily an improvement.

**Treat memory and copies as costs.** Prefer bounded reads and streaming where the capability supports them, especially for large payloads. Avoid duplicating entire data structures without a reason. Preserve deterministic outputs and correctness instead of adopting an optimization that changes behavior unexpectedly.

**Keep boundaries light.** Prefer a direct Controller-to-Repository call for straightforward CRUD and an Operation when orchestration merits it. Extra layers and interfaces are not free, either to execute or maintain. Abstractions need a concrete purpose.

**Measure before and after.** Use representative workloads, realistic production settings, and appropriate percentiles. Keep cold-start and warm-cache measurements separate. Test the impact of throughput, concurrency, I/O, and payload sizes, not just one tiny request on a developer machine.

**Optimize resource use, not just elapsed time.** A faster request that burns much more CPU, or a cache that retains excessive memory, might be the wrong tradeoff. Look at latency, CPU time, peak/retained memory, disk activity, remote calls, and system utilization together.

**Avoid false economy.** A few fewer nanoseconds do not justify fragile code, silent error handling, security regressions, or an unmaintainable architecture. CitOmni prioritizes low overhead **within** clear correctness and safety contracts.

### The 2025 capacity and resource-efficiency study

CitOmni published a [Capacity & Green-by-Design report dated 2 October 2025](https://github.com/citomni/docs/blob/main/reports/2025-10-02-capacity-and-green-by-design.md). It examined one deliberately minimal HTTP page on low-cost shared hosting, using a framework boot, simple template rendering, and translations without application database work.

Representative measurements reported for that environment were:

| Measurement | Reported value | Context |
|---|---|---|
| Server-side request duration | Approximately **0.004 s** | One minimal page, warm conditions. |
| PHP peak memory accounting | Approximately **0.5 MB** | PHP-reported peak allocation, **not** total OS process RSS. |
| Included PHP files | **24** | The historical tested stack, not a current package-wide invariant. |
| Estimated sustainable throughput | Approximately **175 requests/s per worker** | A model based on 4 ms/request and a hypothetical 70% worker utilization, **not** measured concurrent throughput. |

The report includes environment details, methodology, calculations, and the assumptions behind its worker-capacity estimates. The 175 RPS figure follows the illustrative calculation `0.70 / 0.004 = 175`; it should not be quoted as a measured throughput result or a service-level commitment.

The report reflects the **October 2025 implementation and PHP environment**, including historical rendering components. CitOmni has changed substantially since then, and the current PHP 8.5-based package ecosystem is not identical to the stack measured in that report. The results are useful as a documented example of the engineering direction, not proof of the present runtime's performance on every application.

There was no direct electricity metering, carbon-intensity analysis, or lifecycle assessment in that benchmark. Accordingly, the report does not establish a quantified reduction in energy consumption or CO2 emissions.

### How to evaluate current performance

For meaningful comparisons, record at least:

1. **Workload.** Route or command, data volume, template complexity, query count, remote calls, and relevant request parameters.
2. **Environment.** PHP version, extensions, OPcache settings, server/CPU, PHP-FPM or CLI mode, database, and relevant deployed package versions.
3. **Warm and cold behavior.** Separate first-run boot, cache compilation, and warmed request/command iterations.
4. **Latency.** Median and tail measurements such as p50 and p95 over a sufficient representative sample.
5. **Resource use.** CPU time, PHP memory usage and peak memory, OS memory where available, I/O counts, bytes transferred, and number of included files where informative.
6. **Capacity and contention.** Concurrency, worker occupancy, CPU saturation, queueing behavior, and failure rates under load.
7. **Correctness.** Confirm that an optimization produces the same intended behavior, including errors, permissions, and race-condition handling.

Benchmark two comparable implementations with the same inputs and hardware. Do not compare a warm minimal route with a cold database-heavy route and attribute the difference to the framework. Document assumptions and any extrapolation from latency to throughput.

### Sustainability beyond PHP execution

Less CPU work and fewer unnecessary allocations are generally useful, but environmental impact also depends on hardware utilization, energy mix, request volume, data transfer, storage, and equipment lifetimes. Efficiency improvements can enable higher request volume, so reduced energy **per request** need not mean reduced total energy use.

Green coding therefore calls for both good local engineering and responsible system-level measurement. Less waste can support higher server density, reduce cooling demand, or postpone hardware purchases, but only when it changes the actual deployment footprint. Avoiding or extending hardware replacement can also matter for embodied emissions from manufacturing. The primary promise is transparent, efficient software design; specific environmental gains need evidence from a defined workload and hosting environment.

Further context on data-center efficiency and the importance of careful measurement:

- [IEA - Data Centres and Data Transmission Networks](https://www.iea.org/energy-system/buildings/data-centres-and-data-transmission-networks)
- [Masanet et al. (Science, 2020) - Recalibrating global data center energy-use estimates](https://doi.org/10.1126/science.aba3758)
- [IEA 4E (2025) - Data Centre Energy Use: Critical Review of Models and Results](https://www.iea-4e.org/wp-content/uploads/2025/05/Data-Centre-Energy-Use-Critical-Review-of-Models-and-Results.pdf)
- [Lawrence Berkeley National Laboratory - Data Center Efficiency Research](https://datacenters.lbl.gov/)

These are background references on energy use and efficiency; they are not independent measurements of CitOmni.

---

## Performance, deployment, and operational expectations

- **Install framework packages through Composer.** Keep framework source in `vendor/`; application files created from package-owned stubs are managed through the installer lifecycle.
- **Use explicit environment selection.** Development, staging, and production can have different configuration overlays and runtime entrypoints.
- **Warm or invalidate compiled maps deliberately.** Changing providers, routes, service registrations, commands, or configuration may require rebuilding the affected artifacts.
- **Configure OPcache for the actual deployment model.** When timestamp validation is disabled, deployments must arrange appropriate invalidation or restart behavior.
- **Keep internal paths outside the document root.** Configuration, application state, secrets, repositories, and source files must not be served as public assets.
- **Keep SQL in Repositories.** This makes persistence behavior visible and testable, and helps identify unnecessary round trips.
- **Avoid hiding broken assumptions behind broad catches.** Recover explicitly when a fallback is justified; otherwise let the owning error handler surface the failure.
- **Test the selected package combination.** A package's optional integrations and PHP extensions have their own requirements, and the final application's behavior depends on its installed versions and configuration.

The package manuals document the exact APIs and deployment semantics of each component. The code in the installed version is authoritative when a general overview or older documentation disagrees.

---

## Security, privacy, and responsible disclosure

CitOmni aims for explicit security boundaries and predictable handling of sensitive operations. Its HTTP package provides primitives such as CSRF, cookies, sessions, webhook verification, and request/response handling; identity and authorization policies remain the responsibility of the relevant package or application.

The core framework does not require an external telemetry service or automatic phone-home mechanism for normal application execution. Applications may explicitly use outbound HTTP, mail, logging, analytics, and other integrations; they must configure and operate those services responsibly.

If you believe you have found a vulnerability, **do not publish it as a GitHub issue or pull request before coordinated disclosure**. Follow the [organization security policy](SECURITY.md), which explains private reporting channels, expectations, and safe testing. Repository-specific security guidance takes precedence where present.

---

## Documentation and contributions

This repository is the **organization-level overview**, not the API manual for every CitOmni package.

- For an individual package, start with its repository README and `composer.json`.
- For cross-cutting concepts, see the [architecture reference](https://github.com/citomni/docs/blob/main/concepts/architecture.md), [runtime modes](https://github.com/citomni/docs/blob/main/concepts/runtime-modes.md), and [provider model](https://github.com/citomni/docs/blob/main/concepts/services-and-providers.md).
- For coding standards, see the [shared coding and documentation conventions](https://github.com/citomni/docs/blob/main/contribute/CONVENTIONS.md). Current package requirements and implementation take precedence if an older document has not yet been updated.
- For contributions, read the [contribution guide](CONTRIBUTING.md) and the destination repository's specific instructions.

CitOmni's normal implementation style is PHP 8.5+, PSR-1/PSR-4 naming, K&R braces, tabs for indentation, English documentation and PHPDoc, explicit service registration, low runtime overhead, and failures that do not disappear silently. Changes to hot paths should include evidence that they preserve correctness and achieve a worthwhile performance improvement.

---

## About this repository

`citomni/.github` hosts organization-wide GitHub community resources. It is not a Composer framework package.

| File | Purpose |
|---|---|
| [`profile/README.md`](profile/README.md) | Short public presentation on the [CitOmni organization page](https://github.com/citomni). |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Contribution workflow and expectations. |
| [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) | Community conduct policy. |
| [`SECURITY.md`](SECURITY.md) | Responsible vulnerability reporting and disclosure. |
| [`LICENSING.md`](LICENSING.md) | Licensing overview across repository categories. |
| [`TRADEMARKS.md`](TRADEMARKS.md) and [`NOTICE`](NOTICE) | Trademark ownership, naming, and brand-use notices. |
| [GitHub issue and pull-request templates](https://github.com/citomni/.github/tree/main/.github) | Shared templates for repository collaboration. |

The short GitHub organization profile and this longer repository README have different jobs. The profile introduces the framework; this document explains the design, ecosystem, starting points, and reasons behind CitOmni's approach.

---

## Licensing

CitOmni repositories may have **different licenses** depending on their purpose and distribution model. The public framework libraries covered above generally use MIT; the `.github` documentation and organization material use Creative Commons Attribution-ShareAlike 4.0. Access-restricted or proprietary packages may use other terms.

Always consult the affected repository's own `LICENSE` for the authoritative grant. The [licensing overview](LICENSING.md) provides additional context.

Copyright permissions are separate from trademark rights. **CitOmni** and the CitOmni logo are trademarks of **Lars Grove Mortensen**. See [TRADEMARKS.md](TRADEMARKS.md) and [NOTICE](NOTICE). Do not imply official endorsement or affiliation without permission.

---

## Author

Developed by [Lars Grove Mortensen](https://github.com/LarsGMortensen) and contributors.

Copyright (c) 2012-present Lars Grove Mortensen and CitOmni contributors. The `.github` repository's documentation is licensed under [CC BY-SA 4.0](LICENSE).

---

**CitOmni - Low overhead. High performance. Ready for anything.**
