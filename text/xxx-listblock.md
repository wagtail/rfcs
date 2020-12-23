# RFC X: UUIDs for ListBlock items

* RFC: X
* Author: Karl Hobley and Jacob Topp-Mugglestone
* Created: 2020-12-11
* Last Modified: 2020-12-11

## Abstract

This RFC proposes to add UUID identifiers for ``ListBlock`` items.
This allows individual ``ListBlock`` items to be tracked across revisions
which is not easy to do without unique identifies because ``ListBlock`` items can be added, removed and reordered.

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

- Change the internal representation to closely match stream block
- When looped over, it needs to expose the value of the block directly, rather than the wrapper like stream block
- It should have an ID, block type (That’s always “item”) and value fields
- Note that it needs to be distinct from existing list blocks with struct block that contains these same fields
- Use bound block to expose a stream value so we can get the ID
- Nice side effect of matching the representation is it makes it easy to migrate to a stream block later if you find that you need more than one type (use adding video to carousel example)
- Make the block able to read values in old or new formats, and convert if in old format, to make migration easy
