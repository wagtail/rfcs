# RFC 103: Public roadmap updates

- RFC: 103
- Author: Thibaud Colas
- Created: 2024-10-31
- Last Modified: 2024-10-31

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84). See also RFCs [#86](086-roadmap-updates.md), [#88](088-roadmap-updates.md), [#91](091-roadmap-updates.md), [#98](098-roadmap-updates.md), [#101](101-roadmap-updates.md).

## Version number for the February 2025 release

With no specific discussion to date, we currently expect the February 2025 release will be version 6.4\*.

\* Provisional version number.

## Review of roadmap items for Wagtail 6.3 LTS (November 2024)

6 of 7 roadmap items will be marked as Done.

The following Wagtail 6.3 roadmap items will be marked as Done:

- [Enhanced contrast admin themes](https://github.com/wagtail/roadmap/issues/76)
- [Dashboard enhancements](https://github.com/wagtail/roadmap/issues/86)
- [Universal Design](https://github.com/wagtail/roadmap/issues/88)

The following Wagtail 6.3 roadmap items will be marked as Done and there will be a follow-up item in v6.3\*:

- [Admin UI performance benchmark](https://github.com/wagtail/roadmap/issues/87)

The following Wagtail 6.3 roadmap items will be marked as Done and there will be follow-up work outside of the formap roadmap in v6.3\*:

- [Maintainer week](https://github.com/wagtail/roadmap/issues/89)
- [Content checks enhancements](https://github.com/wagtail/roadmap/issues/90)

The following Wagtail 6.3 roadmap items will be moved to v6.4\*:

- [Autosave support refactorings with Telepath](https://github.com/wagtail/roadmap/issues/47)

The following Wagtail 6.3 roadmap items will be moved to v6.5\*:

- None

## Proposed roadmap items for Wagtail 6.4\* (February 2025)

### [StreamField blocks previews](https://github.com/wagtail/roadmap/issues/84)

Existing item, as-is.

### [StreamField blocks drag'n'drop](https://github.com/wagtail/roadmap/issues/85)

Existing item, as-is.

### [Admin UI performance improvements](https://github.com/wagtail/roadmap/issues/80)

Existing item, as-is.

### [Auto-save support refactorings with Telepath #47](https://github.com/wagtail/roadmap/issues/47)

Existing item, as-is – scheduled for Wagtail 6.3\*, moving to Wagtail 6.4\*.

### Headless roadmap

We will do further planning towards improving headless support directly in Wagtail. This includes:

- Review-ready draft of [RFC 100: Enhancing headless support](https://github.com/wagtail/rfcs/pull/100).
- RFC 100 review: approved or rejected.
- Creation of the 5-10 tracking issues needed to implement the RFC.
  - For example feature requests for a headless userbar, headless accessibility checks.
- Completion of [moving headless docs to the developer documentation](https://github.com/wagtail/wagtail/pull/12039)
- TBC: creation of a headless bakerydemo.

## Proposed roadmap items for Wagtail v6.5\* (May 2025)

### [Admin interface accessibility improvements #71](https://github.com/wagtail/roadmap/issues/71)

Existing item – moving from Wagtail 6.4\* to 6.5\* so there is more time to earmark more specific improvements / find a feature sponsor.

### [Sustainability improvements #72](https://github.com/wagtail/roadmap/issues/72)

Existing item – moving from Wagtail 6.4\* to 6.5\* so there is more time to earmark more specific improvements / find a feature sponsor.

## Proposed roadmap items for "Future" releases

### Content Security Policy

This will involve:

- Providing official recommendations for compatible CSP settings.
- Ensuring essential functionality works with a strict CSP.
- Documenting or backlogging all CSP-related issues.

See [CSP compatibility issues #1288](https://github.com/wagtail/wagtail/issues/1288) and the [Wagtail Stimulus Adoption Schedule (2022-25) 🎛️](https://docs.google.com/spreadsheets/d/1LdrXlj8OeCWy3B_moYZ-ynhfZZtFVHPahG9GFoT-XBs/edit).
