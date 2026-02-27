# RFC 113: Public roadmap updates

- RFC: 113
- Author: Thibaud Colas
- Created: 2026-01-27
- Last Modified: 2026-01-27

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. For context, see [past roadmap-focused RFCs](https://github.com/wagtail/rfcs/pulls?q=is%3Apr+label%3Aroadmap) and the [Wagtail release schedule](https://github.com/wagtail/wagtail/wiki/Release-schedule).

## Version number for the next release

Confirmed version number: v7.4 LTS (minor release), in May 2026 based on discussions to date.

## Roadmap for the previous release

Here is the status of roadmap items for the latest release, v7.3 (February 2026):

| Roadmap item                                                               | Status | Notes                          |
| -------------------------------------------------------------------------- | ------ | ------------------------------ |
| [Autosave MVP](https://github.com/wagtail/roadmap/issues/47)               | Done   | UX follow-ups on the roadmap   |
| [Block settings](https://github.com/wagtail/roadmap/issues/103)            | Done   | Follow-ups outside the roadmap |
| [Model search improvements](https://github.com/wagtail/roadmap/issues/116) | v7.4\* |                                |
| [llms.txt for Wagtail docs](https://github.com/wagtail/roadmap/issues/117) | Done   | Follow-ups outside the roadmap |
| [AI checker concepts](https://github.com/wagtail/roadmap/issues/118)       | Done   | Follow-up in new roadmap item  |

## Roadmap for the next release

Proposed roadmap items for v7.4 LTS (May 2026).

### [Model search improvements](https://github.com/wagtail/roadmap/issues/116)

Incremental improvements to the new [django-modelsearch](https://github.com/wagtail/django-modelsearch) re-implementation of Wagtail search backends. Tentatively including:

- Vector indexing support
- Search permission handling
- Filtering on related objects

### [Customizable page explorer](https://github.com/wagtail/roadmap/issues/102)

Size: M

Support for customizations such as additional columns (and filters), reducing the need for developers to create custom views for pages (see [Custom page listings](https://docs.wagtail.org/en/stable/advanced_topics/customization/custom_page_listings.html#custom-page-listings)). See [Support custom columns / filters on main page explorer #11931](https://github.com/wagtail/wagtail/issues/11931) for more info.

### [Independent security audit](https://github.com/wagtail/roadmap/issues/111)

Size: S

An independent audit of the product and-or project for us to procure. Populate a backlog of security improvements based on audit activities.

### [Package maintainers guide](https://github.com/wagtail/roadmap/issues/108)

Size: XS

Revamp of the existing [Python Package Maintenance Guidelines](https://github.com/wagtail/wagtail/wiki/Python-Package-Maintenance-Guidelines), to add more content, reflect the work on [Wagtail Nest](https://github.com/wagtail-nest), and modernize.

### Autosave UX enhancements

Size: S

Follow-up improvements to the [Autosave MVP](https://github.com/wagtail/roadmap/issues/47) based on user feedback and real-world usage. Tentatively including:

- [Deferred validation for required fields in StreamField](https://github.com/wagtail/wagtail/issues/13699)
- [UI polish based on feedback from v7.3 release](https://github.com/wagtail/wagtail/issues/13863)

### Page editor UX

Size: M

A range of UX improvements to the editing experience, with a focus on addressing known pain points / clearing our backlog of UX issues. To be refined, based on:

- [Issues labelled 'UX'](https://github.com/wagtail/wagtail/issues?q=sort%3Aupdated-desc%20state%3Aopen%20label%3AUX)
- [Issues labelled 'Design system'](https://github.com/wagtail/wagtail/issues?q=sort%3Aupdated-desc+state%3Aopen+label%3A%22component%3ADesign+system%22)

### Content quality checker enhancements

Size: M

Follow-up to [AI checker concepts](https://github.com/wagtail/roadmap/issues/118), in particular implementation of generic qualitative checkers.

In scope:

- [SEO search description automated content check #12252](https://github.com/wagtail/wagtail/issues/12252)
- [Universal content comparison interface #158](https://github.com/wagtail/wagtail-ai/issues/158)
- [Implement checker error highlights within the preview panel #12187](https://github.com/wagtail/wagtail/issues/12187)

## Roadmap for the next+1 release

Proposed roadmap items for v7.5\* (August 2026).

### [SEO power tools](https://github.com/wagtail/roadmap/issues/106)

Size: M

See [Looking for sponsorship: SEO power tools](https://wagtail.org/blog/looking-for-sponsorship-seo-power-tools/). New built-in SEO and content quality assurance features, with opportunities for integration with SEO and analytics tools, as well as generative AI.

### [Choosers UI improvements](https://github.com/wagtail/roadmap/issues/109)

Size: XL

Iterative improvements towards a new design for choosers, which brings them closer to the "Universal listings" UX rolled out across listing views.

### Customizable base page model

Size: M

Introduce an overridable AbstractPage class to allow base page fields to be customized per-project.

References:

- [Per-project base page models #836](https://github.com/wagtail/wagtail/issues/836)
- [RFC 53: Swappable Page Model](https://github.com/wagtail/rfcs/pull/53)
- [Feature request: Swappability of main models #11381](https://github.com/wagtail/wagtail/discussions/11381)

### Starter kit relaunch

Overhaul of the `wagtail start` [support for custom templates](https://wagtail.org/blog/building-a-wagtail-starter-template/) and the [news template](https://github.com/wagtail/news-template) for higher usability and maintainability. Key goals:

- Simpler support for multiple starter kits
- Tooling to guarantee starter kits are always up-to-date with latest releases

## Roadmap for "Future" releases

### [Concurrent editing](https://github.com/wagtail/roadmap/issues/24)

Rename the existing "Autosave" roadmap item to "Concurrent editing", to reflect that [Autosave MVP](https://github.com/wagtail/roadmap/issues/47) achieves a lot of our "autosave" goals. It would be confusing to still have an "Autosave" item on the roadmap.

### [Natural language search](https://github.com/wagtail/roadmap/issues/119)

Move existing item to “Future”. Dependent on completion of [Model search improvements](https://github.com/wagtail/roadmap/issues/116).

### Multilingual content lifecycle

Improve Wagtail’s support for multilingual websites by addressing common workflow, UX, and data‑model pain points around creating, updating, reviewing, and maintaining translated content. Tentatively includes:

- Better workflows for maintaining translations.
  - Simpler access to past translations.
  - Switch page language.
  - Translation history / audit trail.
  - TBC: Simultaneous drafts on both source and target content.
- Better editor UX for translation work: more contextual information, rich text support.
- Improved tooling to compare content across locales (source vs translation, and version‑to‑version).
- TBC: Improvements for workflows involving machine translations.

More information:

- [Improving support for multilingual websites #13693](https://github.com/wagtail/wagtail/discussions/13693)
- [RFC 54: Internationalisation](https://github.com/wagtail/rfcs/blob/main/text/054-internationalisation.md)
- [Wagtail Localize issue tracker](https://github.com/wagtail/wagtail-localize/issues)
- [Wagtail issues tagged 'i18n'](https://github.com/wagtail/wagtail/issues?q=sort%3Aupdated-desc%20is%3Aissue%20is%3Aopen%20label%3Acomponent%3Ai18n)

## Proposed roadmap items to close

None

## Items that didn't make the cut

Here are possible roadmap items that were discussed but not included in the roadmap this time, provided for feedback and for future reference. If one of those items is important to you, please comment! They may be available for external contributions or for a [feature sponsorship](https://wagtail.org/sponsor/).

- Documentation improvements incl. restructure
- Front-end cache invalidation improvements
- Strict CSP compatibility completion
