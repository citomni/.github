# CitOmni

**Low overhead. High performance. Ready for anything.**

CitOmni is a **modular PHP 8.5+ framework** for web applications, APIs, command-line tools, and automation. It is built from focused Composer packages so applications can use the capabilities they need without carrying an unnecessary framework stack.

The design is deliberately explicit. Deterministic configuration and service maps, lazy services, clear architectural boundaries, and lean execution paths take priority over reflection-driven discovery and hidden runtime work.

[Explore the framework](https://github.com/citomni/.github/blob/main/README.md) · [Browse packages](https://github.com/orgs/citomni/repositories) · [Website](https://www.citomni.com)

---

## One core, two delivery layers

- **[`citomni/kernel`](https://github.com/citomni/kernel)** supplies the shared application core, configuration, service registration, and base classes.
- **[`citomni/http`](https://github.com/citomni/http)** adds HTTP routing, requests, responses, templates, and web-facing security utilities.
- **[`citomni/cli`](https://github.com/citomni/cli)** adds command dispatch, CLI diagnostics, and terminal-oriented execution.

Build on those foundations with optional packages such as [`infrastructure`](https://github.com/citomni/infrastructure), [`image`](https://github.com/citomni/image), and [`upload`](https://github.com/citomni/upload). HTTP and CLI can share one application model without sharing transport concerns.

---

## Start with an application

Create a mode-neutral Composer project:

```bash
composer create-project citomni/app-skeleton my-app
cd my-app
```

Then add [`citomni/http`](https://github.com/citomni/http) for a web application, [`citomni/cli`](https://github.com/citomni/cli) for command-line execution, or both. [`citomni/installer`](https://github.com/citomni/installer) materializes the runtime-owned entrypoints and scaffold files.

**[Read the setup guide](https://github.com/citomni/.github/blob/main/README.md#get-started)** for the next commands and links to the package manuals.

---

## Built for predictable, efficient software

CitOmni aims to spend CPU time, memory, and I/O on application work rather than framework machinery. Its performance philosophy is documented in the [architecture overview](https://github.com/citomni/docs/blob/main/concepts/architecture.md). A [historical shared-hosting measurement report](https://github.com/citomni/docs/blob/main/reports/2025-10-02-capacity-and-green-by-design.md) describes one specific workload; real applications and deployments should be measured independently.

---

## Documentation and community

Explore the [package repositories](https://github.com/orgs/citomni/repositories) for implementation manuals and the [documentation repository](https://github.com/citomni/docs) for broader architecture and conventions.

- [Contributing](https://github.com/citomni/.github/blob/main/CONTRIBUTING.md)
- [Security policy](https://github.com/citomni/.github/blob/main/SECURITY.md) - Report vulnerabilities privately, not in public issues.
- [Licensing and trademarks](https://github.com/citomni/.github/blob/main/LICENSING.md) - Check each package's own license and the project's trademark notices.

Developed by [Lars Grove Mortensen](https://github.com/LarsGMortensen) and contributors.

**CitOmni - Build explicitly. Run efficiently.**
