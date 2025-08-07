# RFC 109: Public roadmap updates

- RFC: 109
- Author: Thibaud Colas
- Created: 2025-08-06
- Last Modified: 2025-08-06

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84). See also RFCs [#86](086-roadmap-updates.md), [#88](088-roadmap-updates.md), [#91](091-roadmap-updates.md), [#98](098-roadmap-updates.md), [#101](101-roadmap-updates.md), [#103](103-roadmap-updates.md), [#106](106-roadmap-updates.md), [#107](107-roadmap-updates.md).

## Version number for the next release

We expect v7.2\* (minor release) in November 2025 based on discussions to date.

\* Provisional version number.

## Roadmap for v7.1 (August 2025)

Here is the status of roadmap items for the latest release:

| Roadmap item                                                                          | Status | Notes                                 |
| ------------------------------------------------------------------------------------- | ------ | ------------------------------------- |
| [Google Summer of Code 2025](https://github.com/wagtail/roadmap/issues/97)            | Done   | Projects ongoing until end of August  |
| [Content Security Policy compatibility](https://github.com/wagtail/roadmap/issues/92) | v7.2\* | Ongoing (CSS completed, JS under way) |
| [Site settings permissions](https://github.com/wagtail/roadmap/issues/95)             | Done   |                                       |
| [Headless userbar](https://github.com/wagtail/roadmap/issues/100)                     | Done   |                                       |
| [Headless demo site](https://github.com/wagtail/roadmap/issues/98)                    | Done   | Possible follow-ups                   |
| [Image library UX improvements](https://github.com/wagtail/roadmap/issues/101)        | Done   | Follow-ups under way                  |
| [Customizable page explorer](https://github.com/wagtail/roadmap/issues/102)           | v7.3\* | Move to 7.2, 7.3, or Future?          |
| [Block settings](https://github.com/wagtail/roadmap/issues/103)                       | Done   | Possible follow-ups with UI team      |

## Roadmap for v7.2\* (November 2025)

### [Wagtail Space 2025 🚀](https://github.com/wagtail/roadmap/issues/112)

Size: XL

[Wagtail Space 2025](https://wagtail.org/wagtail-space-2025/) is a free virtual event for people who are improving the world through code and content, coming up October 8 - 10!

### [Search backend rewrite](https://github.com/wagtail/roadmap/issues/104)

Size: L

Refactoring of Wagtail's search backends API to reduce technical debt and better support future improvements. Expected scope:

- Bug fixes to existing search backends
- Better test setup across search backends
- Better multilingual support
- Support for multiple indexes

### [Polish sites showcase](https://github.com/wagtail/roadmap/issues/105)

Size: M

Improvements to the [Wagtail.org sites showcase](https://wagtail.org/showcase/) and-or [Made with Wagtail](https://madewithwagtail.org/) to better showcase our community's work.

### [Readability checks](https://github.com/wagtail/roadmap/issues/50)

Size: S

Readability is fundamental to accessibility, and there are a lot of existing checks we could integrate directly in the CMS. Mockup, with a "reading age" metric and a "sentence is hard to read" content check:

<img width="693" height="393" alt="readability checks with reading age metric and sentence is hard to read content check" src="https://github.com/user-attachments/assets/15b2cdd5-8ffd-4737-bee3-d13f2f58eb23" />

### AI concepts

Size: L

Creation of [AI-powered CMS features via third-party packages](https://wagtail.org/blog/ai-in-the-cms-steering-the-ecosystem/), dependent on new general-purpose extension points within Wagtail core. Features considered to date:

- **Content summaries**: for example page descriptions, or even titles.
- **Image descriptions**: captions, alt text, ideally contextual (adjusted for surrounding content).
- **Content classification**: auto-filling categories or tags.
- **Related content suggestions**: encouraging linking to existing pages where suitable.
- **Translations**: making it simpler to create multilingual sites or even one-off pages.
- **Qualitative content feedback**: think readability improvements or style guide alignment.
- **Quantitative feedback**: repeatable metrics based on the page’s content.
- **Improvements suggestions**: based on automated accessibility and SEO checks.
- **PDF to HTML conversion**: a common requirement to make content more accessible.

### Orderable snippets

Size: M

Support to rearrange the order of snippets in the admin interface, similar to pages’ "Sort menu order". See [Rearrange order of Snippets in ModelViewSet index view #10816](https://github.com/wagtail/wagtail/issues/10816) & [PR #12857](https://github.com/wagtail/wagtail/pull/12857).

### Maintainer week

Size: M

Over a week at the end of October, we will go through our pull request and issues backlogs and triage. This will take the form of an online event across multiple days, and other activities to be determined tied with [Hacktoberfest](https://hacktoberfest.com/). See further info from [2024 maintainer week](https://github.com/wagtail/roadmap/issues/89).

## Roadmap for v7.3\* (February 2026)

### [Block settings](https://github.com/wagtail/roadmap/issues/103)

A new capability for StreamField blocks, to group "settings" or rarely-used fields in a separate section to reduce clutter in the editor. Mockup:

![animated block settings sub-block within blocks list](https://github.com/user-attachments/assets/556ca413-75d6-44bc-b912-40491e9342b2)

### [Customizable page explorer](https://github.com/wagtail/roadmap/issues/102)

Support for customizations such as additional columns (and filters), reducing the need for developers to create custom views for pages.

### [Autosave MVP](https://github.com/wagtail/roadmap/issues/47)

**Save draft without page reload, on demand** – an incremental steps towards [autosave functionality](https://github.com/wagtail/roadmap/issues/24). Mockup of future auto-save UI:

<img width="1577" height="1187" alt="auto-saving UI with status message and success toast" src="https://github.com/user-attachments/assets/168b410e-3376-4efc-993b-8d39f814aa40" />

## Roadmap for "Future" releases

None

## Proposed roadmap items to close

None

## Items that didn’t make the cut

Here are possible roadmap items that were discussed but not included in the roadmap this time, provided for feedback and for future reference. If one of those items is important to you, please comment! They may be available for external contributions or for a [feature sponsorship](https://wagtail.org/sponsor/).

- Better support for test data ([pytest fixtures](https://github.com/wagtail/wagtail/issues/11488), [factories](https://github.com/wagtail/wagtail/issues/10628))
- Migration packages
- Document more Wagtail internals
- Large-scale demo site
