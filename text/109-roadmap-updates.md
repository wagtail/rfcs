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

| Roadmap item                                                                          | Status | Notes                                |
| ------------------------------------------------------------------------------------- | ------ | ------------------------------------ |
| [Google Summer of Code 2025](https://github.com/wagtail/roadmap/issues/97)            | Done   | Projects ongoing until end of August |
| [Content Security Policy compatibility](https://github.com/wagtail/roadmap/issues/92) | TBC    | Ongoing                              |
| [Site settings permissions](https://github.com/wagtail/roadmap/issues/95)             | Done   |                                      |
| [Headless userbar](https://github.com/wagtail/roadmap/issues/100)                     | Done   |                                      |
| [Headless demo site](https://github.com/wagtail/roadmap/issues/98)                    | Done   | Possible follow-ups                  |
| [Image library UX improvements](https://github.com/wagtail/roadmap/issues/101)        | Done   | Follow-ups under way                 |
| [Customizable page explorer](https://github.com/wagtail/roadmap/issues/102)           | TBC    | Move to 7.2, 7.3, or Future?         |
| [Block settings](https://github.com/wagtail/roadmap/issues/103)                       | Done   | Possible follow-ups with UI team     |

## Roadmap for v7.2\* (November 2025)

### [Wagtail Space 2025 🚀](https://github.com/wagtail/roadmap/issues/112)

[Wagtail Space 2025](https://wagtail.org/wagtail-space-2025/) is a free virtual event for people who are improving the world through code and content, coming up October 8 - 10!

### [Search backend rewrite](https://github.com/wagtail/roadmap/issues/104)

Refactoring of Wagtail's search backends API to reduce technical debt and better support future improvements, such as vector indexing as a search backend, or better support for hybrid search concepts.

### [Polish sites showcase](https://github.com/wagtail/roadmap/issues/105)

Improvements to the [Wagtail.org sites showcase](https://wagtail.org/showcase/) and-or [Made with Wagtail](https://madewithwagtail.org/) to better showcase our community's work.

### [Readability checks](https://github.com/wagtail/roadmap/issues/50)

Readability is fundamental to accessibility, and there are a lot of existing checks we could integrate directly in the CMS. Mockup, with a "reading age" metric and a "sentence is hard to read" content check:

<img width="693" height="393" alt="readability checks with reading age metric and sentence is hard to read content check" src="https://github.com/user-attachments/assets/15b2cdd5-8ffd-4737-bee3-d13f2f58eb23" />

### AI concepts

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

### Maintainer week

Over a week at the end of October, we will go through our pull request and issues backlogs and triage. This will take the form of an online event across multiple days, and other activities to be determined tied with [Hacktoberfest](https://hacktoberfest.com/). See further info from [2024 maintainer week](https://github.com/wagtail/roadmap/issues/89).

## Roadmap for v7.3\* (February 2026)

TBC

## Roadmap for "Future" releases

None

## Proposed roadmap items to close

None

## Items that didn’t make the cut

Here are possible roadmap items that were discussed but not included in the roadmap this time, provided for feedback and for future reference. If one of those items is important to you, please comment! They may be available for external contributions or for a [feature sponsorship](https://wagtail.org/sponsor/).

- [Enhanced dashboard](https://github.com/wagtail/roadmap/issues/45)
- [Autosave MVP](https://github.com/wagtail/roadmap/issues/47)
- Better support for test data ([pytest fixtures](https://github.com/wagtail/wagtail/issues/11488), [factories](https://github.com/wagtail/wagtail/issues/10628))
- Migration packages
- Orderable snippets
