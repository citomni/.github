# CitOmni Licensing

CitOmni is a collection of independently versioned Composer packages, project skeletons, documentation, and supporting repositories. **The CitOmni name does not imply one universal license for everything under the organization.** The applicable rights depend on the actual repository and the material being used.

This document explains the licensing model and how to determine the correct terms. It is a practical overview, **not** a substitute for the applicable license text or a grant of additional rights.

---

## The authoritative rule

**Read the `LICENSE` file in the repository and version you intend to use.** Also check its `NOTICE` and third-party license files when they are present.

- A repository's own `LICENSE` is authoritative for the material it covers.
- The Composer `license` field identifies package license metadata but does not replace the license text.
- A package's source and license at the installed version take precedence over organization-wide summaries or older documentation.
- Public visibility on GitHub is not, by itself, permission to copy, modify, or redistribute a repository without an applicable license.
- A private or access-controlled CitOmni package does not automatically inherit MIT or Creative Commons terms from public packages.

The source repository and its specific release are the unit of verification. The tables below describe licenses that can be verified in publicly accessible repositories, not an irrevocable promise about future releases.

---

## License overview

| Repository category | Verified examples | License or status |
|---|---|---|
| Application core and delivery | [`kernel`](https://github.com/citomni/kernel), [`http`](https://github.com/citomni/http), [`cli`](https://github.com/citomni/cli), [`infrastructure`](https://github.com/citomni/infrastructure) | MIT |
| Reusable capability packages | [`image`](https://github.com/citomni/image), [`upload`](https://github.com/citomni/upload), [`installer`](https://github.com/citomni/installer), [`jobrunner`](https://github.com/citomni/jobrunner) | MIT |
| AI and data integrations | [`helloai`](https://github.com/citomni/helloai), [`vectorembedding`](https://github.com/citomni/vectorembedding), [`knowledgebase`](https://github.com/citomni/knowledgebase), [`danish-public-sector-data`](https://github.com/citomni/danish-public-sector-data) | MIT |
| Project and provider skeletons | [`app-skeleton`](https://github.com/citomni/app-skeleton), [`provider-skeleton`](https://github.com/citomni/provider-skeleton), [`http-skeleton`](https://github.com/citomni/http-skeleton) | MIT |
| Dedicated documentation | [`citomni/docs`](https://github.com/citomni/docs) | Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0) |
| Organization documentation and policies | [`citomni/.github`](https://github.com/citomni/.github) | CC BY-SA 4.0 for documentation, as stated in its `LICENSE` |
| Uninitialized repositories | [`citomni/pdf`](https://github.com/citomni/pdf), [`citomni/pdf-viewer`](https://github.com/citomni/pdf-viewer) | No readable `LICENSE` while empty; no open-source license should be inferred |
| Access-controlled or proprietary projects | Any non-public or separately distributed CitOmni project | Check that project's actual terms; no organization-wide default license applies |

A repository name in the table does **not** certify production readiness, package publication, or ongoing support. For a current inventory, visit the [GitHub organization](https://github.com/orgs/citomni/repositories).

### Scope of documentation licensing

The standalone documentation in `citomni/docs` and the documentation/policy content in `citomni/.github` use CC BY-SA 4.0. This **does not mean** every README or code example anywhere in the organization is automatically licensed under CC BY-SA.

Documentation distributed as part of an MIT-licensed code package is ordinarily covered by that package's own license unless another file or notice states otherwise. Likewise, the `.github` repository's `LICENSE` expressly describes its scope as documentation; do not assume unrelated software or third-party assets are covered without checking their terms.

---

## MIT-licensed code and skeletons

The MIT License is a permissive software license. Subject to its conditions, it allows use, copying, modification, distribution, sublicensing, and commercial use of the licensed software, including integration into proprietary applications.

The important obligations include:

- Retain the required copyright and permission notices in copies or substantial portions of the software.
- Preserve other applicable notices and third-party terms for bundled or redistributed materials.
- Understand that MIT-licensed software is provided **without warranty**, as specified in the license.

MIT does **not** require an application using an MIT dependency to become open source. It also does **not** transfer ownership of the CitOmni name or logo to downstream users.

Use Composer to install framework packages. Copying the code into application source is neither required nor the supported CitOmni integration model.

### Skeletons and generated applications

CitOmni skeleton repositories are intended as starting points for applications and provider packages. Their MIT license generally permits developers to modify the resulting application, including for commercial projects, while complying with the applicable copyright and license notice conditions.

The original skeleton license and notices should not be silently deleted merely because the project name, namespace, or application purpose has changed. Runtime packages installed through Composer retain their own license terms independently of the application's own licensing choice.

The older `citomni/http-skeleton` repository is still MIT-licensed, but its presence in the license overview is **not** a recommendation to use its legacy runtime setup for new projects.

---

## CC BY-SA 4.0 documentation

The [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/) permits sharing and adaptation of the licensed material, including commercial use, subject to its terms.

When redistributing licensed documentation or adaptations, the important requirements include:

1. **Attribution.** Give appropriate credit and retain required notices.
2. **License information.** Provide a link or reference to the applicable CC BY-SA 4.0 license.
3. **Modification notice.** Indicate whether changes were made.
4. **ShareAlike.** Distribute adapted material under the same license, or another license expressly permitted by CC BY-SA 4.0.
5. **No additional restrictions.** Do not impose extra legal or technological restrictions that prevent recipients from exercising the licensed rights.

The precise conditions and exceptions are defined by the license itself. Read the linked official text before republishing documentation in a way that depends on these rights.

For example, adapting a substantial section of a `citomni/docs` guide into another publication may trigger attribution and ShareAlike obligations for the adaptation. Merely linking to the original guide is different from copying or adapting its content.

A CC BY-SA documentation license does not grant permission to present an unofficial project as an official CitOmni product.

---

## Access-controlled and proprietary packages

Some applications, internal tools, and separately distributed extensions may have licensing arrangements that differ from public CitOmni packages.

- Access to source code is not equivalent to permission to redistribute it.
- Do not infer the license of `citomni/authenticate`, `citomni/devkit`, `citomni/admin`, `citomni/testing`, or any other access-controlled project from the license of `citomni/kernel` or from a past organization summary.
- Consult the project's actual `LICENSE`, distribution agreement, or written permission before use outside its authorized scope.
- Do not describe an access-controlled package as MIT-licensed, proprietary, or distributed through a particular channel without evidence from its current terms.

A package can be technically compatible with the framework without sharing its license. The package's ownership, redistribution rights, and support commitments are separate questions.

---

## Third-party dependencies and bundled assets

The repository's license covers only material that its rights holders are entitled to license. External dependencies, fonts, images, datasets, generated resources, copied snippets, and other third-party materials can carry their **own** licenses and attribution requirements.

Before adding or redistributing third-party material:

- Check the upstream license and whether the intended copying, modification, distribution, and commercial use are permitted.
- Record the dependency in Composer metadata when it is a Composer package, rather than vendoring a second copy without a concrete reason.
- Preserve copyright statements, license files, required notices, and attribution for redistributed assets.
- Check copyleft, network-use, patent, and other obligations where relevant. A dependency is not automatically suitable merely because it is publicly accessible or widely used.
- Add or update `THIRD-PARTY-NOTICES.md` or other repository-specific notices when required by the material being included.
- Treat datasets, source documents, and API responses separately from software code; public availability does not imply unrestricted reuse.

A downstream application must also comply with the licenses of its **transitive dependencies**, not only the direct CitOmni package it installs.

---

## Contributions and license compatibility

Contributions to a repository are intended to be supplied under that repository's applicable license unless its maintainers have explicitly established a different process.

- Contribute only original work or material you have permission to license on those terms.
- Preserve attribution for material from another source and disclose relevant third-party licensing in the pull request.
- For an MIT-licensed package, do not add third-party code with incompatible redistribution requirements without a deliberate licensing review.
- For CC BY-SA documentation, ensure copied or adapted material may legally be distributed under the documentation license.
- For proprietary or otherwise restricted projects, confirm contribution terms with the maintainers rather than assuming the public-project policy applies.

The organization-wide [pull request template](PULL_REQUEST_TEMPLATE.md) requests a Developer Certificate of Origin (DCO) sign-off. The procedure is described in [CONTRIBUTING.md](CONTRIBUTING.md). A DCO sign-off is a certification about the contributor's right to submit the work; it is not, by itself, a copyright assignment or a replacement for the target repository's license.

---

## SPDX identifiers and repository metadata

Use identifiers that match the **actual license of the target repository and file**.

| License | SPDX identifier | Composer `license` value when applicable |
|---|---|---|
| MIT | `MIT` | `"MIT"` |
| CC BY-SA 4.0 | `CC-BY-SA-4.0` | `"CC-BY-SA-4.0"` for an applicable published package, if used |
| Proprietary | No universal SPDX identifier implied | `"proprietary"` when appropriate |

For example, a PHP source file in an MIT-licensed CitOmni package may use a short notice containing:

```php
/*
 * SPDX-License-Identifier: MIT
 * Copyright (c) <applicable years> <rights holder>
 */
```

Replace the placeholders with the **actual** copyright information for that file or contribution. Do not copy a fixed author name or copyright start year into unrelated files without verifying ownership.

For standalone documentation licensed under CC BY-SA 4.0, a file may carry a short documentation notice with `SPDX-License-Identifier: CC-BY-SA-4.0` and a link to the repository's license. A boilerplate comment is not required at the top of every Markdown document merely to make the repository's existing license effective.

Metadata, headers, `NOTICE` files, and license text should agree. If they disagree, resolve the inconsistency rather than assuming metadata overrides the actual grant of rights.

---

## Trademarks and attribution

**Copyright permissions and trademark permissions are separate.** The CitOmni name and logo are trademarks of **Lars Grove Mortensen**, as stated in the organization's [Trademark Policy](TRADEMARKS.md) and [NOTICE](NOTICE).

The project's trademark policy explains when descriptive references such as *"Compatible with CitOmni"* are permitted and when permission is required for product names, vendor names, domains, logos, or suggestions of official endorsement.

Using MIT-licensed CitOmni code does not automatically authorize marketing another product as an official CitOmni release. For permission requests and current restrictions, consult the [Trademark Policy](TRADEMARKS.md).

---

## Practical license checklist

When **consuming a package**, confirm the version, its `LICENSE`, required notices, and third-party dependencies. When **contributing a change**, confirm the target repository's license, your right to submit the work, and any new dependency obligations. When **publishing a derived work**, retain notices and satisfy the license conditions applicable to each incorporated component.

If a repository is empty, has no readable license, or contains conflicting license information, **do not infer a permission grant from its name or visibility**. Resolve the licensing question before redistributing material.

---

## Related policies

- [Contributing to CitOmni](CONTRIBUTING.md)
- [Security Policy](SECURITY.md)
- [Trademark Policy](TRADEMARKS.md)
- [Organization NOTICE](NOTICE)
- [CitOmni GitHub organization](https://github.com/orgs/citomni/repositories)

---

CitOmni - low overhead, high performance, ready for anything.
