# RFC 116: Public roadmap updates

- RFC: 116
- Author: Thibaud Colas
- Created: 2026-05-05
- Last Modified: 2026-07-20

## Abstract

This RFC provides a high-level overview of proposed [public roadmap](https://github.com/wagtail/roadmap) updates for future releases. For context, see [past roadmap-focused RFCs](https://github.com/wagtail/rfcs/pulls?q=is%3Apr+label%3Aroadmap) and the [Wagtail release schedule](https://github.com/wagtail/wagtail/wiki/Release-schedule).

For each item, we provide an indicative "size" to represent the effort involved, and a strategic theme which maps to areas of [our product strategy](https://wagtail.org/product-strategy/):

- **🏔️ Future-ready content management**: High-quality structured content, multiplayer content workflows, content discoverability and accessibility
- **🪢 Deep integrations and customizations**: Structured data model, layered integration architecture, healthy package ecosystem.
- **🛠️ DX for humans with AI**: Strong opinionated defaults, automation, comprehensive documentation.
- **🌳 How we build Wagtail**: Strong design principles, sustainable contributor experience, community stewardship.

And supporting work:

- **🏗️ Enabling capabilities**: Key supporting and enabling capabilities to deliver our vision.
- **🪴 Maintenance**: Table stakes / fundamental hygiene that supports other work.
- **Other**: Everything else.

## Version number for the next release

Provisional version number: v8.1 (minor release), in November 2026 based on discussions to date.

## Roadmap for the previous release

Here is the status of roadmap items for the latest release, v8.0 (August 2026):

| Roadmap item                                                                    | Status | Notes                                         |
| ------------------------------------------------------------------------------- | ------ | --------------------------------------------- |
| [Write API](https://github.com/wagtail/roadmap/issues/230)                      | Done   | API v3 foundation in v8.0, follow-ups in v8.1 |
| [Cyber Resilience Act readiness](https://github.com/wagtail/roadmap/issues/231) | Done   | Follow-up security automation in v8.1         |
| [Starter kit relaunch](https://github.com/wagtail/roadmap/issues/124)           | Done   | Follow-ups outside the roadmap                |
| [Customizable base page model](https://github.com/wagtail/roadmap/issues/126)   | Done   | Follow-ups outside the roadmap                |
| [Demo website redesign](https://github.com/wagtail/roadmap/issues/232)          | Done   | Follow-ups outside the roadmap                |

## Roadmap for the next release

Proposed roadmap items for v8.1 (November 2026).

| Item                                                        | Size | Strategic theme                    |
| ----------------------------------------------------------- | ---- | ---------------------------------- |
| [SEO power tools](#seo-power-tools)                         | M    | 🏔️ Future-ready content management |
| [Agent-ready publishing](#agent-ready-publishing)           | M    | 🏔️ Future-ready content management |
| [Wagtail Space 2026](#wagtail-space-2026)                   | XL   | Other                              |
| [Agent skills for DX](#agent-skills-for-dx)                 | S    | 🛠️ DX for humans with AI           |
| [Security process automation](#security-process-automation) | M    | 🌳 How we build Wagtail            |
| [Write API follow-ups](#write-api-follow-ups)               | L    | 🏗️ Enabling capabilities           |
| [Maintenance automation](#maintenance-automation)           | M    | 🪴 Maintenance                     |

### [SEO power tools](https://github.com/wagtail/roadmap/issues/106)

Size: M, Strategic theme: 🏔️ Future-ready content management

Item TBC based on feedback from potential feature sponsor. See [Looking for sponsorship: SEO power tools](https://wagtail.org/blog/looking-for-sponsorship-seo-power-tools/). New built-in SEO and content quality assurance features, with opportunities for integration with SEO and analytics tools, as well as generative AI. Likely:

- Social / SEO meta previews
- Content checks UX enhancement

### Agent-ready publishing

Size: M, Strategic theme: 🏔️ Future-ready content management

From resources like [Website Spec - Agent Readiness](https://specification.website/spec/agent-readiness/) - audit Wagtail's current capabilities, to populate a backlog of improvements in core and via a new package. And deliver top 3-5 quick wins in core if possible. Likely includes:

- [Markdown content responses](https://specification.website/spec/agent-readiness/markdown-source-endpoints/) as a core feature
- [Heading anchor links](https://github.com/wagtail/wagtail/issues/5515) / anchor links as a core feature
- [llms.txt](https://specification.website/spec/agent-readiness/llms-txt/) package
- [Agent skills discovery](https://specification.website/spec/agent-readiness/agent-skills-discovery/) package

### [Wagtail Space 2026](https://github.com/wagtail/roadmap/issues/233)

Size: XL, Strategic theme: Other

[Wagtail Space 2026](https://wagtail.org/wagtail-space-2026/) is a free virtual event for people who are improving the world through code and content, coming up November 18-20! The event is reflected on the roadmap as it's a big endeavour for our community.

### Agent skills for DX

Size: S, Strategic theme: 🛠️ DX for humans with AI

Publication and maintenance pipeline / tooling for [agent skills](https://agentskills.io/) for the developers audience. Including writing and refinements of a few skills.

- 3-5 skills focused on increasing quality / adoption of best practices and intermediate Wagtail capabilities
    - Including relevant developer documentation updates
    - Skill: general best practices
    - Skill: content modelling
    - Skill: [Wagtail version upgrades](https://wagtail.org/blog/an-agent-skill-to-upgrade-your-wagtail-site/)
- Publication in [agent skills well-known URI](https://github.com/cloudflare/agent-skills-discovery-rfc), or [AI Catalog](https://agent-card.github.io/ai-catalog/), or [library skills](https://library-skills.io/)
- Skills maintenance tooling: eval suite, agent skills publication metadata validation

### Security process automation

Size: M, Strategic theme: 🌳 How we build Wagtail

Follow-up to [Cyber Resilience Act readiness](https://github.com/wagtail/roadmap/issues/231), with a focus on automation and proactivity when it comes to security vulnerabilities. Earmarked items:

- Deep security review
- Agentic security auditing (trial [scrutineer](https://github.com/alpha-omega-security/scrutineer) or similar?)
- Security-focused AI triage (labelling, issue reproduction)
    - See [Do excellent vulnerability reports](https://daniel.haxx.se/blog/2026/06/29/do-excellent-vulnerability-reports/)

### Write API follow-ups

Size: L, Strategic theme: 🏗️ Enabling capabilities

Follow-up on what will ship in Wagtail 8.0. Exact items TBC based on [write API implementation progress](https://github.com/wagtail/wagtail/issues/14295) against the plans outlined in [RFC 115](https://wagtail.org/rfc-115/). Likely items:

- Greater write API feature coverage
- v2 API deprecation plan
- Official write API CLI client
- TBC: modern CLI beyond the API client

### Maintenance automation

Size: M, Strategic theme: 🪴 Maintenance

Investments in fundamental maintenance and QA capabilities to sustain the project. Expected work:

- Maintenance backlog curation, with weighted list of issues
- Experiment with agentic code review for core
- [bakerydemo dev app setup](https://github.com/wagtail/bakerydemo/issues/733)
- Contributing docs overhaul
    - [More documentation guidelines for contributors](https://github.com/wagtail/wagtail/issues/13801)
    - [Switch from make / Makefile to just / justfile](https://github.com/wagtail/wagtail/issues/14396)
- Automated performance reviews (higher confidence on risk of performance regressions in releases)

## Roadmap for the next+1 release

Proposed roadmap items for v8.2 (February 2027):

| Item                                                                        | Size | Strategic theme                    |
| --------------------------------------------------------------------------- | ---- | ---------------------------------- |
| [Enhanced dashboard UX](#enhanced-dashboard-ux)                             | M    | 🏔️ Future-ready content management |
| [Integrated measurement and tracking](#integrated-measurement-and-tracking) | L    | 🏔️ Future-ready content management |

### Enhanced dashboard UX

Size: M, Strategic theme: 🏔️ Future-ready content management

A new iteration of [Enhanced dashboard](https://github.com/wagtail/roadmap/issues/45). UX research and scoping to identify 3-5 high-impact dashboard improvements, derived from [Wagtail dashboard enhancements #8325](https://github.com/wagtail/wagtail/discussions/8325). Likely items:

- A "Create content" entry point
- Custom dashboard panels (frequent actions, bookmarks, owned pages)
- Workflow approval UI improvements.

### Integrated measurement and tracking

Size: L, Strategic theme: 🏔️ Future-ready content management

Spinning this off from [Enhanced dashboard](https://github.com/wagtail/roadmap/issues/45). R&D into use cases and architecture for content performance within the Wagtail admin. Likely items:

- A lightweight framework for tracking content KPIs (accessibility, sustainability / carbon emissions, SEO, analytics)
- Surfacing them in the page editor, reports, APIs, dashboard panels.

Delivery as a core feature and-or official package, based on discovery findings.

## Roadmap for "Future" releases

| Item                                                  | Size | Strategic theme                         |
| ----------------------------------------------------- | ---- | --------------------------------------- |
| [GEO power tools](#geo-power-tools)                   | M    | 🏔️ Future-ready content management      |
| [Documentation overhaul](#documentation-overhaul)     | L    | 🏗️ Enabling capabilities                |
| [Generic content importer](#generic-content-importer) | M    | 🏔️ Future-ready content management      |
| [Image rendition backends](#image-rendition-backends) | M    | 🪢 Deep integrations and customizations |
| [Shareable previews](#shareable-previews)             | S    | 🏔️ Future-ready content management      |

### GEO power tools

Size: M, Strategic theme: 🏔️ Future-ready content management

Built-in or package-based content checks and other supporting tools for Generative Engine Optimization, as a follow-up to [SEO power tools](https://github.com/wagtail/roadmap/issues/106). Focusing on aspects that are most proven and best align with Wagtail’s SEO and accessibility capabilities. This requires discovery for us to earmark specific improvements, but here are ideas that align well already:

- QA tools for power users.
    - Heading quality. Check(s) based on heading density, keyword relevance, and suitability of wording for extraction as an answer snippet.
    - llms.txt / Markdown page format preview
- Automation for everyone.
    - Structured data presence. TBC - likely as documentation recommending specific techniques or packages, so structured data creation is fully automated.
    - Q&A structured data example in bakerydemo
    - [Wagtail AI](https://wagtail.org/wagtail-ai/) GEO integration (LLM-based content checks and suggestions).

### Documentation overhaul

Size: L, Strategic theme: 🏗️ Enabling capabilities

New documentation audit and [documentation reorganisation](https://github.com/wagtail/wagtail/issues/6781) to increase the quantity and quality of our docs, in line with the [product strategy](https://wagtail.org/product-strategy/)

### Generic content importer

Size: M, Strategic theme: 🏔️ Future-ready content management

A generic re-implementation of [Wagtail Content Import](https://github.com/torchbox/wagtail-content-import), based on our Markdown / HTML importer work in [RFC 115: Write API](https://wagtail.org/rfc-115/). Optioanlly enhanced with LLMs to generate StreamFIeld block structures.

### Image rendition backends

Size: M, Strategic theme: 🪢 Deep integrations and customizations

Refactoring of image renditions to a system of pluggable backends, to help us achieve crucial improvements that are architecturally hard to implement:

- [Upfront generation of image renditions #929](https://github.com/wagtail/wagtail/issues/929)
- [Deferring image renditions](https://github.com/wagtail/wagtail/issues/3868)
- [Support disabling/limiting use of image renditions](https://github.com/wagtail/wagtail/issues/3210)

### Shareable previews

Size: S, Strategic theme: 🏔️ Future-ready content management

Implement a new system to share drafts or previews with users who do not have access to the CMS, as demonstrated by [wagtail-sharing](https://github.com/cfpb/wagtail-sharing), and to some degree by [wagtail-headless-preview](https://github.com/torchbox/wagtail-headless-preview).

## Proposed roadmap items to close

The adoption of [our new product strategy](https://wagtail.org/product-strategy/) is a good occasion for us to revisit existing roadmap items, and archive/close them if they’re no longer in alignment with the strategy, or just too out of date / no longer relevant.

### [Fully accessible admin](https://github.com/wagtail/roadmap/issues/27)

There have been a lot of improvements on accessibility over the years, this item being on the roadmap gives the wrong impression. As part of closing this, we will better document and backlog the results of our [Accessibility Conformance Reports](https://wagtail.org/accessibility/acr/) and [ATAG auditing](https://wagtail.org/accessibility/atag/).

### [Concurrent editing](https://github.com/wagtail/roadmap/issues/24)

The return-on-investment is too low for forms of concurrent editing more advanced than autosave. Although the steps we outlined on the proposed [collaborative content management roadmap](https://wagtail.org/blog/our-roadmap-for-collaborative-content-management/) are compelling, they would be a distraction compared to functionality that is more relevant for our audiences.

### [Image optimization performance](https://github.com/wagtail/roadmap/issues/110)

This item will be replaced by the new "Image rendition backends", which is more specific, and will provide the needed flexibility to cater to different needs.

### [Sustainability improvements](https://github.com/wagtail/roadmap/issues/72)

This item isn’t specific enough. There have been a lot of improvements on sustainability over the years, and we’re now at a point where we want to be more strategic about which specific ones we spend time on (like [JPEG XL images support](https://github.com/wagtail/roadmap/issues/235)). As part of closing this, we will document all our recent work on sustainability across:

- wagtail.org landing page content
- docs.wagtail.org recommendations for developers
- guide.wagtail.org considerations for CMS users

### [Draftail for general text entry](https://github.com/wagtail/roadmap/issues/26)

This item represents an outdated approach to rich text support in Wagtail. There is work to do in this area but it will not be done with Draftail.

### [Single-line rich text fields](https://github.com/wagtail/roadmap/issues/28)

This specific feature doesn’t serve its place on the roadmap. A lot of progress has been made and the remaining work ([#8249](https://github.com/wagtail/wagtail/issues/8249)) should be 2-3h with agentic development.

### [Multi-tenancy improvements](https://github.com/wagtail/roadmap/issues/74)

Without a feature sponsor emerging, improvements would be better served by more specific issues. As part of closing this item, we will clearly document the current state of multi-tenancy in Wagtail and signpost possible improvements. Documentation via:

- wagtail.org overview
- docs.wagtail.org recipes
- guide.wagtail.org multi-site docs

## Items that didn't make the cut

Here are roadmap items that were discussed but not included in the roadmap this time, provided for feedback and for future reference. If one of those items is important to you, please comment! They may be available for external contributions or for a [feature sponsorship](https://wagtail.org/sponsor/).

- Package ecosystem maintenance
- [Worfklow enhancements](https://github.com/wagtail/roadmap/issues/49)
- Admin UI components library
