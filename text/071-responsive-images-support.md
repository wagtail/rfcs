# RFC 71: support for responsive images

- RFC: 71
- Author: Thibaud Colas
- Created: 2021-08-27
- Last Modified: 2021-08-27

> 🚧 A prototype implementation of this proposal is available for review and testing as a standalone package: [wagtail_picture_proposal#3](https://github.com/torchbox/wagtail_picture_proposal/pull/3). Discussions: [#lean-images on the Wagtail Slack](https://github.com/wagtail/wagtail/wiki/Slack).

## Abstract

We propose extending Wagtail’s [image rendering](https://docs.wagtail.io/en/stable/advanced_topics/images/renditions.html) to support generating multiple renditions of a given image at once. This will help implementing [responsive images](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images) on Wagtail sites. This has been discussed for a while ([#285](https://github.com/wagtail/wagtail/issues/285)) and there are known workarounds, but they fall short on performance and ease of use.

One of the key goals behind this proposal is to greatly reduce the carbon impact of sites built with Wagtail. With appropriate support for responsive images rendering, and complementary practices, we are hoping current unoptimised implementations can be refactored to achieve up to a 10x reduction in bandwidth needed to load image-heavy pages. This figure is consistent with the [sample site homepages we tested](#performance-impact).

## Current workarounds

Site implementers are already generating multiple renditions for responsive images by using the `{% image %}` tag multiple times, however this type of code becomes very verbose and the implementation has fundamental performance shortcomings. Here is a typical implementation:

```twig
<picture>
  {% image page.test_image width-800 format-webp as image_desktop_webp %}
  {% image page.test_image width-400 format-webp as image_mobile_webp %}
  <source
    srcset="{{ image_desktop_webp.url }} {{ image_desktop_webp.width }}w, {{ image_mobile_webp.url}} {{ image_mobile_webp.width }}w"
    sizes="(max-width: 600px) 480px, 800px"
    type="image/webp"
  />
  {% image page.test_image width-800 as image_desktop_fallback %}
  {% image page.test_image width-400 as image_mobile_fallback %}
  <img
    srcset="{{ image_desktop_fallback.url }} {{ image_desktop_fallback.width }}w, {{ image_mobile_fallback.url}} {{ image_mobile_fallback.width }}w"
    sizes="(max-width: 600px) 480px, 800px"
    src="{{ image_desktop_fallback.url }}"
    alt="original"
  />
</picture>
```

This generates four variants of the same image. The resulting code is very verbose, and using the `{% image %}` tag four times results in four sequential database queries, which is inefficient.

## Proposed approach

We propose adding support for generating multiple renditions at once to the main `{% image %}` tag, and to a new `{% picture %}` tag, in a backwards-compatible way.

- In the case of `{% image %}`, we would generate an image with the [`srcset`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attr-srcset) attribute with width descriptors.
- `{% picture %}` will generate a [`picture`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture) element, with a `source` child per supported image format, and a `img` fallback.

### Proposed API

The most important API is the way to specify which renditions to generate – our preferred option at this stage is to extend the current syntax for filter specifiers to support [brace expansion](https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html):

```jinja2
{% image page.test_image width-{200,400,600} sizes="100vw" %}
```

This would generate a single `<img>` tag with an `srcset` attribute with URLs of three renditions: `width-200`, `width-400`, `width-600`. We chose to leverage existing syntax to facilitate understanding of how the specifier works. The `sizes` attribute is passed as-is, and will need to be set so the browser loads the correct image.

To keep the implementation simple, we recommend only supporting a single brace expansion per `image` tag. For example, `fill-{300x400,400x600,900x1200}` would be valid, while `fill-{300,400,900}x{400,600,1200}` would be invalid.

### Examples

Here are three examples of using those template tags:

- [`homepage.html` in our prototype demo site](https://github.com/torchbox/wagtail_picture_proposal/pull/3)
- [Refactoring the bakerydemo homepage](https://github.com/thibaudcolas/bakerydemo/pull/4)
- [Refactoring torchbox.com/wagtail-cms](https://github.com/thibaudcolas/wagtail-torchbox/pull/1)

Sample diffs for illustration:

```diff
bakerydemo:
-{% image childpage.image fill-430x254-c100 %}
+{% webp_picture_wip childpage.image fill-{430x254,335x198}-c100 q-60 sizes="30vw, (max-width: 375px) 335px" loading="lazy" %}
torchbox.com:
-{% image item.author.image fill-100x100 class="avatar__image" %}
+{% webp_picture_wip item.author.image fill-100x100 q-80 class="avatar__image" loading="lazy" %}
```

## Complementary changes

Along with the API to generate multiple renditions at once with brace expansions, we propose making a few additional changes to image rendering to make it more flexible.

### Variables for filter specifiers

```jinja2
{% image page.test_image three_widths %}
```

with `three_widths` defined as a filter specification string in the template context:

```python
three_widths = "fill-{300x400,400x600}"
```

This simple change would make it possible for implementers to create abstractions around which renditions are generated should this be desirable – for example centrally defining all renditions in Django settings.

### Template rendering for images

To support the more advanced image rendering, we should also switch images’ HTML to be driven by a template. This will make it possible for implementers to customise how images are rendered site-wide if desired, for example to control which attributes are added to `<img>` tags by default.

Here is the default template we are expecting at this stage:

```twig
{# Placement of template syntax done to maximise readability of output HTML. #}
<img
  {% if fallback_renditions|length > 1 %}srcset="{% for rendition in fallback_renditions %}
      {{ rendition.url }} {{ rendition.width }}w{% if not forloop.last %},{% endif %}{% endfor %}"{% endif %}
  src="{{ fallback_renditions.0.url }}"
  width="{{ fallback_renditions.0.width }}"
  height="{{ fallback_renditions.0.height }}"{% for name, value in attributes.items %}
  {{ name }}="{{ value }}"{% endfor %}
>
```

## Other implementation changes

### Renditions generation performance

As part of implementing those changes, we will make sure the new implementation has as good of a performance at all potential hot spots:

- Querying and inserting all renditions at once, as opposed to one by one sequentially
- Generating rendition images and saving to file storage, as now
- Making sure all renditions are cached, as now

Sample bulk querying code for reference:

```python
# We need to get renditions that have both attributes matching in pairs.
q_objects = Q()
for filter, cache_key in rendition_params:
    q_objects |= Q(filter_spec=filter.spec, focal_point_key=cache_key)

renditions = list(self.renditions.filter(q_objects))
```

Sample bulk insertion code:

```python
bulk_objs = []
# Only bulk-creating for renditions that weren’t fetched by the previous query.
for filter, cache_key in missing_rendition_params:
    bulk_objs.append(Rendition(
        image=self,
        filter_spec=filter.spec,
        focal_point_key=cache_key,
        file=File(generated_image.f, name=output_filename),
    ))

created_renditions = list(Rendition.objects.bulk_create(bulk_objs))
```

### Store output as variable

In the case of storing the output of `{% image %}` or `{% picture %}` as a variable, we can simply provide the same data that would normally be provided to the tag template’s context – a list of renditions for `image`, a more complex data structure for `picture`. Here is an example with `{% image %}`:

```twig
{% image page.test_image width-{200,400,600} as srcset %}

<style>
  #test-bg-image {
    min-height: 300px;
    background-repeat: no-repeat;
    background: url("{{ srcset.0.url }}");
  }
  @media screen and (min-width: 601px) {
    #test-bg-image {
      background: url("{{ srcset.1.url }}");
    }
  }
  @media screen and (min-width: 801px) {
    #test-bg-image {
      background: url("{{ srcset.2.url }}");
    }
  }
</style>
<div id="test-bg-image">
  <p>test background image</p>
</div>
```

This is a representative example of how variables output is used currently, however this type of pattern shouldn’t be encouraged as it prevents images from being lazy-loaded with the native `loading="lazy"` attribute, and doesn’t offer a simple way to add alternative text for screen reader users, and when the image fails to load.

### Programmatic retrieval of renditions

The existing [`get_rendition_or_not_found` shortcut](https://github.com/wagtail/wagtail/blob/main/wagtail/images/shortcuts.py#L4) would be kept as-is, with a new `get_renditions_or_not_found` added to retrieve multiple renditions at once.

## Opinionated implementation choices

- The `srcset` attributes will use width descriptors, with no option to use pixel density descriptors. They aren’t expected to be as valuable, and if needed could be added bespoke by site implementers by customising the new image HTML template.
- The `picture` element won’t use `media` attriutes for art direction. This will only be possible when customising the new tag’s template.
- For both tags, the fallback image `src` will come from the first rendition generated in `srcset`.
- The image’s `width` and `height` will be set based on the dimensions of the first rendition generated in `srcset`, with CSS needed to apply any overrides per-breakpoint.

## Future evolutions

Here are potential future evolutions which we have considered as part of researching this topic and seemed promising but we chose not to pursue further.

### Named filter specifiers

For example, specifying in templates:

```twig
{% image page.test_image thumbnails-plus loading="lazy" %}
```

Combined with a Django setting (or other type of registry) like:

```python
WAGTAIL_NAMED_FILTERS = {
    "thumbnails-plus": "fill-{160x120,300x200}",
}
```

We chose not to pursue this for two reasons:

1. Creating this type of filters’ registry makes it harder to add new image sizes to a site, which is currently a templates-only change.
2. For projects which want to enforce a fixed set of sizes by increasing the friction of adding image sizes like this, it will be possible to do so with the new support for variables for filter specifiers.

### Parallel generation of renditions images

To run the filters processing in parallel rather than sequential:

```python
for filter, cache_key in missing_rendition_params:
    # Generate the rendition image
    generated_image = filter.run(self, BytesIO())
```

We expect this to be an additional performance improvement, but didn’t pursue it further due to a lack of knowledge in the performance characteristics of thread-based parallelism in Django application servers.

### Generating renditions with the dynamic rendition view

Wagtail already supports generating renditions on request of each image. The built-in tags could be switched over to using this view, by generating signed URLs for it. The database cost of renditions would be the same, but the image processing would only happen as needed.

This feels promising but we chose to not explore it further for now. One potential concern is the additional server load in serving images – as of now images for most cloud deployments of Wagtail images are taxing only for the first load of the pages, and are served by dedicated file server afterwards.

### Dynamic rendition view: Save-Data

Further to the above, dynamically generating renditions would also make it possible to implement the [`Save-Data`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Save-Data) HTTP header, thereby allowing Wagtail sites to compress images to different levels depending on user preference.

### Pluggable image rendition generation backbends

Similarly to the above, it seems promising to have a way to altogether delegate image renditions generation to an external image resizing API – for example Cloudinary, Cloudflare Image Resizing, or custom implementation running on edge servers.

### Image placeholders

With template rendering for images, it will be much simpler for implementers to enhance their images with placeholders to defer loading more aggressively than the browser does. See for example [Wagtail Lazy Images](https://github.com/ptrck/wagtail-lazyimages).

## Performance impact

Here are reports of applying responsive images and WebP `picture` tags to existing sites with our [wagtail_picture_proposal prototype](https://github.com/torchbox/wagtail_picture_proposal/pull/3), to compare the size of image payloads before/after. For the performance of the page’s generation, we expect any change to be minimal since:

- Database access has been optimised to make as few queries as possible
- Renditions will be cached in any configured Django cache backend

### bakerydemo home

Rebuilding the image-heavy homepage of [bakerydemo (PR #4)](https://github.com/thibaudcolas/bakerydemo/pull/4) to demonstrate the potential, with different levels of changes.

| Viewport | Test case                    | Image weight | Relative |
| -------- | ---------------------------- | ------------ | -------- |
| desktop  | baseline, no change          | 402 kB       | 1.00     |
| iPhone 6 | baseline, no change          | 402 kB       | 1.00     |
| desktop  | srcset                       | 402 kB       | 1.00     |
| iPhone 6 | srcset                       | 229 kB       | 0.57     |
| desktop  | srcset, WebP                 | 240 kB       | 0.60     |
| iPhone 6 | srcset, WebP                 | 135 kB       | 0.34     |
| desktop  | srcset, WebP, loading="lazy" | 132 kB       | 0.33     |
| iPhone 6 | srcset, WebP, loading="lazy" | 55 kB        | 0.14     |

### torchbox.com Wagtail home

Rebuilding the Wagtail service page on [torchbox.com (PR #1)](https://github.com/thibaudcolas/wagtail-torchbox/pull/1).

| Viewport | Test case                    | Image weight | Relative |
| -------- | ---------------------------- | ------------ | -------- |
| desktop  | baseline, no change          | 1900 kB      | 1.00     |
| iPhone 6 | baseline, no change          | 1800 kB      | 1.00     |
| desktop  | srcset                       | 1800 kB      | 0.95     |
| iPhone 6 | srcset                       | 310 kB       | 0.17     |
| desktop  | srcset, WebP                 | 538 kB       | 0.28     |
| iPhone 6 | srcset, WebP                 | 111 kB       | 0.06     |
| desktop  | srcset, WebP, loading="lazy" | 534 kB       | 0.28     |
| iPhone 6 | srcset, WebP, loading="lazy" | 107 kB       | 0.06     |

## Open Questions

- Are there more intuitive syntaxes than brace expansion?
- What should new tag(s) be called?
- For `picture`, is it useful to be able to specify quality at once for both JPEG and WebP with `q-90`? Or would we be better off supporting setting the two separately only, with `jpegquality-90 webpquality-90`?
- Setting `loading="lazy"` on all images site-wide would be an incredible performance improvement – is there a good way to achieve this? If not, should we add one?
- What would be a good example of using output variables that would still retain lazy-loading support and as good accessibility as possible?

## References

- Wagtail: [Create a tag for the picture element + support for responsive image sets #285](https://github.com/wagtail/wagtail/issues/285)
- Wagtail: [Allow images to be generated without width and height attributes #5289](https://github.com/wagtail/wagtail/issues/5289)
- Willow: [[WIP] Image optimisation operations #69](https://github.com/wagtail/Willow/pull/69)
- wp_image tag from [@ababic](https://github.com/ababic): <https://gist.github.com/thibaudcolas/3c6b9c354e7d636f08133f93b65e7978>
- <https://gist.github.com/coredumperror/41f9f8fe511ac4e88547487d6d43c69b>
- <https://github.com/ephes/wagtail_srcset>
- <https://github.com/ptrck/wagtail-lazyimages>
- <https://github.com/jams2/wagtail-responsive-images>
- <https://pypi.org/project/easy-thumbnails/>
- <https://nextjs.org/docs/api-reference/next/image>
- <https://www.gatsbyjs.com/plugins/gatsby-plugin-image>

## Credits

The bulk of the code in the prototype comes from [wagtail/wagtail](https://github.com/wagtail/wagtail). Additional modifications by Thibaud Colas, with support from Chris Lawton. WebP picture tag prototype by Andy Babic. Thank you to Jane Hughes and Lara Thompson for early feedback on the proposed APIs.
