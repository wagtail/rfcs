# RFC 100: Headless support in Wagtail core

- RFC: 100
- Author: Thibaud Colas
- Created: 2024-07-31
- Last Modified: 2024-07-31

## Abstract

Wagtail’s support for headless websites should move closer to feature-parity with Django monoliths, without reliance on third-party packages for compatibility with core functionality.
For features that aren’t supported because of technical constraints, design decisions, or inertia, there should be documented workarounds or alternatives.

## Why

- Because a lot of existing and future sites would benefit from more cohesive headless support.
- Because it’s harder to maintain this kind of support across external packages.

## Features to support

Here are two rules to determine which current and future Wagtail features must be supported for headless websites:

1. If a feature is "on by default" in Wagtail core, we must aim to support it with no external package or other custom Django implementation.
2. If a feature is on by default and unsupported, it must be possible to turn it off and we must document a workaround or alternative.

## Feature-by-feature review

Based on feature definitions from [Are we headless yet?](https://areweheadlessyet.wagtail.org/).

| Feature                                                                                   | Goal in Wagtail core | Current status     |
| ----------------------------------------------------------------------------------------- | -------------------- | ------------------ |
| [REST API](https://areweheadlessyet.wagtail.org/rest-api)                                 | Full support         | Full support       |
| [GraphQL](https://areweheadlessyet.wagtail.org/graphql)                                   | No support           | External package   |
| [Page Preview](https://areweheadlessyet.wagtail.org/page-preview)                         | Full support         | External package   |
| [Images](https://areweheadlessyet.wagtail.org/images)                                     | Full support         | Partial support    |
| [Page URL Routing](https://areweheadlessyet.wagtail.org/page-url-routing)                 | Full support         | Full support       |
| [Rich Text](https://areweheadlessyet.wagtail.org/rich-text)                               | Full support         | Known shortcomings |
| [Multi-site support](https://areweheadlessyet.wagtail.org/multi-site-support)             | Full support         | Known shortcomings |
| [Form submissions](https://areweheadlessyet.wagtail.org/form-submissions)                 | No support           | No support         |
| [Password-protected Pages](https://areweheadlessyet.wagtail.org/password-protected-pages) | Full support         | No support         |
| [Internationalisation](https://areweheadlessyet.wagtail.org/internationalisation)         | Full support         | TBC                |
| [StreamField](https://areweheadlessyet.wagtail.org/streamfield)                           | Full support         | TBC                |
| Userbar                                                                                   | Full support         | No support         |
| Content checks                                                                            | Full support         | No support         |

## Gaps to address

### Page Preview

Support depends on the [wagtail-headless-preview](https://github.com/torchbox/wagtail-headless-preview) package.
This should instead be part of Wagtail core, either as-is implementation or equivalent.

### Images

- Add first-class support for responsive images and multi-format image generation in API responses.

### Rich Text

- Add a built-in way to render rich text fields in the API.

### Password-protected Pages

Missing support in the API.

### Userbar

Only available as a Django Templates template tag or Jinja2 function. Requires:

- Authentication and authorization
- Loading of Wagtail static assets (CSS & JS)
- Loading of Wagtail UI components (HTML / Django Templates)

### Content checks

Only available via the userbar. In addition to the userbar, also requires:

- Page Preview
- Cross-domain cross-frame communication

## Open Questions

TODO
