# RFC 086: Public roadmap updates

- RFC: 086
- Author: Thibaud Colas
- Created: 2023-04-23
- Last Modified: 2023-04-23

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows recent updates and process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84).

The majority of the changes here are proposed additions, but there are also roadmap items being rescheduled from v5.1\* to future releases.

## Proposed items scheduled for Wagtail 5.1\* (August 2023)

> 5.1\*: Provisional version number

### [Enhanced dashboard #45](https://github.com/wagtail/roadmap/issues/45)

> Proposed tags: ux, customisations

Redesign of Wagtail’s dashboard to make it more useful. This will include:

- Workflow functionality improvements
- Visual design changes to summary items and welcome message

See also:

- [The Wagtail dashboard could be enhanced #8325](https://github.com/wagtail/wagtail/discussions/8325)
- [Workflow approval UI on dashboard is confusing #7132](https://github.com/wagtail/wagtail/issues/7132)

### [Permissions consolidation #39](https://github.com/wagtail/roadmap/issues/39)

> Proposed tags: tech debt, performance

Consolidating our approach to permissions so code is easier to maintain.

- Consistent permission model
- Single approach for permissions (tech debt/performance as we can then introduce better caching in one place)
- Page choose permission
- Cache permissions to reduce queries

### [Treeless page listings #33](https://github.com/wagtail/roadmap/issues/33)

> Proposed tags: ux, scale, needs sponsorship

See [RFC 82: Treeless page listings](https://github.com/wagtail/rfcs/pull/82).

### [More snippets enhancements #48](https://github.com/wagtail/roadmap/issues/48)

> Proposed tags: ux, content lifecycle

Goal: Snippets feature parity with ModelAdmin. Follow-up to [Snippets enhancements #35](https://github.com/wagtail/roadmap/issues/35). See [RFC 85: Snippets parity with ModelAdmin](https://github.com/wagtail/rfcs/pull/85), and [Snippets vs ModelAdmin parity wagtail#10206](https://github.com/wagtail/wagtail/discussions/10206).

### [Admin accessibility audit #46](https://github.com/wagtail/roadmap/issues/46)

> Proposed tags: ux, accessibility

A comprehensive accessibility audit of Wagtail’s admin interface, according to our target [accessibility standards](https://wagtail.org/accessibility/) (WCAG 2.1 AA, ATAG 2.0), potentially also including WCAG 2.1 AAA and WCAG 2.2 A, AA, AAA criteria.

#### Intended outcome

- Updated accessibility statement with known issues
- Prioritised backlog of accessibility issues to resolve

#### More information

- [Living accessibility audit](https://docs.google.com/spreadsheets/d/1l7tnpEyJiC5BWE_JX0XCkknyrjxYA5T2aee5JgPnmi4/edit)
- [WCAG 2.1 AA for CMS admin backlog](https://github.com/wagtail/wagtail/projects/5).

### Content APIs in rich text / StreamField

> Proposed tags: content lifecycle, customisations

See [Feature request: add support for more customisations of Draftail #5580](https://github.com/wagtail/wagtail/issues/5580). Potentially also including more editor-agnostic APIs, and StreamField APIs (discovery needed).

See also [Draftail for general text entry #26](https://github.com/wagtail/roadmap/issues/26)

### UI auditing & consolidation

> Proposed tags: ux, tech debt

Reduce UI debt via more UI auditing / patterns consolidation to be done. Potentially including completion of [chooser UI updates #9246](https://github.com/wagtail/wagtail/pull/9246).

### Page editor UX improvements

> Proposed tags: ux

Further refinements on recent UX improvements in recent releases.

#### Intended Outcome

- [Page editor minimap UI refinements #10055](https://github.com/wagtail/wagtail/issues/10055)
- Other more minor UX changes based on design review ([Wagtail 3.x-4.x editor experience feedback #9553](https://github.com/wagtail/wagtail/discussions/9553))

#### More information

See [Wagtail 3.x-4.x editor experience feedback #9553](https://github.com/wagtail/wagtail/discussions/9553)

### Extend Stimulus adoption

> Proposed tags: tech debt, security

Further adoption of Stimulus to support stricter CSP policies (stop using inline JS to initialise features).

Exact improvements TBC in line with [Stimulus adoption schedule](https://docs.google.com/spreadsheets/d/1LdrXlj8OeCWy3B_moYZ-ynhfZZtFVHPahG9GFoT-XBs/edit).

### [Google Summer of Code 2023 #31](https://github.com/wagtail/roadmap/issues/31)

Wagtail will take part in the Google Summer of Code programme in 2023.

#### Intended Outcome

We will work with students and beginners to open source as part of the internships.

#### More information

- See [Wagtail wiki: Google Summer of Code 2023](https://github.com/wagtail/wagtail/wiki/Google-Summer-of-Code-2023).

## Proposed items scheduled for Wagtail 5.2\* (November 2023)

### [Auto-locking for pages #41](https://github.com/wagtail/roadmap/issues/41)

> Moving from v5.1 to v5.2 as we are hoping to find a sponsor for this work.

A preliminary step to the existing [Autosave (roadmap#24)](https://github.com/wagtail/roadmap/issues/24).

### [RFC 72: Background workers](https://github.com/wagtail/rfcs/pull/72)

> Moving from v5.1 to v5.2 because of prioritisation.

### [Image optimisations #44](https://github.com/wagtail/roadmap/issues/44)

> Moving from v5.1 to v5.2 pending planning of Google Summer of Code project.

Improvements to reduce the weight of images generated with Wagtail, that will be relevant for all sites with little to no rework.

#### Intended outcome

- [RFC 71: support for responsive images](https://github.com/wagtail/rfcs/blob/main/text/071-responsive-images-support.md)
- Lossless and lossy [image optimisation operations (Willow#69)](https://github.com/wagtail/Willow/pull/69)
- AVIF support (see [Pillow#5201](https://github.com/python-pillow/Pillow/pull/5201))

### [Telepath everything! #47](https://github.com/wagtail/roadmap/issues/47)

> Moving from v5.1 to v5.2 because of prioritisation.

Extend usage of [Telepath in Wagtail](https://wagtail.org/blog/telepath/) to more of Wagtail’s forms. A preliminary refactoring to the existing [Autosave (roadmap#24)](https://github.com/wagtail/roadmap/issues/24). One of the first steps will be to clarify any potential overlaps with Stimulus usage.

### [Search improvements #42](https://github.com/wagtail/roadmap/issues/42)

> Moving from v5.1 to v5.2 due to prioritisation changes.

Follow-up to earlier consolidation of search functionality.

## Intended Outcome

- Implement missing features in db search backends: boosting / limiting fields
- Pick a successor to elasticsearch as our best-of-breed search backend
- Move search into separate package
- New search back-end
- Better docs/APIs for third-parties

## Proposed items for indeterminate "Future" releases

> In addition to all items currently present in "Future" column

### [RTL languages support #43](https://github.com/wagtail/roadmap/issues/43)

> Proposed tags: ux, needs sponsorship
> Moving from v5.1 to "Future" as we are hoping to find a sponsor for this work.

Full support for right-to-left (RTL) languages in the Wagtail admin.

#### Intended Outcome

- The admin UI works with RTL layout out of the box
- Contributor documentation shows how RTL languages should be handled
- Testing of correct RTL rendering

#### More information

- [Supporting RTL languages](https://github.com/wagtail/wagtail/discussions/7793)

### Scheduled page reviews

> Proposed tags: content lifecycle, needs sponsorship

- Audit and compliance reporting: who did what and when?

### Media versioning

> Proposed tags: content lifecycle, needs sponsorship

Bring snippets addons to image and document models for versioning and history of images and documents.
