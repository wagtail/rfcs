# RFC 091: Public roadmap updates

- RFC: 091
- Author: Thibaud Colas
- Created: 2024-01-26
- Last Modified: 2024-01-29

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84). See also [RFC #86](086-roadmap-updates.md), [RFC #88](088-roadmap-updates.md).

## Version number for the May 2024 release

With no specific discussion to date, we currently expect the May 2024 release will be version 6.1\*.

\* Provisional version number.

## Review of roadmap items for Wagtail 6.0 (February 2024)

The following Wagtail 6.0 roadmap items will be marked as Done:

- [Right-To-Left languages support #43](https://github.com/wagtail/roadmap/issues/43)
- [Accessibility manual audit - WCAG 2.2 #63](https://github.com/wagtail/roadmap/issues/63)
- [Sustainability roadmap #64](https://github.com/wagtail/roadmap/issues/64)
- [Polish Dark Mode #65](https://github.com/wagtail/roadmap/issues/65)
- [Accessibility checker in page editor #66](https://github.com/wagtail/roadmap/issues/66)
- [Wagtail Developer Onboarding tutorials #67](https://github.com/wagtail/roadmap/issues/67)
- [Wagtail.org website accessibility #68](https://github.com/wagtail/roadmap/issues/68)
  - This Outreachy internship is still under way but the work will be completed much earlier than the May 2024 release.
- [Accessibility features documentation #69](https://github.com/wagtail/roadmap/issues/69)
  - This Outreachy internship is still under way but the work will be completed much earlier than the May 2024 release.
- [Adopt generic class based views across all areas of admin #70](https://github.com/wagtail/roadmap/issues/70)
  - This was planned as an Outreachy internship but the internship didn’t happen due to funding issues.
  - The large majority of the refactoring has happened anyway, with only a few holdouts that can happen without a roadmap item.

The following Wagtail 6.0 roadmap items will be marked as Done and there will be a follow-up item in v6.1\*:

- [Universal listings improvements #62](https://github.com/wagtail/roadmap/issues/62)
  - Universal listings search and filter MVP features have been implemented for pages and snippets. There are more features to add, and the new listings need to be rolled out in other areas of the CMS.

The following Wagtail 6.0 roadmap items will be moved to v6.1\* as-is:

- [Auto-save support refactorings with Telepath #47](https://github.com/wagtail/roadmap/issues/47) – (previously "Telepath for auto-save", "Telepath everything!")
  - Further improvements to be made. This is a precursor to auto-save work which we still see as a good strategic investment, but has been side-lined in past releases.

There will be no roadmap item moving to v6.2\* or Future milestones.

## Proposed roadmap items for Wagtail 6.1\* (May 2024)

### Universal listings continued

Follow-up to [Universal listings improvements #62](https://github.com/wagtail/roadmap/issues/62) and [Universal listings #33](https://github.com/wagtail/roadmap/issues/33). Universal listings search and filter MVP features have been implemented for pages and snippets. There are more features to add, and the new listings need to be rolled out in other areas of the CMS.

See [Universal listings (search and filtering) #10446](https://github.com/wagtail/wagtail/discussions/10446#discussioncomment-7302866) in GitHub Discussions for the latest update on progress, and [static-wagtail-v6-0](https://static-wagtail-v6-0.netlify.app/admin/pages/60/) for a static demo.

### [Admin interface accessibility improvements #71](https://github.com/wagtail/roadmap/issues/71)

Existing item – already scheduled for Wagtail 6.1\*. Will be refined into a set of 10 core improvements and 10 "stretch" improvements based on the existing [WCAG 2.2 & ATAG 2.0 CMS admin project backlog](https://github.com/orgs/wagtail/projects/9?query=is%3Aopen+sort%3Aupdated-desc).

### [Wagtail Space organization #73](https://github.com/wagtail/roadmap/issues/73)

Existing item – already scheduled for Wagtail 6.1\*, retained as-is.

### [Auto-locking for pages #41](https://github.com/wagtail/roadmap/issues/41)

Existing item – already scheduled for Wagtail 6.1\*, retained as-is.

### [Auto-save support refactorings with Telepath #47](https://github.com/wagtail/roadmap/issues/47)

Existing item – scheduled for Wagtail 6.0, moving to Wagtail 6.1\* as-is.

### [RFC 72: Background workers #53](https://github.com/wagtail/roadmap/issues/53)

Existing item – moving from Future to Wagtail 6.1\* as-is. See [RFC 72: Background workers](https://github.com/wagtail/rfcs/pull/72)

### Information-dense admin interface

Refinements to the CMS admin interface to improve information density. See [Information density: spacious/snug mode #11265](https://github.com/wagtail/wagtail/issues/11265).

#### Intended outcome

- Admin interface UI tweaks for all users to improve information density.
- A possible opt-in "theme" option for users who want even greater information density.

## Proposed roadmap items for Wagtail v6.2\* (August 2024)

### [Sustainability improvements #72](https://github.com/wagtail/roadmap/issues/72)

Existing item – currently scheduled for Wagtail 6.1\*, will be moved to Wagtail 6.2\* so there is more time to earmark more specific improvements.

Based on a roadmap of improvements to Wagtail as defined in the previous release cycle. As a follow-up to [Sustainability considerations for Wagtail sites #38](https://github.com/wagtail/roadmap/issues/38) and [Google Summer of Code 2023 #59](https://github.com/wagtail/roadmap/issues/59), implementation of possible improvements to Wagtail.

### High-contrast admin themes

A follow-up to [Polish Dark Mode #65](https://github.com/wagtail/roadmap/issues/65). We will introduce a set of high-contrast themes for the Wagtail admin interface. This builds upon the work to make the admin theme-able, and aligns with Wagtail’s goals to improve accessibility.

#### Intended outcome

- The themes will be available for all users via the theme switcher in the account profile form.
- A high-contrast "dark" theme.
- Possibly a high-contrast "light" theme.

## Proposed roadmap items for "Future" releases

None.
