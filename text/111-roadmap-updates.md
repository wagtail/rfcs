# RFC 111: Public roadmap updates

- RFC: 111
- Author: Thibaud Colas
- Created: 2025-11-06
- Last Modified: 2025-11-06

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84). See also RFCs [#86](086-roadmap-updates.md), [#88](088-roadmap-updates.md), [#91](091-roadmap-updates.md), [#98](098-roadmap-updates.md), [#101](101-roadmap-updates.md), [#103](103-roadmap-updates.md), [#106](106-roadmap-updates.md), [#107](107-roadmap-updates.md), [#109](109-roadmap-updates.md).

## Version number for the next release

We expect v7.3\* (minor release) in February 2026 based on discussions to date.

\* Provisional version number.

## Roadmap for v7.2 (November 2025)

Here is the status of roadmap items for the latest release:

| Roadmap item                                                                          | Status | Notes                                                                                   |
| ------------------------------------------------------------------------------------- | ------ | --------------------------------------------------------------------------------------- |
| [Wagtail Space 2025 🚀](https://github.com/wagtail/roadmap/issues/112)                | Done   |                                                                                         |
| [Search backend rewrite](https://github.com/wagtail/roadmap/issues/104)               | Done   | Follow-ups outside the roadmap                                                          |
| [Polish sites showcase](https://github.com/wagtail/roadmap/issues/105)                | Close  | Will be worked on outside the roadmap                                                   |
| [AI concepts](https://github.com/wagtail/roadmap/issues/106)                          | Done   | Possible follow-ups, see [AI roadmap](https://github.com/wagtail/wagtail-ai/issues/124) |
| [Content Security Policy compatibility](https://github.com/wagtail/roadmap/issues/92) | Done   | Follow-ups outside the roadmap                                                          |
| [Readability checks](https://github.com/wagtail/roadmap/issues/50)                    | Done   |                                                                                         |
| [Orderable snippets](https://github.com/wagtail/roadmap/issues/108)                   | Done   |                                                                                         |
| [Maintainer week](https://github.com/wagtail/roadmap/issues/115)                       | Done   | Planned for next week                                                                   |

## Roadmap for v7.3\* (February 2026)

### [Autosave MVP](https://github.com/wagtail/roadmap/issues/47)

Size: L

**Save draft without page reload, on demand** – an incremental steps towards [autosave functionality](https://github.com/wagtail/roadmap/issues/24). Mockup of future auto-save UI:

<img width="1577" height="1187" alt="auto-saving UI with status message and success toast" src="https://github.com/user-attachments/assets/168b410e-3376-4efc-993b-8d39f814aa40" />

### [Block settings](https://github.com/wagtail/roadmap/issues/103)

Size: M

A new capability for StreamField blocks, to group "settings" or rarely-used fields in a separate section to reduce clutter in the editor. Mockup:

![animated block settings sub-block within blocks list](https://github.com/user-attachments/assets/556ca413-75d6-44bc-b912-40491e9342b2)

### Model search improvements

Size: L

Incremental improvements to the new [django-modelsearch](https://github.com/kaedroho/django-modelsearch) re-implementation of Wagtail search backends. Tentatively including:

- Vector indexing support
- Search permission handling
- Filtering on related objects

### llms.txt for Wagtail docs

Size: S

Adoption of the [llms.txt](https://llmstxt.org/) proposal for the Wagtail developer documentation and user guide, to help Wagtail users more easily find information via AI tools.

See also: [llms.txt to make the docs more accessible to LLMs #13097](https://github.com/wagtail/wagtail/issues/13097).

### AI checker concepts

Size: M

Enhancements of [Wagtail AI](https://github.com/wagtail/wagtail-ai) functionality, with a focus on better integration with content checks and metrics. Features considered to date:

- **Qualitative content feedback**: think readability improvements or style guide alignment.
- **Quantitative feedback**: repeatable metrics based on the page’s content.
- **Improvements suggestions**: based on automated accessibility and SEO checks.

Related:

- [SEO power tools](https://github.com/wagtail/roadmap/issues/106)
- [SEO search description automated content check #12252](https://github.com/wagtail/wagtail/issues/12252)
- [Custom content metrics support in page editor #12935](https://github.com/wagtail/wagtail/issues/12935)

## Roadmap for v7.4\* (May 2026)

### [Customizable page explorer](https://github.com/wagtail/roadmap/issues/102)

Size: M

(Moved from v7.3\* to v7.4\* to accommodate other priorities)

Support for customizations such as additional columns (and filters), reducing the need for developers to create custom views for pages.

### [Independent security audit](https://github.com/wagtail/roadmap/issues/111)

Size: S

An independent audit of the product and-or project for us to procure.

### [Package maintainers guide](https://github.com/wagtail/roadmap/issues/108)

Size: XS

Revamp of the existing [Python Package Maintenance Guidelines](https://github.com/wagtail/wagtail/wiki/Python-Package-Maintenance-Guidelines), to add more content, reflect the work on [Wagtail Nest](https://github.com/wagtail-nest), and modernize.

### Natural language search

Size: L

**Tentative item, highly dependent on evolution of vector indexing / search capabilities across database backends**. Improvement to Wagtail’s search interface to support natural language queries. Depending on underlying search backend capabilities, this would either be:

- Vector search, with semantic understanding of queries and content.
- Or RAG (retrieval-augmented generation) approach, combining traditional search with LLMs to interpret queries and content.
- Or (TBC) conversational search interface, with follow-up questions and clarifications.

## Roadmap for "Future" releases

### Flexible forms

Size: M

Revamp of Wagtail’s forms creation, to create more flexible forms with StreamField.

See [New StreamField-based form builder](https://github.com/wagtail/wagtail/discussions/11389) and [wagtail-flexible-forms](https://github.com/coderedcorp/wagtail-flexible-forms).

## Proposed roadmap items to close

None

## Items that didn't make the cut

Here are possible roadmap items that were discussed but not included in the roadmap this time, provided for feedback and for future reference. If one of those items is important to you, please comment! They may be available for external contributions or for a [feature sponsorship](https://wagtail.org/sponsor/).

- Improved permissions policy
- Improved document library
- Choosers UI improvements
- Increased django.tasks usage
- Customizable dashboard
