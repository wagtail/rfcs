# RFC 110: EXIF metadata removal of Wagtail images

- RFC: 110
- Author: Storm Heg (co-written by GitHub Copilot and language refined with ChatGPT)
- Created: 2025-09-03
- Last Modified: 2025-09-03

## Abstract

This RFC proposes that Wagtail strips sensitive [EXIF metadata](https://en.wikipedia.org/wiki/Exif) from renditions by default, to reduce privacy risks and slightly decrease file size in line with sustainability goals.

The feature will be configurable so site implementers can choose which tags to keep.

## Rationale

EXIF metadata can include camera details, timestamps, and GPS coordinates. While useful in some cases, this data can expose sensitive information (such as home address in a recipe photo). Many cameras also embed unnecessary data like thumbnails, which increase file size. Stripping such data reduces transfer/storage overhead and supports Wagtail’s sustainability goals.

## EXIF tags will be stripped (by default)

By default, Wagtail will strip the following EXIF tags from <abbr title="resized or cropped versions of the original image">image renditions</abbr>. The original uploaded image in `original_images` remains unchanged.

- GPSInfo (location data)
- Camera make and model (information about the device used to take the photo)
- Owner name (sometimes included by digital cameras, this is often the name of the camera owner)
- Software (information about the software or firmware running on the camera, which may include version numbers)
- Thumbnail (often embedded by digital cameras for their own use, not needed for web display, size is often in the order of a few kilobytes)

## Optional configuration: Retain only certain non-sensitive EXIF tags

In the default configuration, Wagtail will retain all other EXIF tags not listed in the previous section about common sensitive tags.

We will include an option that is based on an allowlist of tags to keep. These tags are important for proper display or have reasonable uses:

- Orientation (ensures correct display of images taken in portrait mode)
- ICCProfile (color profile for accurate color representation)
- ImageWidth and ImageLength (dimensions of the image)
- XResolution and YResolution (image resolution)
- Copyright (intellectual property rights)
- DateTimeOriginal (when the photo was taken)
- (any other tags determined to be non-sensitive and useful - please suggest!)

Because this feature is much more rigorous in stripping EXIF data, it will not be the default configuration and must be explicitly enabled by site administrators.

## Configurability

Recognizing that different websites may have varying requirements regarding EXIF data, for example a photography portfolio site may want to always retain camera information, this feature will be configurable via Wagtail settings.

Making this configurable allows site administrators to control which EXIF tags to strip or retain based on their specific requirements. The default configuration prioritises privacy and sustainability, but provide flexibility for other use cases.

## Comparison to other Content Management Systems

Other popular Content Management Systems (CMS) do not strip EXIF metadata from images by default. We would be pioneering privacy-conscious defaults in this area. Here is a brief overview of how some other CMS handle EXIF metadata:

- **WordPress**: retains EXIF; plugins are available to strip during upload or processing.
- **Drupal**: retains EXIF; modules exist but must be installed and configured separately.
- **Joomla**: retains EXIF; extensions can remove it, but not enabled by default.
- **Ghost**: retains EXIF; users must remove it manually before upload.
- **Squarespace**: no option to strip EXIF; users must remove it manually.
- **Wix**: retains EXIF; removal requires external tools before upload.

## Impact on users of Wagtail

The author is not aware of any Wagtail users who actively rely on EXIF metadata (please make yourself and your use-cases known!), but some may. photography-focused sites are most likely to be affected; for them, defaults can be disabled.

This will be documented in release notes as a potentially breaking change, and the default behaviour (stripping GPS and camera info) will be noted on guide.wagtail.org.

Existing images are unaffected until renditions are regenerated (via `manage.py wagtail wagtail_update_image_renditions`). Earlier behaviour can be restored by disabling EXIF processing in settings.

## Specification

This section outlines the technical implementation details for the proposed EXIF stripping feature in Wagtail.

### Process EXIF operation in Willow

The functionality to control EXIF metadata will be implemented in the Wagtail image processing pipeline, through adding a new `process_exif` operation in the [`Willow`](https://github.com/wagtail/Willow/) support library used by Wagtail for image manipulation. Doing EXIF manipulation in Willow allows for a clean separation of concerns, keeping image processing logic within Willow while allowing Wagtail to configure and utilize this functionality as needed.

Support for this operation will be limited to the **Pillow backend only**, as only Pillow provides the necessary EXIF handling capabilities. The alternative `ImageMagick` backend does NOT support EXIF manipulation, making this non-trivial to implement for that backend.

The `process_exif` operation will accept a function as argument, which will be called with the image's EXIF data as a dictionary of hexadecimal tags with constants available through `PIL.ExifTags.TAGS`.

The return value of this function will replace the existing EXIF data in the image. This allows for flexible manipulation of EXIF metadata, including stripping specific tags or modifying values as needed.

The pseudocode for the `process_exif` operation is as follows:

```python
from PIL import ExifTags
from willow.image import Image
from willow.plugins.pillow import PillowImage


def custom_modify_exif_function(exif):
    # Example: Remove GPSInfo tag
    exif.pop(ExifTags.Base.GPSInfo)

    # Example: add your own copyright tag
    exif[ExifTags.Base.Copyright] = "© Willy Wagtail | wagtail.org | CC-BY-4.0"
    return exif


with open("path/to/image.jpg", "rb") as f:
    image = Image.open(f)
    image.process_exif(custom_modify_exif_function)
    # ... further image processing & saving ...

```

It is also possible for the custom EXIF processing function to take more than one argument, for example to accept configuration options. This allows for more complex EXIF manipulation based on image specific settings. Wagtail will always pass `image` as keyword argument to the function, which is the Wagtail image instance being processed. This allows for EXIF processing functions to access fields on the Wagtail image model, such as `title`, `description`, or custom fields added through model inheritance.

Here is an example that encodes the Wagtail image's description field into the EXIF `ImageDescription` tag [^1]:

```python
from PIL import ExifTags

def apply_image_description(exif, **kwargs):
    wagtail_image_instance = kwargs.get("image")
    description = wagtail_image_instance.description
    if description:
        exif[ExifTags.Base.ImageDescription] = description
    return exif

from wagtail.images.models import Image

image = Image.objects.get(...)
print(image.description) # "Sage, Chris, Thibaud in front of the DjangoCon Europe 2025 banner"

with image.get_willow_image() as willow_image:
    willow_image.process_exif(apply_image_description, image=image)
```

### Configuration in Wagtail

A new setting `WAGTAILIMAGES_EXIF_PROCESSORS` setting will be introduced to configure which functions will be called during the image rendition generation process. This setting will be a list of callables (functions) that will be executed in sequence, each receiving the current EXIF data and returning the modified EXIF data. Later functions in the list will receive the EXIF data modified by earlier functions. To entirely disable EXIF processing, the list can be empty.

Wagtail will provide one built-in default EXIF processing function, which is also the default configuration:

- `wagtail.images.exif.strip_common_sensitive_exif`: Strips common sensitive EXIF tags such as GPSInfo and camera make/model [^2].
- `wagtail.images.exif.retain_only_common_exif`: (optional) Retains only a predefined set of non-sensitive EXIF tags, removing all others.

Site administrators can customize this setting to add their own EXIF processing functions or modify the default behaviour. For example, to add a custom function that adds a copyright tag, the configuration would look like this:

```python
WAGTAILIMAGES_EXIF_PROCESSORS = [
    'wagtail.images.exif.strip_common_sensitive_exif',
    'myapp.exif.add_copyright_tag',
]
```

How developers can write their own EXIF processing functions will be documented in the Wagtail documentation, as passing extra arguments like `image` to the function are a convention specific to Wagtail's integration with Willow. Not Willow itself.

### Integration into the Wagtail image pipeline

Wagtail will integrate the `process_exif` operation into its image rendition processing pipeline inside [`wagtail.images.models.Filter.run()`](https://github.com/wagtail/wagtail/blob/e6d828ef547ed18a5a5899baaf3ba35ae056c04f/wagtail/images/models.py#L1054), which is the only place where image renditions are generated.

It will iterate over the configured `WAGTAILIMAGES_EXIF_PROCESSORS` functions and apply each in sequence to the image's EXIF data. The final modified EXIF data will then be automatically saved when calling any of Willow's `save_as_[format]` methods.

## Open Questions

- API design: this RFC proposes a function-based configuration / API for EXIF processing. The author is open to suggestions for alternative APIs designs that may offer advantages in terms of developer experience, performance and such.
- Should we remove EXIF data from original images too? Given a rendition file url, it is possible to derive the original image file url, and then access the original image file directly. This would be a still allow access to sensitive EXIF data.
- Should we -- instead of relying on Pillow's EXIF handling -- use a dedicated EXIF library as part of the API surface in Willow? Such as [piexif](https://pypi.org/project/piexif/) (abandoned) or similar library for more robust EXIF manipulation? This make for an additional dependency to install for end-users.
  - Pillow source code considers its own EXIF handling as experimental: https://github.com/python-pillow/Pillow/blob/6a3bde05a46a8326fe02fb53fcab5a6f915d7193/src/PIL/Image.py#L3983-L3986
- Deserializing and serializing the EXIF data again may not necessarily result in byte-for-byte identical EXIF data.
  - For example, if our EXIF implementation wasn't able to deserialize certain non-standard EXIF tags, those tags may be lost upon serializing. Is this acceptable?
- Should we make an effort to implement EXIF stripping for the ImageMagick backend too? This would likely involve the use of a dedicated EXIF library as ImageMagick does not support EXIF manipulation natively. This would add more complexity to the implementation and subsequent maintenance / testing burden.

## Out of scope / future possibilities

This RFC opens the door to further possibilities, like showing EXIF data in the Wagtail admin interface, or allowing users to edit certain EXIF tags directly from Wagtail. However, these features are considered out of scope for this RFC and can be explored in future RFCs or feature proposals.

## Footnotes

[^1]: The EXIF `ImageDescription` tag is a standardised tag used to store a textual description of the image. However, it is not commonly used or widely supported across all platforms and software. Since there is little benefit to using this tag, we haven't included a function to populate it by default.
[^2]: The `strip_common_sensitive_exif` function will be implemented on best-effort basis, as non-standard tags may exist in practice that may still contain sensitive information.
