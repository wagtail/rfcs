# RFC 114: Public roadmap updates

- RFC: 114
- Author: Thibaud Colas
- Created: 2026-05-05
- Last Modified: 2026-06-09

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases.
For context, see [past roadmap-focused RFCs](https://github.com/wagtail/rfcs/pulls?q=is%3Apr+label%3Aroadmap) and the [Wagtail release schedule](https://github.com/wagtail/wagtail/wiki/Release-schedule).

## Version number for the next release

Provisional version number: v8.0 (major release), in August 2026 based on discussions to date.

## Roadmap for the previous release

Here is the status of roadmap items for the latest release, v7.4 LTS (May 2026):

| Roadmap item                                                                          | Status | Notes                                     |
| ------------------------------------------------------------------------------------- | ------ | ----------------------------------------- |
| [Model search improvements](https://github.com/wagtail/roadmap/issues/116)            | Done   | Shipped as part of django-modelsearch 1.3 |
| [Customizable page explorer](https://github.com/wagtail/roadmap/issues/102)           | Done   |                                           |
| [Independent security audit](https://github.com/wagtail/roadmap/issues/111)           | Done   |                                           |
| [Package maintainers guide](https://github.com/wagtail/roadmap/issues/108)            | Done   |                                           |
| [Autosave UX enhancements](https://github.com/wagtail/roadmap/issues/127)             | Done   |                                           |
| [Page editor UX](https://github.com/wagtail/roadmap/issues/123)                       | Done   |                                           |
| [Content quality checker enhancements](https://github.com/wagtail/roadmap/issues/125) | Done   | Follow-ups in core and packages           |

## Roadmap for the next release

Proposed roadmap items for v8.0 (August 2026).

### [Customizable base page model](https://github.com/wagtail/roadmap/issues/126)

Size: XL

Introduce an overridable AbstractPage class to allow base page fields to be customized per-project.

### Write API

Size: XL

A new official CMS API that supports programmatic creation and modification of Wagtail-managed content, including pages, snippets, revisions, and editorial workflow state transitions. The goal is to support integrations, automation, structured content workflows, and AI-assisted tooling while preserving Wagtail’s editorial governance, permissions, accessibility, and auditability guarantees.

See [RFC 115: write API](https://github.com/wagtail/rfcs/pull/115).

### [Starter kit relaunch](https://github.com/wagtail/roadmap/issues/124)

Size: M

Overhaul of the `wagtail start` support for custom templates and the news template for higher usability and maintainability.

### Cyber Resilience Act readiness

Size: S

We want to improve our processes around vulnerability disclosure and supply chain management, ahead of the EU Cyber Resilience Act. For now the focus will be on improvements that are generally valuable for a wide range cybersecurity requirements, such as:

- Producing SBOMs for our releases, and signing release artifacts
- Revising our [security policy](https://docs.wagtail.org/en/stable/contributing/security.html), in particular with more information on our behind-the-scenes security team process
- Signposting and sharing how to do this as part of the package ecosystem, via changes to our [package guidelines](https://wagtail.org/package-guidelines/).

### [Demo website redesign](https://github.com/wagtail/bakerydemo/issues/566)

An incremental redesign of the [Wagtail bakery demo site](https://github.com/wagtail/bakerydemo?rgh-link-date=2025-10-23T15%3A14%3A12.000Z), with the intention to make it a more suitable demo for larger projects, and other verticals than breadmaking. This is done as a Google Summer of Code project, started in June 2026.

## Roadmap for the next+1 release

### [SEO power tools](https://github.com/wagtail/roadmap/issues/106)

Size: M

See [Looking for sponsorship: SEO power tools](https://wagtail.org/blog/looking-for-sponsorship-seo-power-tools/). New built-in SEO and content quality assurance features, with opportunities for integration with SEO and analytics tools, as well as generative AI.

## Roadmap for "Future" releases

### [Choosers UI improvements](https://github.com/wagtail/roadmap/issues/109)

Size: XL

Iterative improvements towards a new design for choosers, which brings them closer to the "Universal listings" UX rolled out across listing views.

## Proposed roadmap items to close

None

## Items that didn't make the cut

None
