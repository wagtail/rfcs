# RFC 106: Public roadmap updates

- RFC: 106
- Author: Thibaud Colas
- Created: 2025-01-29
- Last Modified: 2025-02-12

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84). See also RFCs [#86](086-roadmap-updates.md), [#88](088-roadmap-updates.md), [#91](091-roadmap-updates.md), [#98](098-roadmap-updates.md), [#101](101-roadmap-updates.md), [#103](103-roadmap-updates.md).

## Version number for the May 2025 release

With no specific discussion to date, we currently expect the February 2025 release will be version 6.5\*.

\* Provisional version number.

## Review of roadmap items for Wagtail 6.4 (February 2025)

4 of 5 roadmap items will be marked as Done.

The following Wagtail 6.4 roadmap items will be marked as Done, with possible follow-up work outside of the formal roadmap:

- [Admin UI performance improvements](https://github.com/wagtail/roadmap/issues/80)
  - Follow-up outside the roadmap: further performance improvements
- [StreamField blocks drag'n'drop](https://github.com/wagtail/roadmap/issues/85)
  - Follow-up outside the roadmap: likely UI tweaks
- [StreamField blocks preview](https://github.com/wagtail/roadmap/issues/84)
  - Follow-up outside the roadmap: likely UI tweaks
- [Headless improvements roadmap](https://github.com/wagtail/roadmap/issues/91)
  - Follow-up on the roadmap: Headless userbar, headless API improvements

The following Wagtail 6.4 roadmap items will be moved to v6.6\*:

- None

The following Wagtail 6.4 roadmap items will be moved to Future:

- [Autosave support refactorings with Telepath](https://github.com/wagtail/roadmap/issues/47)
  - Switching to _Validation on publish_ as the next step in the auto-save journey.

## Proposed roadmap items for Wagtail 6.5\* (May 2025)

### Validation on publish

Implementation of a frequently-requested feature: save draft versions of pages in an incomplete state, while still enforcing validation when publishing. This is a pre-requisite to implementing auto-save as proposed in [RFC 99: Preliminary work to support auto-save functionality](https://github.com/wagtail/rfcs/pull/99).

For more information, see [RFC 104: Validation on publish](https://github.com/wagtail/rfcs/pull/104).

### Headless API improvements

Implementation of top 3-5 headless improvements as identified in the [2024 headless survey](https://wagtail.org/blog/2024-headless-survey/), with a particular focus on the built-in REST API and documentation.

Currently earmarked tasks:

- New version of [headless support documentation](https://docs.wagtail.org/en/stable/advanced_topics/headless.html)
- [Fix headless Wagtail page URL #9984](https://github.com/wagtail/wagtail/pull/9984)
- [Improvements to API router reverse lookups #11310](https://github.com/wagtail/wagtail/pull/11310)
- [Retain fields parameter when redirecting from find_view of API #12141](https://github.com/wagtail/wagtail/pull/12141)
- [Refactor redirect middleware to use its lookup logic for redirects API #11524](https://github.com/wagtail/wagtail/pull/11524)
- [Override serializing StreamBlocks to JSON api #3454](https://github.com/wagtail/wagtail/issues/3454)
- [Documentation for using the "Dynamic image serve" view for generating image renditions on a headless site #6113](https://github.com/wagtail/wagtail/issues/6113)
- [API should provide access to the HTML-converted versions of rich text fields #2695](https://github.com/wagtail/wagtail/issues/2695)
- [No Image URL in the API for ImageChooserBlock #2087](https://github.com/wagtail/wagtail/issues/2087)
- Some work towards of [Support OpenAPI Schema generation for Wagtail API #6209](https://github.com/wagtail/wagtail/issues/6209)

For more information, see [RFC 100: Enhancing headless support](https://github.com/wagtail/rfcs/pull/100).

### Site settings permissions

A new permission model for sites, where groups can be assigned permissions at the level of individual sites. This better maps organisational structures, compared to the current model of permissions being assigned at the level of the settings model rather than instances.

For more information, see [RFC 105: Site settings permissions](https://github.com/wagtail/rfcs/pull/105).

### CSP compatibility audit

An up-to-date review of [CSP compatibility issues](https://github.com/wagtail/wagtail/issues/1288) in Wagtail, with the view to help resolve them all in future releases.

This will replace the [last feature-by-feature review](https://github.com/wagtail/wagtail/issues/7053) which was 4 years ago. We need a clearer record as we now have CSP [on our roadmap](https://github.com/wagtail/roadmap/issues/92) and [as a GSoC project idea](https://github.com/wagtail/gsoc/blob/main/project-ideas.md#content-security-policy-compatibility). Aside from auditing, this will involve adding findings in Wagtail issues, or logged to a "CSP compatibility" board in GitHub Projects.

And if there is time, here are stretch tasks on our way to the [CSP roadmap item](https://github.com/wagtail/roadmap/issues/92):

- [Updating the bakerydemo CSP](https://github.com/wagtail/bakerydemo/blob/main/.env.example) to be on by default, and document stricter options that are compatible, if possible.
- Updating the developer docs to document current level of CSP support
- Updating guide.wagtail.org or wagtail.org with a CSP

## Proposed roadmap items for Wagtail 6.6\* (August 2025)

### Google Summer of Code 2025

Exact items TBC. Showcasing our participation to [Google Summer of Code](https://summerofcode.withgoogle.com/) on the project roadmap.

### Headless demo site

An official headless demo site – primarily so our contributors and maintainers can more easily work on headless improvements.

For more information, see [RFC 100: Enhancing headless support](https://github.com/wagtail/rfcs/pull/100).

## Proposed roadmap items for "Future" releases

### [Sustainability improvements #72](https://github.com/wagtail/roadmap/issues/72)

Existing item – moving to "Future" to earmark more specific improvements / find a feature sponsor.

### Headless preview & userbar

Adding built-in support for the Wagtail userbar in headless mode, including accessibility checks, and built-in headless preview support.

For more information, see [RFC 100: Enhancing headless support](https://github.com/wagtail/rfcs/pull/100).

## Proposed roadmap items to close

### [Admin interface accessibility improvements #71](https://github.com/wagtail/roadmap/issues/71)

This seems too fuzzy at this stage to be meaningfully considered "done" in any specific release.

## Open questions

### What’s the current plan for the headless demo site?

The primary goal is to be able to demo / test the headless capabilities of Wagtail in a representative site. Demonstrating best practices would be good, where it makes sense.

For long-term maintenance, we aim to keep ongoing efforts as low as possible: no hosted environment, make changes in the [existing bakerydemo](https://github.com/wagtail/bakerydemo) as much as possible (for example server-side API configuration), limit adoption of patterns that are very framework-specific where possible.

Here is the provisional choices made with the vue to repurpose the [bakerydemo-nextjs](https://github.com/thibaudcolas/bakerydemo-nextjs) as this new demo site:

- Keep Next.js, as it’s the most popular option per our survey. Use server-rendered Next.js as it will be a better test of headless Wagtail capabilities.
- Keep TypeScript. Keep using the existing bakerydemo’s styles and template markup wherever possible
- Switch from GraphQL to the built-in Wagtail REST API (more popular and it’s better to demo core features).

Features which are (currently) out of scope:

- Multi-site
- GraphQL
