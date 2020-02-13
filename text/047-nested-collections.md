# RFC 047: Nested Collections

* RFC: 047
* Author: Seb Jansen
* Created: 2020-02-13
* Last Modified: 2020-02-13

## Abstract

It is currently not possible to nest collections. Having this ability would improve our ability to organise media files.

## Specification

Possible implementation including tests, can be found here: https://github.com/SebJansen/wagtail/commit/71bb9267a176603ac4c77086fb1e86ec6e17dc1e

The collection model already supports nested collections since it uses Treebeard. However, the collection views and forms had to be changed to support changing depth and this could be construed as too big of a change. 
The existing collection chooser is slightly modified with space-indentations to designate depth. Spaces aren't the best solution, but Treebeard uses them as seen in `mk_indent` in `MoveNodeForm`. 
If user has permission on parent collection, permission is automatically given for child collections. This might be too permissive.
If user has only has access to child collection, don't show parent collection in collection chooser. This might not be expected behavior and hinder .

Preexisting sorting for collections is changed and is now based on its path, to correctly display hierarchical relationships. This might upset some existing expectations regarding ordering.   

### Possible implementation


## Open questions

- Is the default `<option>` collection chooser in the frontend adequate?
- Should we make it more performant? (e.g. looking at the `format_collection` template tag)
- Is security compromised in any way?