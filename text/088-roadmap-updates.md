# RFC 088: Public roadmap updates

- RFC: 088
- Author: Thibaud Colas
- Created: 2023-10-11
- Last Modified: 2023-10-11

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84). See also [RFC #86](086-roadmap-updates.md).

## Existing items changing schedule

The following items would carry over from v5.2 to v5.3\*:

- [Universal listings #33](https://github.com/wagtail/roadmap/issues/33)
  - A lot of progress has been made, particularly on the designs, and there is more to do to complete the implementation.
- [Admin accessibility audit #46](https://github.com/wagtail/roadmap/issues/46)
  - The ATAG audit will be completed as part of the v5.2 release, the WCAG audit will be carried over to v5.3.
- [CSP compatibility improvements #61](https://github.com/wagtail/roadmap/issues/61)
  - Further improvements to be made.
- [Telepath everything! #47](https://github.com/wagtail/roadmap/issues/47)
  - Further improvements to be made.
- [Readability checks #50](https://github.com/wagtail/roadmap/issues/50)
  - We’ve made progress on new designs, and haven’t started implementation.
- [Enhanced dashboard #45](https://github.com/wagtail/roadmap/issues/45)
  - We’ve made progress on new designs, and haven’t started implementation.

The following items from our Future milestone would move to v5.3\*:

- [RTL languages support #43](https://github.com/wagtail/roadmap/issues/43)
  - We’ve made progress on new designs and the steps to complete this are now very clear.

The following items would move to v5.4\*:

- [WCAG 2.1 AA for Wagtail admin #27](https://github.com/wagtail/roadmap/issues/27)
  - We will have enough auditing in place to proceed with a large number of improvements.
- [Auto-locking for pages #41](https://github.com/wagtail/roadmap/issues/41)
  - We have a clear idea of the steps to complete this.
- [Enhanced dashboard #45](https://github.com/wagtail/roadmap/issues/45)
  - We expect this to carry over from v5.3\* to v5.4\*.

## Proposed new items for Wagtail v5.3\* (February 2024)

### Wagtail hosting providers

Implementation of the [Wagtail hosting providers](https://github.com/wagtail/wagtail/wiki/Wagtail-Hosting-Providers) content in our documentation and on the Wagtail website.

#### Intended outcome

- Agreed criteria for hosting providers to be listed.
- First batch of providers added to the relevant listings.

### Accessibility checker in page editor

Implementation of [Integrate accessibility checker within the page editor / inline preview #10136](https://github.com/wagtail/wagtail/issues/10136).

#### Intended outcome

- The accessibility checker should be available for people who do not use the Wagtail userbar.
- Accessibility checker results should be available at the point of saving drafts or publishing.

### Sustainability roadmap

As a follow-up to [Sustainability considerations for Wagtail sites #38](https://github.com/wagtail/roadmap/issues/38) and [Google Summer of Code 2023 #59](https://github.com/wagtail/roadmap/issues/59), a review of possible improvements to Wagtail that we could integrate on our roadmap in the future.

#### Intended outcome

- Extensive review of the [Web Sustainability Guidelines](https://w3c.github.io/sustyweb/), with a particular focus on applicability for the Wagtail CMS and Wagtail websites.
- Refinement of a potential program of sustainability improvements to Wagtail, with the view to reduce its [carbon footprint](http://wagtail.org/sustainability/).

### Polish Dark Mode

A small extension to [Dark mode for the admin interface #52](https://github.com/wagtail/roadmap/issues/52). Includes further tweaks to the dark theme. Possibly integration of high-contrast variants of the theme.

### Wagtail Developer Onboarding tutorials

Reflecting our ongoing project [Google Season of Docs: Creating Wagtail Developer Onboarding Tutorials](https://wagtail.org/blog/google-season-of-docs-creating-wagtail-developer-onboarding-tutorials/), in particular with the view of completing this project in time for the February 2024 release.

### Wagtail.org website accessibility

An Outreachy internship happening over this release cycle. See [our blog post](https://wagtail.org/blog/our-outreachy-projects-in-2023/).

### Accessibility features documentation

An Outreachy internship happening over this release cycle. See [our blog post](https://wagtail.org/blog/our-outreachy-projects-in-2023/).

### Adopt generic class based views across all areas of admin

An Outreachy internship happening over this release cycle. See [our blog post](https://wagtail.org/blog/our-outreachy-projects-in-2023/).

## Proposed items scheduled for Wagtail v5.4\* (May 2024)

### Multi-tenancy improvements

A range of improvements for multi-site and multi-tenant Wagtail projects. Exact improvements to be confirmed.

### Sustainability improvements

Based on a roadmap of improvements to Wagtail as defined in the previous release cycle. As a follow-up to [Sustainability considerations for Wagtail sites #38](https://github.com/wagtail/roadmap/issues/38) and [Google Summer of Code 2023 #59](https://github.com/wagtail/roadmap/issues/59), implementation of possible improvements to Wagtail.

### Wagtail Space organization

Reflecting the effort of Wagtail Space NL and Wagtail Space US organizers, with events happening in June.

## New items for "Future" releases

No additions. See [wagtail.org/roadmap/#future](https://wagtail.org/roadmap/#future) for a full list of items parked under "Future".
