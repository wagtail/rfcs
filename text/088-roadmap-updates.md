# RFC 088: Public roadmap updates

- RFC: 088
- Author: Thibaud Colas
- Created: 2023-10-11
- Last Modified: 2023-10-25

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84). See also [RFC #86](086-roadmap-updates.md).

## Version number for the February 2024 release

After discussion within the core team, we agreed to have the February 2024 release as Wagtail 6.0 (major version). This will make it possible to remove backwards-compatibility code for older versions of Elasticsearch.

## Existing items changing schedule

The following items would carry over from v5.2 to v6.0:

- [Telepath everything! #47](https://github.com/wagtail/roadmap/issues/47) – re-titled to "Telepath for auto-save"
  - Further improvements to be made. This is a precursor to auto-save work which we still see as a good strategic investment, but has been side-lined in past releases.

The following items from our Future milestone would move to v6.0:

- [RTL languages support #43](https://github.com/wagtail/roadmap/issues/43)
  - We’ve made progress on new designs and the steps to complete this are now very clear.

The following items would move to v6.1\*:

- [Auto-locking for pages #41](https://github.com/wagtail/roadmap/issues/41)
  - We have a clear idea of the steps to complete this.
 
The following items would move from v5.2 to Future:

- [Readability checks #50](https://github.com/wagtail/roadmap/issues/50).
  - We’ve made progress on new designs, and decided to take the concept well beyond readability checks. Readability checks remain a key element of this but the capabilities need to be more generic. For further details, see [Content quality checkers #11063](https://github.com/wagtail/wagtail/discussions/11063).
- [Enhanced dashboard #45](https://github.com/wagtail/roadmap/issues/45)
  - We’ve made progress on new designs, and will proceed with implementation once the designs have been reviewed, with no set schedule until then. See [Wagtail dashboard enhancements #8325](https://github.com/wagtail/wagtail/discussions/8325) for latest progress.

## Follow-ups to past roadmap items in v6.0

### Universal listings improvements

Follow-up to [Universal listings #33](https://github.com/wagtail/roadmap/issues/33). A lot of progress has been made, particularly on the designs, and there is more to do to complete the implementation.

See [Universal listings (search and filtering) #10446](https://github.com/wagtail/wagtail/discussions/10446#discussioncomment-7302866) in GitHub Discussions for the latest update on progress, and [static-wagtail-v5-2](https://static-wagtail-v5-2.netlify.app/admin/pages/60/) for a static demo.

### WCAG 2.2 manual audit

Follow-up to [Admin accessibility audit #46](https://github.com/wagtail/roadmap/issues/46). In the last release, we focused on auditing all aspects of ATAG aside from Guideline _A.1.1: Ensure that web-based functionality is accessible_, which is a matter of WCAG conformance. We did automated testing of the Wagtail admin, and now have manual tests to go through.

See:

- [Wagtail 5.1 UI overview](https://docs.google.com/spreadsheets/d/1FMSA_BI3ZvkeAvuaIL2QtqRTgMNwz_vhfKBeyx2Onnk/edit)
- [ATAG 2.0 audit](https://wagtail.org/accessibility/atag-audit/)
- [WCAG 2.2 AAA* audit of the Wagtail admin](https://github.com/wagtail/wagtail/discussions/11180)

### Sustainability roadmap

As a follow-up to [Sustainability considerations for Wagtail sites #38](https://github.com/wagtail/roadmap/issues/38) and [Google Summer of Code 2023 #59](https://github.com/wagtail/roadmap/issues/59), a review of possible improvements to Wagtail that we could integrate on our roadmap in the future.

#### Intended outcome

- Extensive review of the [Web Sustainability Guidelines](https://w3c.github.io/sustyweb/), with a particular focus on applicability for the Wagtail CMS and Wagtail websites.
- Refinement of a potential program of sustainability improvements to Wagtail, with the view to reduce its [carbon footprint](http://wagtail.org/sustainability/).

### Polish Dark Mode

A small extension to [Dark mode for the admin interface #52](https://github.com/wagtail/roadmap/issues/52). Includes further tweaks to the dark theme. Possibly integration of high-contrast variants of the theme.

## Proposed new items for Wagtail v6.0 (February 2024)

### Accessibility checker in page editor

Implementation of [Integrate accessibility checker within the page editor / inline preview #10136](https://github.com/wagtail/wagtail/issues/10136).

#### Intended outcome

- The accessibility checker should be available for people who do not use the Wagtail userbar.
- Accessibility checker results should be available at the point of saving drafts or publishing.

### Wagtail Developer Onboarding tutorials

Reflecting our ongoing project [Google Season of Docs: Creating Wagtail Developer Onboarding Tutorials](https://wagtail.org/blog/google-season-of-docs-creating-wagtail-developer-onboarding-tutorials/), in particular with the view of completing this project in time for the February 2024 release.

### Wagtail.org website accessibility

An Outreachy internship happening over this release cycle. See [our blog post](https://wagtail.org/blog/our-outreachy-projects-in-2023/).

### Accessibility features documentation

An Outreachy internship happening over this release cycle. See [our blog post](https://wagtail.org/blog/our-outreachy-projects-in-2023/).

### Adopt generic class based views across all areas of admin

An Outreachy internship happening over this release cycle. See [our blog post](https://wagtail.org/blog/our-outreachy-projects-in-2023/).

## Proposed new items scheduled for Wagtail v6.1\* (May 2024)

### Admin accessibility improvements

Remedial work as part of a long-term goal: [WCAG 2.1 AA for Wagtail admin #27](https://github.com/wagtail/roadmap/issues/27).

### Sustainability improvements

Based on a roadmap of improvements to Wagtail as defined in the previous release cycle. As a follow-up to [Sustainability considerations for Wagtail sites #38](https://github.com/wagtail/roadmap/issues/38) and [Google Summer of Code 2023 #59](https://github.com/wagtail/roadmap/issues/59), implementation of possible improvements to Wagtail.

### Wagtail Space organization

Reflecting the effort of Wagtail Space NL and Wagtail Space US organizers, with events happening in June.

## New items for "Future" releases

> See [wagtail.org/roadmap/#future](https://wagtail.org/roadmap/#future) for a full list of items parked under "Future".

### Multi-tenancy improvements

A range of improvements for multi-site and multi-tenant Wagtail projects. Exact improvements to be confirmed.

