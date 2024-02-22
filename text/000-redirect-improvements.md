# RFC 0: Redirect Improvements

* RFC: 0
* Author: Jake Howard
* Created: 2024-02-22
* Last Modified: 2024-02-22

## Meta Abstract

Wagtail's current redirect system is helpful for creating short "vanity" URLs, and redirect old page locations to new. However, it can behave strangely when used in other ways, which can have a potentially negative impact on SEO.

Notably, the current implementation has the following _issues_:

- A redirect which results in another redirect means search engines spend longer indexing a site, which can severely impact search rankings.
- Redirects can be created which point to themselves, which results in them not working or shadowing other functionality
- Redirects which are intended to go to Wagtail pages are left as raw URLs. If the page is moved in Wagtail, this results in another link in the redirect chain.

This RFC proposes 3 notable changes to how redirects work, with the intention of removing potential footguns, improving SEO performance, and improving usability.

## Reject circular redirects

### Abstract

Redirects currently have little validation on the source and destination URLs. Notably, there's no validation that the source and destination are different. Depending on how `RedirectMiddleware` is deployed, this can result in infinite redirect loops, or shadow other functionality such as Wagtail pages and Django views.

### Specification

As part of the redirect creation process, redirects will confirm that the source and destination URL are different. This is relatively simple to determine for single-site deployments:

```python
redirect.old_path == Redirect.normalise_path(redirect.link)
```

And can be implemented similarly taking into account `redirect.site`.

Ensuring redirects don't point to themselves prevent users from accidentally create redirect loops, which can have larger impact in combination with the below new features.

Detecting a multi-stage loop is out of scope, as it's significantly more complicated.

This is also [an open issue](https://github.com/wagtail/wagtail/issues/11659), raised shortly before this RFC.

### Open questions

1. Should we also confirm whether the `old_path` matches an existing view?

## Allow redirects to take precedence over other aspects of Wagtail

### Abstract

Wagtail redirects are currently executed as late in the request-response cycle as possible. This ensures they only run if nothing else would otherwise create a valid response (eg Wagtail page, Django view etc). This is implemented by recommending keeping the middleware as late in `MIDDLEWARES` as possible, only running after the rest of Django's view system has executed.

However, this also means redirects happen much later than may be expected. To many, a "redirect" should take priority over other implementations in the system, rather than afterwards.

### Specification

An `is_eager` column will be added to a `Redirect`, which flags whether the redirect should run before the rest of the Wagtail page process, or afterwards (as they would currently). `is_eager` will default to `False`, to ensure it's not a breaking change, or impact other site behaviours.

To properly support `is_eager`, the `RedirectMiddleware` will also need bringing up the `MIDDLEWARES` list, below incredibly high middleware such as `SecurityMiddleware`, `GzipMiddleware` and `WhitenoiseMiddleware`, but still as high as possible. For redirects to apply before Django's `APPEND_SLASH` functionality, and thus keep the redirect chains short, the `RedirectMiddleware` must come before `CommonMiddleware`.

A Django check will be added to warn users when `RedirectMiddleware` is placed below `CommonMiddleware`.

The redirects list in the Admin will also allow filtering by "eager" redirects, to more easily find a redirect.

## Convert URL redirects to page redirects

### Abstract

Wagtail supports 2 types of redirect "destination". A redirect can either point to a URL, or directly to a Page instance. Pointing to a page has a number of benefits, notably around the page moving or changing in future, as the redirect's destination will update too.

Users can enter the URL to a Wagtail page, and it'll stay that way. If the page is moved in future, the redirect isn't updated. Wagtail automatically creates redirects when pages are moved, so users still get to the end page, but it does result in a redirect chain.

### Specification

I propose Wagtail automatically update URL redirects which point to Wagtail pages to reference the page instead. An example implementation exists in [#6465](https://github.com/wagtail/wagtail/issues/6465).

This adds additional overhead to creating redirects, which will impact how many redirects can be imported in bulk. We should ensure the `import_redirects` management command is well documented as a performant alternative which isn't susceptible to request timeouts.

### Open questions

1. Should this be automatic, or a warning during creation or a report? Adding links in RichText automatically detects internal links, but prompts the user. This might be more difficult in a conventional creation flow.
