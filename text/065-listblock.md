# RFC 65: UUIDs for ListBlock items

* RFC: 65
* Author: Karl Hobley and Jacob Topp-Mugglestone
* Created: 2020-12-11
* Last Modified: 2020-12-30

## Abstract

This RFC proposes to add UUID identifier for ``ListBlock`` items to allow them to be tracked across different revisions.

## Motivation

The need to have a unique identifier on ``ListBlock`` items has come up on a few occasions.

### Diffing

When items get added/removed/reorderd in a ``ListBlock``, we have no record of what the user actually did.
When generating a comparison of two list blocks, there's no way for us to know if the changed blocks were reordered, newly created, or modified.
This leads to very noisy comparisons being generated if a new block was inserted at the beginning or any reordering occurred.

If each ``ListBlock`` item was given a UUID that was saved in the revision JSON, we could quickly work out if any blocks were created/moved/deleted, then we could perform a clean comparison of the items that were modified.
This would work exactly the same as how ``StreamField`` blocks are compared today.

### Localize

Wagtail Localize is our new translation tool that works by breaking pages down into small translatable "segments".
A segment can be a complete field value (for example a complete value for a ``CharField``/``TextField``) or a piece of a field (eg, a paragraph or ``CharBlock``/``TextBlock`` value).

When Wagtail Localize breaks down a ``StreamField`` into translatable segments, it gives each translatable segment a "content path".
Content paths are constructed by concatenating the field name with any ``StreamBlock`` UUIDs and ``StructBlock`` field names that exist in parent blocks of the one where the segment was extracted (for example: ``body.<UUID for an ImageBlock>.caption``).
This content path is used later to insert the translation of the segment back into the ``StreamField``.

The problem is that there is no unique identifier for a ``ListBlock`` that is stable across revisions like there is for ``StreamBlock``/``StructBlock`` meaning that if the list block is later changed, all of the translated strings will be invalid and the entire ``ListBlock`` would have to be retranslated again.
``ListBLock`` support is currently disabled in Wagtail Localize and users must use a ``StreamBlock`` instead.

If each ``ListBlock`` item was given a UUID, we could use this UUID in the content paths.

### Commenting

- Comment positions identified by a field level contentpath (immutable, in the format `field.block_id.field`…) + a position within the field
- Blocks inside `ListBlock` do not have a stable id, so need a lot of special case logic

## Proposed solution

We will change the internal format of ``ListBlock`` to closely match the format of ``StreamBlock``.
Each ``ListBlock`` item will be represented by a JSON object with the ``id``, ``type`` and ``value`` keys.

The ``id`` key contains the item's UUID, the ``type`` key is always set to ``'item'`` (this is overridable) and the ``value`` key contains the value of the block. For example:


```json
[
    {
        "id": "<UUID>",
        "type": "item",
        "value": "The value"
    }
]
```

Unlike ``StreamBlock``, iterating ``ListBlock`` values yields values of the internal block and not a wrapper type like ``StreamValue``.
For example, iterating over the value of a ``ListBlock(CharBlock())`` will yield values with the type ``str``. In order to access the IDs, you must iterate over the ``.data`` attribute of the list block which yields ``ListValue`` objects.

### Compatibility with ``StreamBlock``

Adding the "type" here makes it easy to convert a ``ListBlock`` into a ``StreamBlock`` later.
The developer only needs to replace their ``ListBlock`` definition with a ``StreamBlock`` that implements a type for ``item`` with the same internal type that was used in the ``ListBlock``.

### Upgrading existing data

The new format will only apply to new ``ListBlocks`` and past data will not be migrated so both formats will exist in the database at the same time.

We will detect that the ``ListBlock`` is using the new format if the following is true:

 - The object has ``id``, ``type``, and ``value`` fields.
 - The ``type`` field is set to ``'item'`` 

If either of these are false, the value would be assumed to be in the old format.

The API for reading ``ListBlock`` value's from the database will be exactly the same for both formats, the only difference being is that the IDs of ``ListBlock`` items saved in the old format will be ``None``.

When the Streamfield is next saved, the format will be upgraded and IDs assigned.
