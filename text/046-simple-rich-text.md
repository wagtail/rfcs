# Outline RFC 46: Rich text segment blocks

* RFC: 46
* Authors: Karl Hobley
* Created: 2019-11-18
* Last Modified: 2019-11-18

## Abstract

This is an outline proposal for a new field and block type which would behave like a ``RichTextField``/``RichTextBlock`` but only allows a single segment of text.

Inline elements such as formatting and hyperlinks may be allowed. But block elements like paragraphs, images and bullet points will not. The HTML segment in the database will not be enclosed in a `<p>` tag giving developers more flexibility in where it can be used.

I think this will be very useful for:
 - Introductions/summaries which can usually only be one paragraph but use rich text fields as they may need inline formatting. For example, the sentence above "Tom Dyson" in https://wagtail.io/blog/wagtail-2-7/.
 - Image captions
 - Inline styling in titles. For example https://torchbox.com/digital-products/
 - Forcing editors to use separate rich text blocks in streamfields

I think it would also be very useful to have an internal ``RichTextSegment`` type to represent the values of these fields. Having a separate type rather than reusing ``RichText`` would be useful because logic, like diffing and translating, would work very differently when working with individual segments.

## Specification

I don't think this would be a very technically challenging change to make, but I'm leaving this bit out since I'm only looking to get feedback on the general idea for now.
