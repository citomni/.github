# CitOmni

**Low overhead. High performance. Ready for anything.**

CitOmni is a modular PHP 8.2+ framework built around deterministic execution, zero-magic boot, and measurable green performance.  
It scales from tiny shared-hosting sites to enterprise-grade automation without ever changing its philosophy: *explicit, deterministic, fast.*

---

## ⚙️ What it is

A minimal, composable foundation for PHP applications:

| Layer | Package | Purpose |
|-------|----------|----------|
| 🧠 Core | [`citomni/kernel`](https://github.com/citomni/kernel) | Deterministic config + service map |
| 🌐 HTTP | [`citomni/http`](https://github.com/citomni/http) | Router / Request / Response |
| 💻 CLI | [`citomni/cli`](https://github.com/citomni/cli) | Command runner / Scheduler |
| 🧩 Providers | [`citomni/*`](https://github.com/orgs/citomni/repositories) | Auth / Infra / Testing / etc. |

Two runtime modes. One philosophy.  
-> [Runtime Modes explained ›](https://github.com/citomni/docs/blob/main/concepts/runtime-modes.md)

---

## 🧩 Architecture in one sentence

CitOmni merges **static constants** - not runtime code.

```

Vendor baseline  ->  Providers  ->  App base  ->  Env overlay

````

* Config: deep, last-wins  
* Services: left-wins (ID override surface)  
* Routes: associative, per-path merge  

Providers are pure declarative overlays - no reflection, no scanning, no surprises.  
-> [Providers & Services ›](https://github.com/citomni/docs/blob/main/concepts/services-and-providers.md)

---

## 💚 Green by design

CitOmni minimizes CPU cycles, allocations, and file I/O at the framework layer.

| Metric | Result (shared hosting, PHP 8.2 + OPcache) |
|:--|:--|
| Exec time | **≈ 0.004 s** |
| Peak memory | **≈ 0.5 MB** |
| Included files | **24** |
| Est. throughput | **~175 RPS per worker @ 70 % utilization** |

-> [Full report: *Capacity & Green by Design* (2025-10-02)](https://github.com/citomni/docs/blob/main/reports/2025-10-02-capacity-and-green-by-design.md)

> Fast software *is* green software - fewer joules per request, fewer servers, smaller footprint.

---

## 🚀 Quick start

```bash
composer create-project citomni/http-skeleton my-app
cd my-app
php -S 127.0.0.1:8080 -t public
````

Visit [http://127.0.0.1:8080](http://127.0.0.1:8080)
Append `?_perf` to any URL in dev to see timing, memory, and file metrics.

More examples -> [citomni/http-skeleton](https://github.com/citomni/http-skeleton)

---

## 📚 Documentation

* [Runtime / Execution Modes](https://github.com/citomni/docs/blob/main/concepts/runtime-modes.md)
* [Services & Providers](https://github.com/citomni/docs/blob/main/concepts/services-and-providers.md)
* [Capacity & Green by Design Report](https://github.com/citomni/docs/blob/main/reports/2025-10-02-capacity-and-green-by-design.md)
* [Conventions & Contribution Guide](https://github.com/citomni/docs/blob/main/contribute/CONVENTIONS.md)

---

## 🧠 Philosophy

1. **Low overhead** - no bloat, no runtime guessing.
2. **High performance** - deterministic paths, cache-friendly arrays.
3. **Ready for anything** - HTTP / CLI / automation, same core.

> CitOmni doesn't guess. It knows.

---

© 2012-present [Lars Grove Mortensen](https://github.com/LarsGMortensen) - MIT license (core packages).
Docs © CitOmni Contributors, CC BY-SA 4.0.
