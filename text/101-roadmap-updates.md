# RFC 101: Public roadmap updates

- RFC: 101
- Author: Thibaud Colas
- Created: 2024-07-31
- Last Modified: 2024-08-01

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. This follows process changes introduced in [RFC 84](https://github.com/wagtail/rfcs/pull/84). See also RFCs [#86](086-roadmap-updates.md), [#88](088-roadmap-updates.md), [#91](091-roadmap-updates.md), [#98](098-roadmap-updates.md).

## Version number for the November 2024 release

With no specific discussion to date, we currently expect the November 2024 release will be version 6.3\*.

\* Provisional version number.

## Review of roadmap items for Wagtail 6.2 (August 2024)

The following Wagtail 6.2 roadmap items will be marked as Done:

All items 🎉

- [Concurrent editing notifications](https://github.com/wagtail/roadmap/issues/41)
- [Google Summer of Code 2024](https://github.com/wagtail/roadmap/issues/78)
- [Content metrics](https://github.com/wagtail/roadmap/issues/82)
- [Alt text validation](https://github.com/wagtail/roadmap/issues/83)

The following Wagtail 6.2 roadmap items will be marked as Done and there will be a follow-up item in v6.3\*:

- None

The following Wagtail 6.2 roadmap items will be moved to v6.3\*:

- None

The following Wagtail 6.2 roadmap items will be moved to v6.4\*:

- None

## Proposed roadmap items for Wagtail 6.3\* (November 2024)

### [Auto-save support refactorings with Telepath #47](https://github.com/wagtail/roadmap/issues/47)

Existing item, as-is.

### [Enhanced contrast admin themes](https://github.com/wagtail/roadmap/issues/76)

Existing item (previously "High-contrast admin themes") – scheduled for Wagtail 6.2\*, moving to Wagtail 6.3\*, with the following changes:

- Renamed to "Enhanced contrast"
- Going beyond colors – see ["Prefers contrast" admin theming #12176](https://github.com/wagtail/wagtail/issues/12176)

### Dashboard enhancements

See [Wagtail dashboard enhancements #8325](https://github.com/wagtail/wagtail/discussions/8325#discussioncomment-10114586).

### Admin UI performance benchmark

Necessary preparation for [Admin UI performance improvements](https://github.com/wagtail/roadmap/issues/80).

### Universal Design

Rollout of further design changes relating to [Universal listings (search and filtering) #10446](https://github.com/wagtail/wagtail/discussions/10446#discussioncomment-9774389), with a particular focus on reducing technical debt across existing views where there could be more code reuse.

### Content checks enhancements

Follow-up to Wagtail 6.2 design enhancements:

- [Implement checker error highlights within the preview panel #12187](https://github.com/wagtail/wagtail/issues/12187)
- [Update accessibility checker documentation examples to enable more relevant rules #12188](https://github.com/wagtail/wagtail/issues/12188)

### Triage week

Over a week at the end of October, we will go through our pull request and issues backlogs and triage. This will take the form of an online event across multiple days, and other activities to be determined tied with [Hacktoberfest](https://hacktoberfest.com/).

For pull requests, this means reviewing every open PR and:

- Assess its current state, post a comment on that state
- Merge the ones that can be merged.
- Close the ones that should be transferred into a new issue (or an update posted on its existing related issue).

PRs that are not closed or merged will be targeted for resolution in future release cycles.

For issues, we will similarly triage all:

- [Unconfirmed bug reports](https://github.com/wagtail/wagtail/issues?page=1&q=sort%3Aupdated-desc+is%3Aissue+is%3Aopen+label%3Astatus%3AUnconfirmed) (current: 86)
- [“Needs design decision” issues](https://github.com/wagtail/wagtail/issues?q=is%3Aopen+is%3Aissue+label%3A%22status%3ANeeds+Design+Decision%22) (current: 42)
- Possibly other issues based on criteria to be determined.

## Proposed roadmap items for Wagtail v6.4\* (February 2025)

### StreamField blocks drag'n'drop

See [Add drag-and-drop support to StreamField #1033](https://github.com/wagtail/wagtail/issues/1033).

### StreamField blocks previews

See [StreamField block type descriptions and image previews in block chooser #11997](https://github.com/wagtail/wagtail/issues/11997).

### [Admin UI performance improvements](https://github.com/wagtail/roadmap/issues/80)

Existing item – scheduled for Wagtail 6.3\*, moving to Wagtail 6.4\* so there is more time to earmark more specific improvements.

### [Admin interface accessibility improvements #71](https://github.com/wagtail/roadmap/issues/71)

Existing item – scheduled for Wagtail 6.3\*, moving to Wagtail 6.4\* so there is more time to earmark more specific improvements.

### [Sustainability improvements #72](https://github.com/wagtail/roadmap/issues/72)

Existing item – currently scheduled for Wagtail 6.3\*, will be moved to Wagtail 6.4\* so there is more time to earmark more specific improvements.

## Proposed roadmap items for "Future" releases

No changes.
