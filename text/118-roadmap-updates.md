# RFC 118: Public roadmap updates

- RFC: 118
- Author: Thibaud Colas
- Created: 2026-11-04
- Last Modified: 2026-11-04

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. For context, see [past roadmap-focused RFCs](https://github.com/wagtail/rfcs/pulls?q=is%3Apr+label%3Aroadmap) and the [Wagtail release schedule](https://github.com/wagtail/wagtail/wiki/Release-schedule).

## Version number for the next release

Provisional version number: v8.2 (minor release), in February 2027 based on discussions to date.

## Roadmap for the previous release

Here is the status of roadmap items for the latest release, v8.1 (November 2026):

| Roadmap item                                                                 | Status | Notes |
| ---------------------------------------------------------------------------- | ------ | ----- |
| [SEO power tools](https://github.com/wagtail/roadmap/issues/106)             | Done    | Follow-ups outside the roadmap       |
| [Agent-ready publishing](https://github.com/wagtail/roadmap/issues/244)      | Done    | Follow-ups outside the roadmap |
| [Wagtail Space 2026](https://github.com/wagtail/roadmap/issues/233)          | Done    | Follow-ups outside the roadmap |
| [Agent skills for DX](https://github.com/wagtail/roadmap/issues/241)         | Done    | Follow-ups outside the roadmap |
| [Security process automation](https://github.com/wagtail/roadmap/issues/240) | Done    | Follow-ups outside the roadmap |
| [Write API follow-ups](https://github.com/wagtail/roadmap/issues/239)        | Done    | Follow-ups outside the roadmap |
| [Maintenance automation](https://github.com/wagtail/roadmap/issues/243)      | Done    | Follow-ups outside the roadmap |

## Roadmap for the next release

Proposed roadmap items for v8.2 (February 2027).

| Item                                                                        | Size | Strategic theme                        |
| --------------------------------------------------------------------------- | ---- | -------------------------------------- |
| [JPEG XL images support](#jpeg-xl-images-support)                           | S    | 🏔️ Future-ready content management     |
| API v3 stable                                                               | L    | 🏗️ Enabling capabilities               |
| Permissions improvements                                                    | L    | 🏗️ Enabling capabilities               |
| [Documentation overhaul](#documentation-overhaul)                           | L    | 🏗️ Enabling capabilities               |
| MCP                                                                         | L    | 🛠️ DX for humans with AI               |

### [JPEG XL images support](https://github.com/wagtail/roadmap/issues/235)

Size: M, Strategic theme: 🏔️ Future-ready content management

Support for [JPEG XL](https://en.wikipedia.org/wiki/JPEG_XL) (.jxl) image uploads and renditions in Wagtail. JPEG XL delivers archival-quality images at significantly lower file sizes than JPEG and PNG, making it a compelling upload and storage format. This will become more essential with improving browser support, so we can achieve tangible reductions in web pages weight with the new format.

### API v3 stable

Size: L, Strategic theme: 🏗️ Enabling capabilities

Stabilization of the [v3 API](https://docs.wagtail.org/en/latest/advanced_topics/api/v3/index.html.md) preview, with the goal of it being a safe default for new API-facing projects. Tentatively including:

- Dogfooding of API v3 within the project
- More API v3 coverage in documentation
- TBC: API v2 feature parity goals
- TBC: Custom / complex use cases

### Granular permission model

Size: L, Strategic theme: 🏗️ Enabling capabilities

Improvements to Wagtail's permissions model beyond the current page tree-based inheritance. Tentatively including:

- [Implement a 'choose' permission for pages](https://github.com/wagtail/wagtail/issues/9714)
- TBC: [Implement a "choose" permission for non-page models (snippets etc.)](https://github.com/wagtail/wagtail/issues/14272)
- Per-instance permission checks for [custom permissions support](https://github.com/wagtail/wagtail/issues/2907)

### [Documentation overhaul](https://github.com/wagtail/roadmap/issues/236)

Size: L, Strategic theme: 🏗️ Enabling capabilities

New documentation audit and [documentation reorganisation](https://github.com/wagtail/wagtail/issues/6781) to increase the quantity and quality of our docs, in line with the [product strategy](https://wagtail.org/product-strategy/)

### MCP experimentation

Size: M, Strategic theme: 🛠️ DX for humans with AI

Iterations on the experimental [Wagtail MCP package](https://github.com/wagtail/wagtail-mcp) to bring it closer to production readiness, incorporating feedback from early adopters and addressing identified gaps in functionality and usability. Tentatively:

- Production infrastructure compatibility
- TBC: In-CMS agentic interface
- TBC: Prompt injection defense

## Roadmap for the next+1 release

Proposed roadmap items for v8.3 LTS (May 2027).

| Item                                                                        | Size | Strategic theme                        |
| --------------------------------------------------------------------------- | ---- | -------------------------------------- |
| [Enhanced dashboard UX](#enhanced-dashboard-ux)                             | M    | 🏔️ Future-ready content management     |
| [Integrated measurement and tracking](#integrated-measurement-and-tracking) | L    | 🏔️ Future-ready content management     |
| [API v3 final](#api-v3-final) | L    | 🏔️ Future-ready content management     |


### [Enhanced dashboard UX](https://github.com/wagtail/roadmap/issues/246)

Size: M, Strategic theme: 🏔️ Future-ready content management

A new iteration of [Enhanced dashboard](https://github.com/wagtail/roadmap/issues/45) based on UX research and scoping to identify 3-5 high-impact dashboard improvements.

Likely items:

- A "Create content" entry point
- Custom dashboard panels (frequent actions, bookmarks, owned pages)
- Workflow approval UI improvements

### [Integrated measurement and tracking](https://github.com/wagtail/roadmap/issues/247)

Size: L, Strategic theme: 🏔️ Future-ready content management

Spinning this off from [Enhanced dashboard](https://github.com/wagtail/roadmap/issues/45). R&D into use cases and architecture for content performance within the Wagtail admin.

Delivery as a core feature and-or official package, based on discovery findings. Likely items:

- A lightweight framework for tracking content KPIs (accessibility, sustainability / carbon emissions, SEO, analytics)
- Reporting of the KPIs in the page editor, dedicated reports, APIs, dashboard panels

### API v3 final

Size: L, Strategic theme: 🏗️ Enabling capabilities

Finalization of the API v3 and deprecation of the v2 API. Including any further improvements identified during the v3 preview period. Earmarked features:

- API tokens expiry and more advanced token management features
- CMS UX adjustments for service accounts

## Roadmap for "Future" releases

No changes

## Proposed roadmap items to close

TBC

## Items that didn't make the cut

TBC
