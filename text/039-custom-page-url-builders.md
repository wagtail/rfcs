# RFC 39: Page URL builder customisation

* RFC: 39
* Authors: Karl Hobley, Neal Todd, Andy Babic
* Created: 2019-04-26
* Last Modified: 2019-04-26

This RFC proposes a way to allow developers to customise Wagtail’s page URL generation logic which is currently implemented in `Page.get_url`.

A few cases where this would be useful are:

- Headless sites that may be hosted on a separate domain and/or use a different method for building URLs.
- Non-headless sites can sit behind a proxy where the user-facing domain is different from the domain of a Wagtail Site record (the domain requested by the reverse proxy).
- Internationalised sites that use `i18n_patterns` need to take into account the language of the page when reversing URLs

# Design

To implement this, we will add a new setting called `WAGTAIL_PAGE_URL_BUILDER` which provides a dotted path to a function that contains the new URL generation logic.

For example:

```python
WAGTAIL_PAGE_URL_BUILDER = 'myapp.my_build_url'
```

When this is set, All call’s to Wagtail’s `Page.get_url` and `Page.get_full_url` methods on any page will be handled by the specified function.

The signature of this function is as follows:

```python
def my_build_url(page, request=None, full_url=False):
```

It should return either a string containing the URL to the page or `None` if the page does not have a routable URL.

The parameter it takes are:

- `page` the page generate a URL for
- `request` (optional) the current request that is being handled
- `full_url` if set to `True`, the function must return a full URL (including scheme, domain and port)

## Default URL builder function

The default URL builder function is as follows. This will be provided as a snippet in the documentation so it can be copied and used as a starting point for customisation:

```python
def default_page_url_builder(page, request=None, full_url=False):
    url_parts = page.get_url_parts(request=request)
    if url_parts is None:
        # page is not routable
        return

    site_id, root_url, page_path = url_parts

    relative_url = False
    if not full_url:
        current_site = getattr(request, 'site', None)

        # Generate a relative URL if the page is on the same site
        if current_site is not None and site_id == current_site.id:
            relative_url = True

        # Generate a relative URL if there is only one site
        # Note: This allows single-site installations to still
        #       work even if they aren't configured correctly
        elif len(get_site_root_paths(request)) == 1:
            relative_url = True

    if relative_url:
        return page_path
    else:
        return root_url + page_path
```

## Implementation

I've created a PoC here: https://github.com/wagtail/wagtail/compare/master...kaedroho:custom-url-builders

In order to make the signature of the function simple, I've removed the ability to manually specify ``current_site`` to ``get_url``. In all cases
where this is used, the ``current_site`` is taken from the ``.site`` attribute of the request that is also passed in to the function.

This also means that there is no longer any difference between the ``relative_url`` and ``get_url`` methods so I think we should deprecate the ``relative_url`` method.
