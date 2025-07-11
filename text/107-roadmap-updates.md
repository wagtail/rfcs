# RFC 107: Public roadmap updates

- RFC: 107
- Author: Thibaud Colas
- Created: 2025-05-07
- Last Modified: 2025-05-07

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84). See also RFCs [#86](086-roadmap-updates.md), [#88](088-roadmap-updates.md), [#91](091-roadmap-updates.md), [#98](098-roadmap-updates.md), [#101](101-roadmap-updates.md), [#103](103-roadmap-updates.md), [#106](106-roadmap-updates.md).

## Version number for the August 2025 release

With no specific discussion to date, we currently expect the February 2025 release will be version 7.1\*.

\* Provisional version number.

## Review of roadmap items for Wagtail 7.0 (May 2025)

3 of 4 roadmap items will be marked as Done.

The following Wagtail 7.0 roadmap items will be marked as Done, with possible follow-up work:

- [Validation on publish](https://github.com/wagtail/roadmap/issues/93)
  - Follow-up on the roadmap: [Autosave](https://github.com/wagtail/roadmap/issues/24), exact next steps TBC.
- [Headless API improvements](https://github.com/wagtail/roadmap/issues/94)
  - Follow-up outside the roadmap: more improvements based on feedback in the [2024 headless survey](https://wagtail.org/blog/2024-headless-survey/).
- [CSP compatibility audit](https://github.com/wagtail/roadmap/issues/96)
  - Follow-up on the roadmap: Headless userbar, headless API improvements

The following Wagtail 7.0 roadmap items will be moved to v7.1\*:

- [Site settings permissions](https://github.com/wagtail/roadmap/issues/95)
  - Moved as-is from 7.0 to 7.1\*.

The following Wagtail 7.0 roadmap items will be moved to Future:

- None

## Proposed roadmap items for Wagtail 7.1\* (August 2025)

### [Google Summer of Code 2025](https://github.com/wagtail/roadmap/issues/97)

Showcasing [our four projects](https://wagtail.org/blog/four-contributors-for-gsoc-2025/) on the public roadmap:

- Grid-aware websites
- Content Security Policy (CSP) Compatibility
- Media listings UX
- Improving keyboard shortcuts

### [Content Security Policy compatibility](https://github.com/wagtail/roadmap/issues/92)

See above. This was already a roadmap item so will be showcased as such.

### [Site settings permissions](https://github.com/wagtail/roadmap/issues/95)

A new permission model for sites, where groups can be assigned permissions at the level of individual sites. This better maps organisational structures, compared to the current model of permissions being assigned at the level of the settings model rather than instances.

For more information, see [RFC 105: Site settings permissions](https://github.com/wagtail/rfcs/pull/105).

### [Headless userbar](https://github.com/wagtail/roadmap/issues/100)

Adding built-in support for the Wagtail userbar in headless mode, including accessibility checks, based on built-in preview support.

For more information, see [RFC 100: Enhancing headless support](https://github.com/wagtail/rfcs/pull/100).

### [Headless demo site](https://github.com/wagtail/roadmap/issues/98)

An official headless demo site – primarily so our contributors and maintainers can more easily work on headless improvements.

### Image library UX improvements

Improvements to the design and user experience of image management in Wagtail, complementing the existing "Media listings UX" GSoC Project. Currently-earmarked changes for GSoC 2025:

1. [Listing view for image admin index page #909](https://github.com/wagtail/wagtail/issues/909)
2. [Image listings & choosers needs to display more information about images #654](https://github.com/wagtail/wagtail/issues/654)
3. [Listings filters and sorting for usage count #12550](https://github.com/wagtail/wagtail/issues/12550)

Stretch goals:

- Grid view for documents
- Grid / listing mode for choosers of images and document
- [Listings filters and sorting for usage count #12550](https://github.com/wagtail/wagtail/issues/12550)

Currently-earmarked for implementation outside of GSoC 2025:

- [UX usability in images and collections #6285](https://github.com/wagtail/wagtail/issues/6285)
- [Improve UX/Performance of image overview and chooser #11330](https://github.com/wagtail/wagtail/issues/11330)
- [Set default collection when uploading images from empty collection #6899](https://github.com/wagtail/wagtail/issues/6899)
- [Image chooser modal loading is slow and does not convey progress #2364](https://github.com/wagtail/wagtail/issues/2364)

### Customizable page explorer

Support for customizations such as additional columns (and filters), reducing the need for developers to create custom views for pages.
See [Support custom columns / filters on main page explorer #11931](https://github.com/wagtail/wagtail/issues/11931).

### Block settings

A new capability for StreamField blocks, to group "settings" or rarely-used fields in a separate section to reduce clutter in the editor.

See:

- [Ability to display some model fields as settings for a group of fields, in editor UI. #2299](https://github.com/wagtail/wagtail/issues/2299)
- [Collapsible fields and sections beyond top-level panels #10227](https://github.com/wagtail/wagtail/issues/10227)

## Proposed roadmap items for Wagtail 7.2\* (November 2025)

### Search backend rewrite

Refactoring of Wagtail’s search backends API to better support future improvements, such as vector indexing as a search backend.

### Polish Sites showcase

Improvements to the [Wagtail.org sites showcase](https://wagtail.org/showcase/) and-or [Made with Wagtail](https://madewithwagtail.org/) to better showcase our community’s work.

## Proposed roadmap items for "Future" releases

### SEO power tools

See [Looking for sponsorship: SEO power tools](https://wagtail.org/blog/looking-for-sponsorship-seo-power-tools/). New built-in SEO and content quality assurance features, with opportunities for integration with SEO and analytics tools, as well as generative AI. Examples:

- Manual quality assurance tools (metadata preview in the page editor and site-wide reporting)
- Automated quality checks ([SEO search description automated content check #12252](https://github.com/wagtail/wagtail/issues/12252))
- Generative content (for example link / related pages suggestions)
- Reporting ([Custom content metrics support in page editor #12935](https://github.com/wagtail/wagtail/issues/12935))

### New table block

Creation of a new table editing UI in the CMS, addressing the shortcomings of the existing [TableBlock based on Handsontable 6.2.2](https://docs.wagtail.org/en/stable/reference/contrib/table_block.html), and [TypedTableBlock](https://docs.wagtail.org/en/stable/reference/contrib/typed_table_block.html) based on StreamField.

### Package maintainers guide

Revamp of the existing [Python Package Maintenance Guidelines](https://github.com/wagtail/wagtail/wiki/Python-Package-Maintenance-Guidelines), to add more content, reflect the work on [Wagtail Nest](https://github.com/wagtail-nest), and modernize.

### Choosers UI improvements

Iterative improvements towards a [new design for chooser modals](https://github.com/wagtail/wagtail/pull/9246), which brings them closer to the "Universal listings" UX. This includes:

- Sorting and filtering capabilities
- Improved accessibility
- A clearer path towards common link chooser requests: unification with document chooser, custom link types, custom fields

### Image optimization performance

Further investment into image optimizations to provide the best results in more cases:

- [Support disabling/limiting use of image renditions #3210](https://github.com/wagtail/wagtail/issues/3210)
- [Delaying image renditions](https://github.com/wagtail/wagtail/issues/3868)
- Other improvements to reduce file size or make Wagtail’s renditions suitable in more use cases.

### Independent security audit

An independent audit of the product and-or project for us to procure, potentially including:

- Code audit
- Penetration testing
- Dependency audit
- Infrastructure audit

See for example the [Sovereign Tech Resilience](https://www.sovereign.tech/programs/bug-resilience/) program and the [Open Tech Fund Security Lab](https://www.opentech.fund/labs/security-lab/).

### [Autosave](https://github.com/wagtail/roadmap/issues/24) MVP

Incremental steps towards [autosave functionality](https://github.com/wagtail/roadmap/issues/24), building upon [Validation on publish](https://github.com/wagtail/roadmap/issues/93).

## Proposed roadmap items to close

None

## Items that didn’t make the cut

Here are possible roadmap items that were not included in the roadmap this time, provided for feedback and for future reference:

- StreamField form builder
- Typed Wagtail proof-of-concept
- More UI components for advanced AI integrations packages
- Blocks drag-and-drop polish
- Blocks copy-paste
- Unified link picker

## Open questions
