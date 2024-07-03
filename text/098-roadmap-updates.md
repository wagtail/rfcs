# RFC 098: Public roadmap updates

- RFC: 098
- Author: Thibaud Colas
- Created: 2024-06-14
- Last Modified: 2024-06-14

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84). See also RFCs [#86](086-roadmap-updates.md), [#88](088-roadmap-updates.md), [#91](091-roadmap-updates.md).

## Version number for the August 2024 release

With no specific discussion to date, we currently expect the August 2024 release will be version 6.2\*.

\* Provisional version number.

## Review of roadmap items for Wagtail 6.1 (May 2024)

The following Wagtail 6.1 roadmap items will be marked as Done:

- [Information-dense admin interface](https://github.com/wagtail/roadmap/issues/77)
- [Wagtail Space organization](https://github.com/wagtail/roadmap/issues/73)
- [RFC 72: Background workers](https://github.com/wagtail/roadmap/issues/53)
- [Universal listings continued](https://github.com/wagtail/roadmap/issues/75)

The following Wagtail 6.1 roadmap items will be marked as Done and there will be a follow-up item in v6.2\*:

- N/A. There will be [small improvements to universal listings](https://github.com/wagtail/wagtail/discussions/10446#discussioncomment-9774389), and to information density, if time allows.

The following Wagtail 6.1 roadmap items will be moved to v6.2\*:

- [Auto-locking for pages](https://github.com/wagtail/roadmap/issues/41)

The following Wagtail 6.1 roadmap items will be moved to v6.3\*:

- [Admin interface accessibility improvements](https://github.com/wagtail/roadmap/issues/71)
- [Auto-save support refactorings with Telepath](https://github.com/wagtail/roadmap/issues/47)

## Proposed roadmap items for Wagtail 6.2\* (August 2024)

### Concurrent editing notifications

Previously known as [Auto-locking for pages](https://github.com/wagtail/roadmap/issues/41), now [RFC 95: Concurrent editing notifications](https://github.com/wagtail/rfcs/pull/95). The RFC is more compatible with the longer-term vision for [Auto-save](https://github.com/wagtail/roadmap/issues/24).

### Google Summer of Code 2024

Wagtail takes part in the Google Summer of Code programme in 2024, with two projects:

- [Alt text capabilities](https://github.com/wagtail/gsoc/blob/main/project-ideas.md#alt-text-capabilities).
- [Low-carbon accessible project templates](https://github.com/wagtail/gsoc/blob/main/project-ideas.md#alt-text-capabilities).

#### Intended Outcome

We work with beginners to open source as part of the internships.

#### More information

See [Two contributors join Wagtail for Google Summer of Code 2024](https://wagtail.org/blog/two-contributors-join-wagtail-for-google-summer-of-code-2024/).

### Alt text validation

A new feature to validate alt text on images, checking for the presence of problem patterns.

#### Intended Outcome

Improve the quality of alt text for existing content – while the [Alt text capabilities](https://github.com/wagtail/gsoc/blob/main/project-ideas.md#alt-text-capabilities) project is more about new content.

#### More information

See [Alt text validation rule in the accessibility checker #11967](https://github.com/wagtail/wagtail/issues/11967).

### Content metrics

New “Word count” and “Reading time” metrics. Those new built-in metrics will be useful for a lot of editors, and pave the way towards more advanced and bespoke metrics in the future (reading age, SEO keyword density, SEO quality traffic lights, etc).

#### Intended outcome

The metrics will be:

- Placed within the “Checks” panel
- Calculated based on the contents of the page as rendered in the live preview – currently for pages with the Wagtail userbar only.
- Calculated the same way for all languages.

#### More information

See:

- [Content quality checkers](https://github.com/wagtail/wagtail/discussions/11063)
- [Readability checks](https://github.com/wagtail/roadmap/issues/50)

## Proposed roadmap items for Wagtail v6.3\* (November 2024)

### Admin UI performance improvements

We want to make the admin user interface more responsive, by applying various optimizations.
The specific improvements are TBC, depending on the results of performance auditing of the admin interface.

### [Auto-save support refactorings with Telepath #47](https://github.com/wagtail/roadmap/issues/47)

Existing item – scheduled for Wagtail 6.1, moving to Wagtail 6.3\* as-is.

### [Admin interface accessibility improvements #71](https://github.com/wagtail/roadmap/issues/71)

Existing item – scheduled for Wagtail 6.1, moving to Wagtail 6.3\* as-is.

### [Sustainability improvements #72](https://github.com/wagtail/roadmap/issues/72)

Existing item – currently scheduled for Wagtail 6.2\*, will be moved to Wagtail 6.3\* so there is more time to earmark more specific improvements.

### [High-contrast admin themes](https://github.com/wagtail/roadmap/issues/76)

Existing item – scheduled for Wagtail 6.2\*, moving to Wagtail 6.3\* as-is.

## Proposed roadmap items for "Future" releases

### [RFC 92: Permissions Registry](https://github.com/wagtail/rfcs/pull/92)

From the RFC description:

> A new permissions registry that maps a model class to a permission configuration. Everywhere a permission check is performed in Wagtail, the registry will be consulted to determine the configuration to use for the model or model instance in question. The registry will make it easier for developers to access the permission configuration of a model from anywhere in the code. It will also allow developers to define custom permission logic for both custom models and Wagtail's built-in models, to be used in place of the default.

---

We have also discussed a separate "Async image processing" idea for the "Future" roadmap, but it doesn’t have enough details to be added at this time.
