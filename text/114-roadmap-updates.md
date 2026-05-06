# RFC 114: Public roadmap updates

- RFC: 114
- Author: Thibaud Colas
- Created: 2026-05-05
- Last Modified: 2026-05-05

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

### [SEO power tools](https://github.com/wagtail/roadmap/issues/106)

Size: M

See [Looking for sponsorship: SEO power tools](https://wagtail.org/blog/looking-for-sponsorship-seo-power-tools/). New built-in SEO and content quality assurance features, with opportunities for integration with SEO and analytics tools, as well as generative AI.

### [Choosers UI improvements](https://github.com/wagtail/roadmap/issues/109)

Size: XL

Iterative improvements towards a new design for choosers, which brings them closer to the "Universal listings" UX rolled out across listing views.

### [Starter kit relaunch](https://github.com/wagtail/roadmap/issues/124)

Size: M

Overhaul of the `wagtail start` support for custom templates and the news template for higher usability and maintainability.

### [Customizable base page model](https://github.com/wagtail/roadmap/issues/126)

Size: M

Introduce an overridable AbstractPage class to allow base page fields to be customized per-project.

## Roadmap for the next+1 release

TBC

## Roadmap for "Future" releases

TBC

## Proposed roadmap items to close

### [Draftail for general text entry](https://github.com/wagtail/roadmap/issues/26)

> Switch to a rich text style UI for plain text inputs, to support more advanced text interactions.

This kind of switch is still desirable but Draftail is likely not the right foundation as the underlying Draft.js library is [unmaintained since 2022](https://github.com/wagtail/draftail/issues/456).

### [Sustainability improvements](https://github.com/wagtail/roadmap/issues/72)

> Improvements across three key areas for Wagtail sites’ carbon footprints: measurements, awareness, tangible reductions.

Most of the improvements tracked in the roadmap item have been happening incrementally, and the remaining ones will be better served by more precise tracking either on the roadmap or directly in our feature backlog.

### [Fully accessible admin](https://github.com/wagtail/roadmap/issues/27)

> Work towards making [Wagtail’s accessibility](https://wagtail.org/accessibility/) world-class – stellar feedback from users of assistive technologies on the admin interface. Full compliance with relevant standards: WCAG, ATAG.

This is the right long-term goal but it’s not served well by being on the roadmap in "Future" with a vague roadmap item. We’re making continuous improvements to the accessibility of Wagtail, so this work is happening anyway.

## Items that didn't make the cut

TBC
