# RFC 088: Public roadmap updates

- RFC: 088
- Author: Thibaud Colas
- Created: 2023-10-11
- Last Modified: 2023-10-25

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84). See also [RFC #86](086-roadmap-updates.md).

## Existing items changing schedule

The following items would carry over from v5.2 to v5.3\*:

- [CSP compatibility improvements #61](https://github.com/wagtail/roadmap/issues/61)
  - Further improvements to be made. This is largely driven by ongoing refactorings to reduce technical debt, there is no plan for this to be completed in v5.3\*. Alternatively, this could be removed from the roadmap and still happen behind the scenes.
- [Telepath everything! #47](https://github.com/wagtail/roadmap/issues/47)
  - Further improvements to be made. This is a precursor to auto-save work which we still see as a good strategic investment, but has been side-lined in past releases.
- [Enhanced dashboard #45](https://github.com/wagtail/roadmap/issues/45)
  - We’ve made progress on new designs, and will proceed with implementation once the designs have been reviewed. The v5.3\* release would contain a "MVP" implementation of the new designs, with completion of further dashboard enhancements in v5.4\*. See [Wagtail dashboard enhancements #8325](https://github.com/wagtail/wagtail/discussions/8325) for latest progress.

The following items from our Future milestone would move to v5.3\*:

- [RTL languages support #43](https://github.com/wagtail/roadmap/issues/43)
  - We’ve made progress on new designs and the steps to complete this are now very clear.

The following items would move to v5.4\*:

- [WCAG 2.1 AA for Wagtail admin #27](https://github.com/wagtail/roadmap/issues/27)
  - We will have enough auditing in place to proceed with a large number of improvements.
- [Auto-locking for pages #41](https://github.com/wagtail/roadmap/issues/41)
  - We have a clear idea of the steps to complete this.

## Follow-ups to past roadmap items in v5.3\*

### Content quality checks

Follow-up to [Readability checks #50](https://github.com/wagtail/roadmap/issues/50). We’ve made progress on new designs, and decided to take the concept well beyond readability checks. Readability checks remain a key element of this but the capabilities need to be more generic. For further details, see [Content quality checkers #11063](https://github.com/wagtail/wagtail/discussions/11063).

### Universal listings improvements

Follow-up to [Universal listings #33](https://github.com/wagtail/roadmap/issues/33). A lot of progress has been made, particularly on the designs, and there is more to do to complete the implementation.

See [Universal listings (search and filtering) #10446](https://github.com/wagtail/wagtail/discussions/10446#discussioncomment-7302866) in GitHub Discussions for the latest update on progress, and [static-wagtail-v5-2](https://static-wagtail-v5-2.netlify.app/admin/pages/60/) for a static demo.

### WCAG 2.2 AAA manual audit

Follow-up to [Admin accessibility audit #46](https://github.com/wagtail/roadmap/issues/46). In the last release, we focused on auditing all aspects of ATAG aside from Guideline _A.1.1: Ensure that web-based functionality is accessible_, which is a matter of WCAG conformance. We did automated testing of the Wagtail admin, and now have manual tests to go through.

See:

- [Wagtail 5.1 UI overview](https://docs.google.com/spreadsheets/d/1FMSA_BI3ZvkeAvuaIL2QtqRTgMNwz_vhfKBeyx2Onnk/edit)
- [ATAG 2.0 audit (draft)](https://gist.github.com/thibaudcolas/c48b0b4cf8e7966cd09d22677ab63173)

### Sustainability roadmap

As a follow-up to [Sustainability considerations for Wagtail sites #38](https://github.com/wagtail/roadmap/issues/38) and [Google Summer of Code 2023 #59](https://github.com/wagtail/roadmap/issues/59), a review of possible improvements to Wagtail that we could integrate on our roadmap in the future.

#### Intended outcome

- Extensive review of the [Web Sustainability Guidelines](https://w3c.github.io/sustyweb/), with a particular focus on applicability for the Wagtail CMS and Wagtail websites.
- Refinement of a potential program of sustainability improvements to Wagtail, with the view to reduce its [carbon footprint](http://wagtail.org/sustainability/).

### Polish Dark Mode

A small extension to [Dark mode for the admin interface #52](https://github.com/wagtail/roadmap/issues/52). Includes further tweaks to the dark theme. Possibly integration of high-contrast variants of the theme.

## Proposed new items for Wagtail v5.3\* (February 2024)

### Wagtail hosting providers

Implementation of the [Wagtail hosting providers](https://github.com/wagtail/wagtail/wiki/Wagtail-Hosting-Providers) content in our documentation and on the Wagtail website. This will consist of the wiki page detailing our process, and the website page listing providers.

#### Intended outcome

- Agreed criteria for hosting providers to be listed.
- First batch of providers added to the relevant listings.

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

## Proposed items scheduled for Wagtail v5.4\* (May 2024)

### Dashboard improvements

Follow-up to [Enhanced dashboard #45](https://github.com/wagtail/roadmap/issues/45).

### Multi-tenancy improvements

A range of improvements for multi-site and multi-tenant Wagtail projects. Exact improvements to be confirmed.

### Sustainability improvements

Based on a roadmap of improvements to Wagtail as defined in the previous release cycle. As a follow-up to [Sustainability considerations for Wagtail sites #38](https://github.com/wagtail/roadmap/issues/38) and [Google Summer of Code 2023 #59](https://github.com/wagtail/roadmap/issues/59), implementation of possible improvements to Wagtail.

### Wagtail Space organization

Reflecting the effort of Wagtail Space NL and Wagtail Space US organizers, with events happening in June.

## New items for "Future" releases

No additions. See [wagtail.org/roadmap/#future](https://wagtail.org/roadmap/#future) for a full list of items parked under "Future".
