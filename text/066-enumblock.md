# RFC 66: EnumBlock

* RFC: 66
* Authors: Karl Hobley
* Created: 2020-12-30
* Last Modified: 2020-12-30

## Abstract

This RFC proposes to add a new ``EnumBlock`` block type into Wagtail.
This would be a new structural block type that allows the user to choose the type of a single block from a list of choices.

## Examples

### Link chooser

Perhaps the most common use for this block type would be adding link choosers that support both internal and external links. For example:

```python
from wagtail import blocks


class LinkBlock(blocks.EnumBlock):
    internal = blocks.PageChooserBlock()
    external = blocks.URLBlock()
```

This could also be used at the root of the ``StreamField`` so that a link field could be added to a model:

```python
from wagtail.fields import StreamField
from wagtail.models import Page

from .blocks import LinkBlock


class MyPage(Page):
    link = StreamField(LinkBlock())
```

### Form fields

Another use case this could be helpful for is when you have a list of blocks that share common attributes but have some additional type-specific attributes. For example, a form field type:

```python
from wagtail import blocks


class TextFieldBlock(blocks.StructBlock):
    max_length = blocks.IntegerBlock()


class ChoiceFieldBlock(blocks.StructBlock):
    choices = blocks.ListBlock(blocks.CharBlock())


class FormFieldBlock(blocks.StructBlock):
    name = blocks.CharBlock()
    required = blocks.BooleanBlock()
    field_type = blocks.EnumBlock([
        ('text', TextFieldBlock()),
        ('choices', ChoiceFieldBlock()),
    ])
```

## Implementation details

I've started an implementation on [kaedroho/enum-block](https://github.com/wagtail/wagtail/compare/master...kaedroho:enum-block). It's currently missing tests and UI.

### JSON representation

The JSON representation would be similar to ``StreamBlock``, the differences are there will be only one item so there's no need for a list, and the item will not have an ``id``.

For example, an external link using the block definition in the "Link chooser" example above would look like the following in the database:

```json
{
    "type": "external",
    "value": "https://wagtail.io"
}
```

One exception to this is when an ``EnumBlock`` is used in a ``ListBlock``. In this case, the ``EnumBlock``, ``type`` field will be added to the ``ListBlock`` item, alongside its existing ``id`` field. For example:

```json
[
    {
        "id": "<List block item ID>",
        "type": "paragraph",
        "value": "Hello world!"
    }
]
```

This assumes that [RFC 65](https://github.com/wagtail/rfcs/pull/65) is implemented first.

This means that ``ListBlock(EnumBlock(X))`` always has the exact same database representation as ``StreamBlock(X)`` which will simplify switching between the two. I think this would also make it easier to implement StreamField migrations later, as this would mean we only have three simple structural types to deal with (``StructBlock``, ``ListBlock``, and ``EnumBlock``. ``StreamBlock(X)`` can be interpreted as ``ListBlock(EnumBlock(X))`` for the purpose of data migrations).
