# RFC 115: Write API

- RFC: 115
- Author: Thibaud Colas
- Created: 2026-05-06
- Last Modified: 2026-06-04

## Abstract

This RFC proposes a new official CMS API that supports programmatic creation and modification of Wagtail-managed content, including pages, snippets, revisions, and editorial workflow state transitions.
The goal is to support integrations, automation, structured content workflows, and AI-assisted tooling while preserving Wagtail’s editorial governance, permissions, accessibility, and auditability guarantees. This is based on the existing [`wagtail-write-api`](https://github.com/tomdyson/wagtail-write-api), in addition to prior work like [Revisions admin API RFC 15](https://github.com/wagtail/rfcs/blob/main/text/015-revisions-api-revised.md).

## Motivation

An officially-supported CMS admin API is a foundational requirement for us to modernize the CMS user experience and developer experience. For content authors and other CMS users, the API would support the creation of:

- CMS UI outside the CMS. For example, [commenting and workflow UIs directly on live pages](https://github.com/wagtail/roadmap/issues/49).
- Richer integrations with other site management tools. With a focus on automation.
- AI-assisted "agentic" content management. For example, to support [natural language search](https://github.com/wagtail/roadmap/issues/119) of pages.

For developers, the official API makes the above use cases possible without the risk of increasing their own projects’ levels of technical debt.
If done right with appropriate layering of the API implementation, there is also an opportunity to address long-standing gaps in Python APIs to create / manage content. Which are very relevant for tests / QA / migration scripts.

### Why now

There has been interest in this kind of API for a while ([Revisions admin API RFC 15](https://github.com/wagtail/rfcs/blob/main/text/015-revisions-api-revised.md), [Write API: Restful API for Add/Update/Remove operations #4667](https://github.com/wagtail/wagtail/issues/4667)). There are two recent changes brought about by LLMs that make it now more worthwhile:

- The effort needed to create API clients / integrations / bespoke automation is dramatically lower.
- Content management via AI agents dramatically simplifies common CMS tasks.

A common example is CMS users creating first drafts of new content outside of the CMS, and then having to click through the block chooser, split and copy-paste rich text through many fields. Simplifying this kind of tedious and error-prone task used to require costly [content import integrations](https://github.com/torchbox/wagtail-content-import) with specific tools in set input and output formats.

## Goals

The new API would simplify access to CMS operations while providing identical security and trust guarantees:

- Officially supported programmatic content creation/editing
- Preservation of revisions and audit logs
- Permission-aware operations
- Compatibility with workflows and moderation

It’d also need to support the breadth of Django/Wagtail data modeling and architecture across projects:

- Extensible for custom page models and StreamField blocks
- Extensible for arbitrary Django models / snippets
- Extensible for custom CMS operations via packages
- Reusable operations via other API frameworks, or other transport layers than REST / HTTP.

In addition, we will build an official API CLI client outside core (see [CLI client](#cli-client)).

For maintainers, the API will increase the external surface area of Wagtail core that requires support and compatibility guarantees. To mitigate that, we should also clean up and refactor to improve long-term maintainability:

- Long-term, reduces fragmentation between the public read API, internal admin API, and write operations. Near-term, we keep the existing v2 DRF read API in place and only replace the internal admin API.
- Encourages further separation of Wagtail’s CMS operations in an "actions" layer that scales well.
- Forces more streamlined permissions management.

### Incremental approach

Our goal is to add support for CMS operations over the API incrementally. For planning purposes, here are targets for support across our content model, and specific operations that support many models. Once the RFC is approved, this can be converted to a matrix so we can infer which operations to support on which models.

### Content model MVP scope

Key models ordered by importance for the API to support.

1. Pages
2. Revisions
3. Locales
4. Images
5. Rich text
6. Snippets
    1. RevisionMixin
    2. DraftStateMixin
    3. WorkflowMixin
7. StreamField
8. Documents
9. Redirects

### Supported API operations

| Action                            | Priority | Notes                                                                            |
| :-------------------------------- | -------- | :------------------------------------------------------------------------------- |
| `wagtail.create`                  | High     | The object was created                                                           |
| `wagtail.edit`                    | High     | The object was edited (for pages, saved as a draft)                              |
| `wagtail.delete`                  | High     | The object was deleted. Will only surface in the Site History for administrators |
| `wagtail.publish`                 | High     | The object was published                                                         |
| `wagtail.publish.schedule`        | Low      | The draft is scheduled for publishing                                            |
| `wagtail.publish.scheduled`       | Low      | Draft published via `publish_scheduled` management command                       |
| `wagtail.schedule.cancel`         | Low      | Draft scheduled for publishing canceled via “Cancel scheduled publish”           |
| `wagtail.unpublish`               | High     | The object was unpublished                                                       |
| `wagtail.unpublish.scheduled`     | Low      | Object unpublished via `publish_scheduled` management command                    |
| `wagtail.lock`                    | Low      | Object was locked                                                                |
| `wagtail.unlock`                  | Low      | Object was unlocked                                                              |
| `wagtail.rename`                  | High     | A page was renamed                                                               |
| `wagtail.revert`                  | High     | The object was reverted to a previous draft                                      |
| `wagtail.copy`                    | Medium   | The page was copied to a new location                                            |
| `wagtail.copy_for_translation`    | High     | The page was copied into a new locale for translation                            |
| `wagtail.create_alias`            | Low      | An alias of the page was created                                                 |
| `wagtail.convert_alias`           | Low      | An alias was converted into an ordinary page                                     |
| `wagtail.move`                    | Medium   | The page was moved to a new location                                             |
| `wagtail.reorder`                 | Medium   | The order of the page under its parent was changed                               |
| `wagtail.view_restriction.create` | Low      | The page was restricted                                                          |
| `wagtail.view_restriction.edit`   | Low      | The page restrictions were updated                                               |
| `wagtail.view_restriction.delete` | Low      | The page restrictions were removed                                               |
| `wagtail.workflow.start`          | Medium   | The page was submitted for moderation in a Workflow                              |
| `wagtail.workflow.approve`        | Medium   | The draft was approved at a Workflow Task                                        |
| `wagtail.workflow.reject`         | Medium   | The draft was rejected, and changes were requested at a Workflow Task            |
| `wagtail.workflow.resume`         | Medium   | The draft was resubmitted to the workflow                                        |
| `wagtail.workflow.cancel`         | Medium   | The workflow was canceled                                                        |
| `wagtail.comments.create`         | Low      | A comment was added to a field on the page                                       |
| `wagtail.comments.edit`           | Low      | A comment was edited                                                             |
| `wagtail.comments.resolve`        | Low      | A comment was resolved                                                           |
| `wagtail.comments.delete`         | Low      | A comment was deleted                                                            |
| `wagtail.comments.create_reply`   | Low      | A reply was added to a comment                                                   |
| `wagtail.comments.edit_reply`     | Low      | A reply to a comment was edited                                                  |
| `wagtail.comments.delete_reply`   | Low      | A reply to a comment was deleted                                                 |

Taken from [audit log docs](https://docs.wagtail.org/en/stable/extending/audit_log.html#log-actions-provided-by-wagtail).

### Non-goals

- Arbitrary ORM write access
- Replacing the admin UI
- Full parity with every admin action from the get-go
- Bulk operations support
- Real-time collaborative editing over the API

## Architecture

Here are the key architecture decisions to get right early:

- **API fragmentation vs. centralization**: whether we cater to different API needs with one or multiple APIs.
- **Operations layer**: how we structure the code of those operations to scale well.
- **Transport layer**: which HTTP API framework in core and how suitable our API would be for use via other transport layers.

### API fragmentation

We intend to support all use cases with a single API implemented in Wagtail core.

- Read-only "headless" support (v2 API) can happen via a "v3" that also supports write operations and fetching internal CMS data.
- Current internal admin API access is currently very minimal (pages explorer only), and can be supported via the same API as it would support fetching more internal CMS data.
- New use cases for "write" external CMS operations can be added.

This will be harder to build in the short term but easier to maintain long-term. This should scale well as we introduce more operations in the future. There are two clear drawbacks which we will have to mitigate:

- Compatibility commitments. We will have to respect the API versioning even when introducing new uses for internal CMS UI needs.
- Security surface area. When sites only use the API for read-only "headless" access to CMS data, we want strong guarantees that the API can be configured to only support that access.

### Operations layer

To ease creation and maintenance of more and more CMS operations over time, we will use a layered architecture where the operations are implemented separately from both the models they work on, and the transport layer of the API.

The operations layer handles:

- Business logic of the operation
- Permissions checking
- Audit logging
- Signals integration

This is documented as a reusable public API for tests, fixtures, migration scripts, management commands, etc.

This architecture can already be seen in the [wagtail.actions](https://github.com/wagtail/wagtail/tree/main/wagtail/actions) module and (unused) [wagtail.api.actions](https://github.com/wagtail/wagtail/tree/main/wagtail/admin/api/actions) module. It allows us to reuse the same business logic and ancillary actions between CMS UI usage, bulk actions, management commands, API actions, and CLI commands.

As an example, see the existing [MovePageAction](https://github.com/wagtail/wagtail/blob/main/wagtail/actions/move_page.py), which is used to implement:

- The [Page.move()](https://github.com/wagtail/wagtail/blob/4fc0b9ad85083079c2e3f9f8425599858cf36b6c/wagtail/models/pages.py#L1719-L1724) method
- The [page move CMS view](https://github.com/wagtail/wagtail/blob/4fc0b9ad85083079c2e3f9f8425599858cf36b6c/wagtail/admin/views/pages/move.py#L109)
- The (unused) [MovePageAPIAction](https://github.com/wagtail/wagtail/blob/main/wagtail/admin/api/actions/move.py)

### Transport layer

The separate transport layer would use an established Django API framework that comes with a lot of what we need to provide an API:

- Great developer experience
    - OpenAPI schema support
    - Good filtering options
- Authentication options

The transport layer would ideally allow us to provide "read" access to public data and internal CMS data without a lot of boilerplate, so we can focus our design decisions and implementation effort on CMS operations.

#### API framework

**Decision: Django Ninja** for the v3 API. Prior work in [`wagtail-write-api`](https://github.com/tomdyson/wagtail-write-api) (full write surface) and [`wagtail-ninja`](https://github.com/sinnwerkstatt/wagtail-ninja) (Ninja-based read routers for pages and redirects) demonstrates what this will look like. Why Ninja for v3:

- **OpenAPI 3.1 built-in**. No extra dependency; schema export via `manage.py export_openapi_schema` (see [QA capabilities](#qa-capabilities)).
- **Pydantic 2 schemas**. native JSON Schema export, first-class type hints, direct enabler for per-block StreamField schema discovery ([#6495](https://github.com/wagtail/wagtail/issues/6495)).

Known risks that we acknowledge and should work to mitigate:

- Single-maintainer bus factor. Mitigated by the small, readable codebase (~4.3k LoC) and the community [`django-shinobi`](https://github.com/pmdevita/django-shinobi) fork as a drop-in fallback.
- No DRF-style permission classes — v3 uses callable auth/permission decorators; Wagtail's `PermissionPolicy` integrates at the operations layer instead.
- New hard dependency: `django-ninja` + `pydantic>=2,<3`.

v3 mounts at `/api/v3/` as a `NinjaAPI` instance with domain routers (`/pages/`, `/images/`, `/snippets/`, `/schema/`, `/auth/`), following the [`wagtail-write-api`](https://github.com/tomdyson/wagtail-write-api) layout. OpenAPI docs and schema are served at `/api/v3/docs` and `/api/v3/openapi.json`.

## Foundational decisions

### Naming

To reflect our design goals and the breadth of capabilities of this API, we’d ship this as "v3" of the Wagtail API. To explain the differences, we can say v3 introduces "read and write support for all CMS data", or "support for CMS operations/actions".

### Revisions

The API would interact with revisions data where needed, rather than directly accessing the underlying model data.
Precedent in [`wagtail-write-api`](https://github.com/tomdyson/wagtail-write-api), likely suitable for core:

- Every `PATCH` / `POST` creates a revision as the current API user.
- An optional `action: "publish"` on create/update publishes the just-created revision.
- Revisions history available via list and detail views, `GET /pages/{id}/revisions/` and `/revisions/{rev_id}/`.
- Shortcut to read the latest revision: `GET /pages/{id}/?version=draft`.

For core, this is the right baseline. Additions needed:

- `POST /pages/{id}/revisions/{rev_id}/revert/` — `wagtail-write-api` exposes revisions read-only; revert is required for parity with admin UI.
- Integrate with `wagtail.models.Revision.as_object()` for snapshot reads.

### StreamField and rich text formats

We would largely replicate the input and output formats supported in [`wagtail-write-api`](https://github.com/tomdyson/wagtail-write-api), which go beyond our [Wagtail DB HTML](https://docs.wagtail.org/en/stable/extending/rich_text_internals.html) rich text representation.

#### Conversion between formats

The core API should support conversion/transformations between formats. Conversion will be a common need in integrations. Doing this in core / at the API server layer has key benefits:

- Aligning this conversion with the field-by-field rich text features restrictions.
- Stronger round-trip consistency guarantees.
- Better opportunities to support custom rich text formats.
- Code reuse with current and future use cases: rich text copy-paste support, [document importers](https://github.com/torchbox/wagtail-content-import), llms.txt rendering, rich text to StreamField conversions.

#### Rich text formats

For rich text, this means the API layer **guarantees rich text input meets the declared features**, and helps achieving that when converting between formats.

- `{"format": "html", "content": "..."}`: converted to DB HTML.
- `{"format": "markdown", "content": "..."}` — converted to DB HTML.
- Potentially - if there is a use case, `{"format": "contentstate", "content": "..."}` — converted to DB HTML.
- `{"format": "wagtail", "content": "..."}` — stored as-is (with validation or sanitisation step)
- A plain string would be passed through as-is as DB HTML.

The output format for API endpoints would be similarly configurable via a `?rich_text_format=` query parameter. Potentially with a default set project-wide via environment variable, so it’s easier to [retrieve content for site front-ends](https://github.com/wagtail/wagtail/issues/2695).

The goal would be to support a subset of standard Markdown ([CommonMark](https://spec.commonmark.org/)) and HTML as input and output, with as few Wagtail-specific idioms as possible. There will likely be a need for:

- Direct references to internal identifiers to guaranteed round-trip consistency. For example preserving references to image `id` attributes rather than `src` paths.
- Direct references to specific data types. For example comments in rich text could be represented as a `<wagtail-comment>Comment text</wagtail-comment>` [custom element](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements).

See also how WordPress represents its data model via HTML comments:

```html
<!-- wp:heading {"level":3} -->
<h3 class="wp-block-heading">Suggested Routes</h3>
<!-- /wp:heading -->

<!-- wp:list {"ordered":true} -->
<ol class="wp-block-list">
    <!-- wp:list-item -->
    <li>
        <strong>From London</strong>: Take the Eurostar to Lille, and then
        connect via Thalys to Paris. From Paris, you can catch a train to
        Cherbourg, where ferries to Dublin operate regularly.
    </li>
    <!-- /wp:list-item -->
</ol>
```

Or how [Django CMS represents object links](https://www.django-cms.org/en/blog/2026/03/13/markdown-meets-django-cms/):

```markdown
Read more on [our about page](ref:cms.page:42).
```

#### StreamField representation

The API would use Wagtail’s internal DB representation as a list of blocks with `type`, `id`, `value`. There maybe a need for a more advanced / formal schema (see [JSON Schema for StreamField #6495](https://github.com/wagtail/wagtail/issues/6495)).

Rich text within StreamField blocks would be handled with the conversions described above - finding `RichTextBlock` within the block tree and processing their value according to the target format.

### Authentication

**Decision:** Bearer token authentication via Django Ninja's [`HttpBearer`](https://django-ninja.dev/guides/authentication/) security class.

Mechanism:

- Clients send `Authorization: Bearer <token>` on every v3 request.
- Token issuance TBC - [`wagtail-write-api`](https://github.com/tomdyson/wagtail-write-api) precedent for reference:
    - Tokens are 40-character hex keys, Wagtail-specific `ApiToken` model, many-to-one user. Keys are generated with `secrets.token_hex(20)` on creation.
    - `POST /api/v3/auth/token/` — username/password exchange (returns token + username).
    - `wagtail tokens create --user=<u>` management command / CLI command for server-side token creation without password exchange.

The v3 endpoints needed to replicate v2 functionality will support anonymous access, limited to publicly-accessible data. The same endpoints will also support read and write access depending on the user’s authorizations. In the operations layer, API usage is authenticated against specific CMS users. Activity is logged as that user, constrained by the same permissions system as the admin UI. Tokens can back API-only service accounts or existing user accounts.

The "default path" should make it simple to create tokens for access-restricted accounts. Future enhancements (not v8.0):

- More granular token scoping, expiration, rotation, and naming.
- CMS-level UX differentiation between service and user accounts.
- OAuth2 / SSO token exchange flows.

### Permissions

We need to make API clients aware of the permissions available to their current session, and all CMS operations done via the API should enforce the same permissions as when done in the CMS.

Ideally we would do this without significantly increasing the complexity of the permissions-checking code. We would extend the existing `PagePermissionPolicy` / `BasePermissionPolicy` as proposed in [RFC 102: Permissions registry](https://github.com/wagtail/rfcs/pull/102).

In addition to internal refactorings, this likely requires a **`GET /api/v3/whoami/`** endpoint (and `wagtail api whoami` CLI command) returning the authenticated user, groups, permission summary, and token metadata.

### Schema and validation model

The API schema and validation model must go beyond DB and type correctness, and communicate and enforce predicates / constraints expressed in Python (panels configuration, rich text features, workflow state, permissions).

Validation happens in distinct layers, ordered by failure point:

1. **Schema layer** — request shape, types, required vs. optional fields. Generated from per-content-type Pydantic schemas (Django Ninja). Failure → **422** (Ninja/Pydantic default).
2. **Content format layer** — StreamField block types validated against registered block definitions; rich text content validated against the declared `RichTextField` features whitelist. Failure → **422**.
3. **Model layer** — `full_clean()` and other model-level constraints. Failure → **422**.
4. **Permission layer** — checked before model save. Failure → **403**.
5. **Workflow / state layer** — valid field shape but invalid workflow state. Failure → **422**.

We expect the API will use [RFC 7807 `application/problem+json`](https://datatracker.ietf.org/doc/html/rfc7807) (TBC based on early implementation choices) as the error envelope, asserted in the OpenAPI schema. Ninja's default Pydantic 422 responses will be wrapped to match this shape. Precedent for field-level detail: [`wagtail-write-api`](https://github.com/tomdyson/wagtail-write-api)'s `{"error": "...", "message": "...", "details": [...]}` structure, migrated to RFC 7807 fields (`type`, `title`, `status`, `detail`, `errors`).

#### Schema generation and discovery

Per-content-type schemas would be auto-generated from model and panel definitions, and exposed for client introspection (precedent in `wagtail-write-api`):

- `GET /schema/` — registered content types
- `GET /schema/{type}/` — read, create, patch schemas for a type
- If there is a use case: `GET /schema/blocks/{block_type}/` — per-block-type JSON schemas. See also [JSON Schema for StreamField #6495](https://github.com/wagtail/wagtail/issues/6495).

The design goal is for schemas to convey and enforce the same constraints as the admin UI:

- `api_fields` for both read and write\*\*, extending the current v2 usage.
    - TBC: `wagtail-write-api` uses an all-fields opt-out model; for core we will use `api_fields` (opt-in) with per-direction (`read`, `write`) scoping. This keeps parity with v2 and avoids accidentally exposing internal fields. [`wagtail-ninja`](https://github.com/sinnwerkstatt/wagtail-ninja) demonstrates read-side `api_fields` integration with Ninja `ModelSchema`.
- [`FieldPanel`](https://docs.wagtail.org/en/stable/reference/panels.html) configuration: `help_text`, `read_only`, `required_on_save`, `permission`, `disable_comments`.
- Capabilities of other panels, such as [`PageChooserPanel`](https://docs.wagtail.org/en/stable/reference/panels.html#wagtail.admin.panels.PageChooserPanel) page-type filtering.
- [`RichTextField` features whitelist](https://docs.wagtail.org/en/stable/advanced_topics/customization/page_editing_interface.html#limiting-features-in-a-rich-text-field) — surfaced in the schema and enforced at the block layer.
- And other constraints expressed for other field types.

There may also be a need to cache schemas client-side, with cache invalidation on Wagtail upgrade or model migrations.

It’s important the schema contains information that goes beyond data validation (like `help_text` overrides, `HelpPanel` contents, etc), so external CMS clients can provide the same information to their users.
There might not be similar opportunities for client-side data validation but we want to avoid a UX that encourages drafting content that the API would then reject.
For example, an image chooser enforcing [minimum image dimensions](https://github.com/wagtail/wagtail/discussions/12504) will likely convey this requirement via help text even if there is no formal way to constraint the chooser UI to only allow selecting images of given sizes.

#### Field access tiers

Three classes of API field access need to be distinguishable in the schema:

1. Public unauthenticated reads (v2-style headless consumption).
2. Authenticated reads (internal CMS data: locking state, draft revisions, restricted content).
3. Authenticated writes.

The schema response should reflect what the current user can actually see and modify, so OpenAPI / typed clients only advertise what's usable in a given session.

#### Workflow as validation

Some operations are valid in field-shape but invalid in workflow state, like submitting an already-published page to workflow.
We expect we will surface an `allowed_actions` (or similar) array on resource responses (HATEOAS-flavoured affordances for UI and CLI), and return **422** with explicit workflow-state detail when an invalid action is attempted.

### Compatibility and extensions

The v3 API needs explicit extension points for the breadth of Django/Wagtail data modeling and a clear policy for evolving the API while preserving compatibility.

#### Extension points

- **Custom field types**: registration hook (e.g. `@register_api_field(MyField)`) so packages can describe their JSON shape. Replaces the hardcoded `isinstance` ladder in `wagtail-write-api`'s `map_django_field()`.
- **Custom block types**: per-block JSON schema exposure should be hookable per block class. Wagtail blocks already have `get_form_class` / `get_definition` / `get_api_representation` plumbing to build on.
- **Custom operations**: package-defined actions register against the operations layer; the API exposes them via a per-resource action endpoint.
    - Likely URL pattern: `POST /pages/{id}/actions/{action_name}/` (and equivalent for snippets).
- **Lifecycle hooks**: existing Wagtail hooks (`before_create_page`, `after_create_page`, `before_publish_page`, `construct_pages_query_set`, …) fire for API operations because the API routes through the operations layer.
    - In the future, we may also introduce API-only hooks (like a `construct_api_response`, `construct_api_field`) depending on the needs of our users.
- **Signals**: API mutations emit the same Django signals (`pre_save`, `post_save`, `page_published`, …) as admin UI mutations, also for free via the operations layer.

#### Compatibility policy

- **API versioning**: URL-prefixed (`/api/v3/`), continuing with pre-existing setup.
- **Stability tiers**: endpoints documented as `stable` or `experimental`. Endpoints needed for internal CMS UI may ship as `experimental` so we can iterate without breaking external consumers on `stable` endpoints.
- **Field-level deprecation**: surfaced via schema if possible (OpenAPI `deprecated: true`) with a documented removal target. In alignment with Wagtail's compatibility policies.
- **Backwards-compat snapshot**: OpenAPI schema snapshot or snapshot subsets tested in CI. Intentional changes update the snapshot; accidental changes fail the build. (See QA capabilities below)

#### Ecosystem touchpoints

A big appeal of having the API in core will be better compatibility and opportunities for customizations in packages. Here are few examples for illustrative purposes:

- [wagtail-localize](https://github.com/wagtail/wagtail-localize): translation workflows are a primary write-API use case (like `wagtail.copy_for_translation`).
- [wagtail-ai](https://github.com/wagtail/wagtail-ai): AI-assisted CMS operations is an important motivation.
- [wagtail-headless-preview](https://github.com/torchbox/wagtail-headless-preview): preview tokens, draft reads.
- SSO / MFA auth packages: token issuance flows.

### Security

The API surface materially increases the security boundary of a Wagtail install. Existing [security reporting policy](https://docs.wagtail.org/en/stable/contributing/security.html) and deployment guidance for [logging and monitoring](https://docs.wagtail.org/en/stable/deployment/under_the_hood.html#logging-and-monitoring) apply unchanged.

#### Token management

To be confirmed: whether we implement a Wagtail-specific `APIToken` model and management commands in core (not delegated entirely to Django Ninja or a third-party auth package). Precedent: [`wagtail-write-api`](https://github.com/tomdyson/wagtail-write-api)'s `ApiToken` model and `create_api_token` management command. Token lifecycle commands (`create`, `list`, `revoke`, `rotate`) ship in the CLI package and as Django management commands.

Considerations for future releases: token hashing at rest (wagtail-write-api stores plaintext keys), named tokens, expiration dates, and per-token scopes.

#### Authentication boundaries

Focus on Bearer-token auth in v8.0. Session/cookie auth and OAuth2 flows are out of scope for the initial release.

#### CORS

API endpoints are CORS-protected. Default: no cross-origin access. Document `django-cors-headers` explicitly so headless adoption doesn't end up disabling CORS wholesale.

#### Rate limiting

Django Ninja includes built-in throttling (DRF-style rate classes).
For a first release, we would ship basic configurable throttling in v3 settings for v8.0. And document reverse-proxy rate limiting and ecosystem options (`nginx`, `django-ratelimit`) as the production default for high-traffic sites. Not a blocker for initial release.

#### Prompt injection & content sanitization

This is a high risk for an API that is designed for use with AI agents.

The API layer must use rich text allow-listing of formatting. And enforce the same constraints for image and document uploads.

API usage documentation / any official client should:

- Include measures to protect against prompt injection.
- Discourage the use of `RawHTMLBlock` / other direct HTML upload.

#### Auditability

- All API mutations emit log entries via the same `wagtail.actions.*` classes as admin UI mutations (see [Audit logging](#audit-logging)).
- In a future release, we may attach more information to log entries where available; like a client name from an optional `User-Agent` header.

#### Content provenance

Long-term, we expect we would want ways to distinguish different types of edits within the CMS. Either at the level of revisions, or per field. It’s not clear if this should be a goal in the short term.

- **Who / when / how**: captured for free via the audit log + revision authorship.
- **Human vs. automated**: surfaced on revisions and log entries.
- **AI-assisted**: finer-grained authoring source enum (`human`, `assisted`, `generated`). Adding this requires a data migration on revisions.

### Content quality

Content creation via the API would bypass our [built-in content checker](https://docs.wagtail.org/en/stable/advanced_topics/accessibility_considerations.html#built-in-content-checker) within the page editor, and not be subject to the same accessibility quality checks. Even if those checks aren’t currently enforced, this would be a clear regression compared to the current experience.

To mitigate this, we should:

- Encourage API clients to include rendering of the newly-created content as a draft, with the userbar included, which loads the content checks.
- Ship our content metrics within the userbar.
- In the future, consider storing metrics and check results against the revisions they run for, to support API retrieval.

It would be appealing to support running those metrics and checks within the API layer directly. It’s unclear how feasible this is.

### Audit logging

API mutations produce the same audit-log surface as admin UI mutations. The reference is the [audit log documentation](https://docs.wagtail.org/en/stable/extending/audit_log.html).
Any future API-specific actions will also be registered via the [`register_log_actions`](https://docs.wagtail.org/en/stable/reference/hooks.html#register-log-actions) hook, same as user-defined log actions.

## Supporting work

### CLI client

We will build an API CLI in parallel with the v3 API, as a standalone package. The existing `wagtail` binary delegates API commands to it:

```bash
wagtail api pages list --help          # → wagtail-api-v3-cli pages list --help
wagtail api schema show BlogPage
```

An official client will make it easier for us to iterate on API design. Keeping it out of core helps us move faster and attempt more experimental approaches to CLI development.

| Implementation       | Framework                                                                                                                                                                        | Role                                                                                                                           |
| :------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------- |
| **Python prototype** | [Typer](https://typer.tiangolo.com/) and [Clientele](https://github.com/phalt/clientele) or [openapi-python-client](https://github.com/openapi-generators/openapi-python-client) | Fast iteration, OpenAPI client generation via `clientele` / `openapi-python-client`, rich text / StreamField conversion reuse. |
| **Go CLI**           | To be determined (likely [`cobra`](https://github.com/spf13/cobra))                                                                                                              | Lower install barrier, single static binary, faster runtime speed.                                                             |

#### Conventions (from [`wagapi`](https://github.com/tomdyson/wagapi))

These are confirmed as the CLI contract regardless of implementation language:

- **Config cascade:** CLI flags → environment variables (`WAGTAIL_TOKEN`, `WAGTAIL_BASE_URL`) → project configuration → user configuration.
- **Output:** JSON when piped, human-readable when interactive; `--json` / `--human` overrides.
- **Output flags:** `--dry-run`, `-v` / `--verbose`, stable exit codes.
- **Error envelope:** forward the API's RFC 7807 response; never invent client-specific error shapes.
- **Distribution:** standalone PyPI package installable via `uvx wagtail-api-v3-cli`. Bundled `api` namespace delegation from the core `wagtail` package.

### Deprecation timeline for existing APIs

The v3 API replaces both the public v2 read API and the internal admin API. Both need a phased deprecation plan with explicit migration paths.

#### v2 API

v2 will be fully supported until v3 reaches feature parity:

- v3 grows read endpoints, matching v2 shapes where practical.
- v2 enters deprecation per our [deprecation policy](https://docs.wagtail.org/en/latest/releases/release_process.html#deprecation-policy).
- v2 may be extracted to [wagtailapi_legacy](https://github.com/wagtail/wagtailapi_legacy) — **TBC**, the package is currently archived on PyPI and would need reviving.
- After enough of a deprecation phase, v2 will be removed from Wagtail core.

This deferral avoids breaking existing headless deployments while the write API matures.

#### admin API

The admin API will be fully removed / replaced with v3 as part of the release that introduces v3. It is minimal (pages explorer only) and unused by most projects.

#### Migration support

Here are options, to be confirmed based on implementation choices:

- A documented mapping of v2 read endpoints → v3 equivalents, with deltas highlighted (when v3 read parity ships).
- Compatibility tests in CI to assert v3 read responses match v2 for the same resource.

### Documentation

#### New top-level docs

- **API reference**: generated from the OpenAPI schema, but with hand-written context.
- **How-to examples**: for common use cases.
- **Authentication & tokens guide**: token lifecycle, scopes, rotation, service accounts.
- **Permissions guide**: how API permissions map to Wagtail's `PermissionPolicy` (see [RFC 102](https://github.com/wagtail/rfcs/pull/102)), with examples.
- **Migration guide v2 → v3**.
- **Programmatic content model manipulation in Python** — separate guide documenting the operations layer as a usable Python API (confirmed; see [Operations layer](#operations-layer)).

#### How-to examples

- Rich text round-tripping across HTML, Markdown, and Wagtail-DB-HTML.
- Creating a page with StreamField content (multiple block types).
- Uploading and using an image.
- Scripted publishing with revision history and revert.
- Workflow transitions over the API.

### QA capabilities

We want to add those new API capabilities while minimizing the long-term maintenance effort, so it’s crucial we have appropriate ways to QA the API.

#### Core

- **OpenAPI schema snapshot in CI**: export via `manage.py export_openapi_schema` (django-ninja built-in) or fetch from `/api/v3/openapi.json`. Diff-based contract enforcement — intentional changes update the snapshot; accidental changes fail the build.
- **Permissions matrix tests**: every endpoint × every role × every action, asserting expected status codes and response shapes.
- **Validation layer tests**: each of schema / block / model / permission layers has dedicated coverage. Assert **422** uniformly for input-shape failures.
- **Rich text / StreamField round-trip tests**: across all supported input formats and feature-whitelist combinations.
- **Audit-log assertions**: every documented log action is emitted exactly once for the corresponding API call, with the expected `data` payload.
- **More advanced fixtures**: extend `wagtail/test/` test app with content shapes broad enough to exercise the API end-to-end (custom blocks, snippets with the three mixins, translated pages, custom log models).
- **Automated test case generation**: if possible - schema-driven property tests / fuzzing against Pydantic-generated schemas to surface edge cases that hand-written tests miss. Feasible because Ninja exports JSON Schema natively from Pydantic models.

#### bakerydemo

The bakerydemo project should be extended with a reproducible demo of using the API to create site content. It doesn’t necessarily need big adjustments, just enough to demonstrate how a blog page (or equivalent) can be created via the API / a CLI client.

The [headless bakerydemo](https://github.com/wagtail/bakerydemo-headless) should be updated to use the v3 API, and make more use of the OpenAPI schema in particular.

## Remaining design gaps

Decisions still open after prototyping and this revision:

| Area                   | Gap                                                                                         | Notes                                                                |
| :--------------------- | :------------------------------------------------------------------------------------------ | :------------------------------------------------------------------- |
| **Token model (v8.0)** | Hashing at rest, named tokens, expiration, scopes                                           | POC uses plaintext 1:1 user tokens; sufficient for prototyping only  |
| **Field access tiers** | How public (v2) / authenticated read / authenticated write map to schema responses per user | Three tiers identified; per-session schema filtering design not done |

## Open questions

### Do we want GraphQL mutations support?

No. Maybe later. Outside core.

### Should we support StreamField / rich text patching?

No. This is interesting but not clear the ROI is there.

### Does the API support bulk operations?

No. Maybe it will in the future. For now there is already too much to consider.

### How does the API support provenance / traceability requirements?

The first iteration would be very basic. In the future, we might provide more provenance information in audit logs / revisions. Potentially surface it at the field level (think "git blame" for CMS data).

In the meantime, projects needing this will be encouraged to create service accounts, so they can use Wagtail’s Users / Groups to understand what content was updated via the API.

### Do we really want auto-incrementing IDs in the API?

Maybe, maybe not? See [Add a UUIDField to the Page model #6162](https://github.com/wagtail/wagtail/issues/6162), and [Support customizing/encoding IDs in the API #6917](https://github.com/wagtail/wagtail/issues/6917).
The write API is viable regardless of the IDs being used but it’s a good time to revisit this if there was enough interest.

## Appendix

Here is complementary information that sits outside of the RFC to approve but is relevant to understand the work involved.

### Comparison: DRF vs. Ninja

**Outcome:** Django Ninja for v3 write/CMS API.

DRF has clear advantages from a maintenance standpoint (no dependencies, very stable, large group of maintainers) but our v3 goals would require also adding [drf-spectacular](https://github.com/tfranzel/drf-spectacular), which reduces those advantages. Keeping v2 on DRF avoids porting a stable read API; Ninja is adopted only where DRF's serializer/OpenAPI story is the bottleneck.

#### Project & community health

| Criterion            | Django REST Framework                                                                           | Django Ninja                                                                                                    | Notes                                                          |
| :------------------- | :---------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------- |
| First release / 1.0  | 2011 / 2014                                                                                     | 2020 / Nov 2023                                                                                                 | DRF is a known quantity over a full decade of Django releases. |
| Maintainer base      | Encode org, [moving to Django Commons](https://github.com/django-commons/membership/issues/188) | ~74% commits last 2y from one author; healthy fork [django-shinobi](https://github.com/pmdevita/django-shinobi) | Ninja bus factor ≈ 1, mitigated but not removed by the fork.   |
| Release cadence      | Maintenance mode, slow & steady                                                                 | 8 releases in last 7 months, feature development                                                                | Both viable; different shapes of risk.                         |
| Governance / funding | [Multiple maintainers](https://github.com/django-commons/membership/issues/188), no structure   | None visible (no `FUNDING.yml`, no `GOVERNANCE.md`)                                                             | Relevant for a CMS-core dependency.                            |
| GitHub stars (rough) | ~30k                                                                                            | ~9k                                                                                                             | Ecosystem maturity proxy only.                                 |
| License              | BSD-3-Clause                                                                                    | MIT                                                                                                             | Both compatible with Wagtail.                                  |
| Downloads per month  | 25M (52% of Django)                                                                             | 2M (4% of Django)                                                                                               | Both actively-used                                             |

#### Wagtail integration cost

| Criterion                                           | DRF                                                        | Ninja                                                                                | Notes                                                                                                 |
| :-------------------------------------------------- | :--------------------------------------------------------- | :----------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------- |
| Already a Wagtail dependency                        | Yes — v2 read API + internal admin API                     | No                                                                                   | Adopting Ninja means a new hard dep in core (+ [Pydantic v2 dependency](https://pydantic.dev/docs/)). |
| Migration cost for existing v2 read API (DRF)       | None — v2 stays on DRF                                     | N/A (v3 is additive)                                                                 | Dual-stack avoids v2 rewrite.                                                                         |
| Migration cost for existing admin API (DRF)         | None / very low                                            | Full rewrite                                                                         |                                                                                                       |
| Wagtail plugin ecosystem alignment                  | Some packages assume DRF (e.g. `wagtail-headless-preview`) | Would likely replace [wagtail-ninja](https://github.com/sinnwerkstatt/wagtail-ninja) | DRF avoids forcing an ecosystem-wide shift.                                                           |
| Integration with `wagtail.actions` operations layer | Straightforward (`APIAction` precedent already exists)     | Straightforward (operations layer is transport-agnostic by design)                   | Operations layer is the durable contribution either way.                                              |

#### Technical foundation

| Criterion                    | DRF                                                                                                                       | Ninja                                                  | Notes                                                                                                                 |
| :--------------------------- | :------------------------------------------------------------------------------------------------------------------------ | :----------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------- |
| Schema / serialization model | DRF Serializers (custom, declarative, verbose)                                                                            | Pydantic 2 (type-driven)                               | Pydantic exports JSON Schema natively — relevant for the per-block-type schema discovery requirement in §StreamField. |
| Static type-checking story   | Retrofitted; type stubs incomplete                                                                                        | First-class, `py.typed`, strict mypy in upstream       | Affects core contributors and downstream client generation.                                                           |
| Async / ASGI support         | Limited (sync-by-default, async views recently added)                                                                     | First-class (async views, auth, throttling, SSE/JSONL) | Wagtail core is currently sync; not a near-term blocker.                                                              |
| OpenAPI generation           | Requires `drf-spectacular` (extra dep), and [fixes in our implementation](https://github.com/wagtail/wagtail/issues/6209) | Built-in (OpenAPI 3.1)                                 | Ninja removes one dependency; DRF + spectacular is well-trodden outside Wagtail.                                      |
| Codebase size (auditability) | ~24k LoC                                                                                                                  | ~4.3k LoC                                              | Smaller surface = easier to debug at the framework layer.                                                             |
| Dependencies                 | None but requires [drf-spectacular](https://github.com/tfranzel/drf-spectacular) with 11 dependencies                     | `Pydantic>=2.0,<3.0` with 4 dependencies               |                                                                                                                       |
| Python / Django matrix       | Broad, conservative                                                                                                       | Broad (Py 3.7–3.14, Django 3.1–6.0)                    | Both align with Wagtail's matrix.                                                                                     |

#### Feature parity for write API capabilities

| Criterion                            | DRF                                                                                    | Ninja                                             | Notes                                                                                 |
| :----------------------------------- | :------------------------------------------------------------------------------------- | :------------------------------------------------ | :------------------------------------------------------------------------------------ |
| Permission model                     | Class-based (`IsAuthenticated`, `DjangoModelPermissions`, …) + `OPTIONS` introspection | Callable-based; no native permission classes      | DRF maps more naturally onto Wagtail's `PermissionPolicy` and the "whoami / OPTIONS". |
| Filtering                            | Built-in + `django-filter` (mature)                                                    | `FilterSchema` (Pydantic-based)                   | DRF ecosystem deeper; both workable.                                                  |
| Pagination                           | Multiple paginators built-in                                                           | `LimitOffsetPagination`, `CursorPaginator`        | Comparable.                                                                           |
| Throttling / rate limiting           | Built-in                                                                               | Built-in (DRF-style)                              | Both cover §Security rate-limiting requirement.                                       |
| Versioning                           | Built-in (URL path, namespace, header, query, accept-header)                           | URL/header (basic)                                | DRF richer; we likely only need URL path versioning.                                  |
| Content negotiation / renderers      | Mature (JSON, browsable HTML, custom)                                                  | JSON-first; multiple renderers available          | DRF browsable API is unique; arguable benefit for v3.                                 |
| Auto-generated docs UI               | Via `drf-spectacular` (Swagger/Redoc)                                                  | Built-in (Swagger + Redoc)                        | Equivalent in practice.                                                               |
| Client generation (OpenAPI → Python) | `openapi-python-client`, `clientele`                                                   | Same — both ecosystems tested against `clientele` | Relevant for the CLI decisions section.                                               |

#### Developer experience & error contract

The DRF approach is more familiar to existing Wagtail users but a lot of them have expressed frustration with its verboseness. And there is a lot of interest for [adding type hints to Wagtail](https://github.com/wagtail/wagtail/discussions/14104), which the API layer would be a good place to experiment with.

| Criterion                                    | DRF                                             | Ninja                                               | Notes                                                                                         |
| :------------------------------------------- | :---------------------------------------------- | :-------------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| API design idiom                             | Verbose - ViewSets + Serializers + Routers      | Terse - Function views + Pydantic schemas + Routers |                                                                                               |
| Error envelope                               | Loose (`detail` / field-keyed dict)             | 422 + Pydantic error list                           |                                                                                               |
| Validation surface                           | Serializer `.is_valid()` + model `full_clean()` | Pydantic schema + model `full_clean()`              | **422** for all input-shape failures (schema, block, model).                                  |
| StreamField / per-block JSON Schema export   | Manual (no native JSON Schema from Serializers) | Native (Pydantic → JSON Schema)                     | Direct enabler for [StreamField JSON schema](https://github.com/wagtail/wagtail/issues/6495). |
| Documentation quality                        | Good, long-standing, with outdated elements     | Excellent, smaller scope                            | Both are not blockers.                                                                        |
| API discoverability (`OPTIONS`, HATEOAS-ish) | First-class                                     | Not built-in                                        | Relevant if we want `OPTIONS`-driven permission introspection.                                |

#### Dependency trees

DRF itself has no dependencies, but it requires [drf-spectacular](https://github.com/tfranzel/drf-spectacular), which comes with 11. Here are the outputs of `pip tree --package <name>` for both:

##### DRF tree

```txt
drf-spectacular v0.29.0
├── inflection v0.5.1
├── jsonschema v4.26.0
│   ├── attrs v26.1.0
│   ├── jsonschema-specifications v2025.9.1
│   │   └── referencing v0.37.0
│   │       ├── attrs v26.1.0
│   │       └── rpds-py v0.30.0
│   ├── referencing v0.37.0 (*)
│   └── rpds-py v0.30.0
├── pyyaml v6.0.3
└── uritemplate v4.2.0
```

##### django-ninja tree

Note: all dependencies of Pydantic are authored / maintained by Pydantic maintainers, except for [Typing Extensions](https://github.com/python/typing_extensions) which is maintained under the CPython org.

```txt
django-ninja v1.6.2
└── pydantic v2.13.4
    ├── annotated-types v0.7.0
    ├── pydantic-core v2.46.4
    ├── typing-extensions v4.15.0
    └── typing-inspection v0.4.2
```
