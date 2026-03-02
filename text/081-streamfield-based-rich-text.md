# RFC 81: StreamField-based rich text

* RFC: 81
* Author: Matthew Westcott
* Created: 2022-11-24
* Last Modified: 2022-11-24

## Abstract

This RFC proposes a new implementation of rich text that leverages the StreamField data model for managing content as a sequence of blocks, while preserving the familiar Word-like user interface as closely as possible.

## Specification

### Rationale

Content editors often have an aversion to any editing interface that doesn't have the look and feel of Microsoft Word. No matter how much better StreamField is than rich text on technical grounds as a basis for managing content - as long as rich text feels like a Word document and StreamField doesn't, editors will demand rich text, and developers will often have limited ability to push back. Consequently the site ends up failing to benefit from the feature set of StreamField (more diverse content types, better control of front-end rendering, data exports in a structured format, and so on), making Wagtail look less capable than it really is.

### Proposal

Reimplement rich text with a Word-like user interface, but with StreamField as the underlying data model.

### The theory

Rich text, as implemented by Draftail / draft.js, has a two-level data model. At the top level, it is a sequence of block-level elements - headings, paragraphs, list items, block quotes, images, embeds - of which some have text-based content, and some do not. This sequence is a flat list, with no concept of nesting elements (hierarchical lists are implemented by giving each item a 'depth' attribute instead).

Each text-based block then consists of a plain text string, along with a list of styles (bold, italic, underline, strikethrough, subscript, superscript) and inline entities (primarily links, but could be anything that attaches arbitrary properties to a span of text, such as footnotes, stock symbols, usernames, or custom emoji) to apply to specified character ranges within that string. For example, a paragraph might be represented as:

    {
        block_type: "paragraph",
        text: "A wagtail is a bird.",
        styles: [
            {type: "bold", offset: 2, length: 7},
        ]
        entities: [
            {
                type: "external-link", offset: 15, length: 4,
                attributes: {url: "https://en.wikipedia.org/wiki/Bird"}
            },
        ]
    }

(This example is inspired by the contentState format used by draft.js, but does not follow it rigorously.)

The "sequence of block-level elements" aspect maps well to StreamField; the inner "styled text" representation does not. As such, we will introduce a new block type, named ParagraphBlock, to handle formatting within a single paragraph element (or other block-level element, such as a heading). This behaves similarly to an existing RichTextBlock, but only allows inserting inline styles and entities, not new block-level elements. In its native form (as seen when it is a child of a StructBlock, for example), pressing enter (to insert a new paragraph) has no effect - however, the 'capabilities' mechanism (as currently used for splitting blocks) will introduce this ability.

The single-paragraph rich text editor will also be available as a standalone form widget for use in non-StreamField content that requires a single paragraph of rich text, such as article intro copy. When outputting this value on a template, the outer `<p>` element will not be included as standard, allowing the template author to specify their own markup such as `<p class="introduction">{{ page.introduction }}</p>`.

It is yet to be determined whether this rich text widget will be implemented with Draftail, some other editor component, or an entirely custom implementation based on the browser's contentEditable support. Given the need to integrate tightly with StreamField logic in areas such as keyboard control, and the reduced scope (only having a single block-level element to manage), there may be limited value in using an off-the-shelf component.

ParagraphBlock will accept a `features` keyword argument to define the set of elements allowed, but only features corresponding to inline styles and entities will be meaningful.

### Multi-paragraph editing

A regular multi-paragraph rich text field will be implemented as a StreamField where a ParagraphBlock is one of the available blocks, along with other block types (such as image or video embed) as defined by the field's `features` argument. Other text-based block-level elements (such as headings and blockquotes) will be defined as additional distinctly-named instances of ParagraphBlock, with appropriate styling. For example, a RichTextField defined as:

    body = RichTextField(features=['bold', 'italic', 'image', 'link', 'h2', 'h3'])

would be functionally equivalent to:

    body = StreamField([
        ('paragraph', ParagraphBlock(features=['bold', 'italic', 'link'])),
        ('image', ImageBlock()),
        ('h2', ParagraphBlock(features=['bold', 'italic', 'link'])),
        ('h3', ParagraphBlock(features=['bold', 'italic', 'link'])),
    ])

(where ImageBlock is a StructBlock consisting of an image chooser, alt text field and alignment selector)

Similarly, a RichTextBlock inside a StreamField can be translated to a StreamBlock definition.

It is crucial that editing actions spanning multiple paragraph blocks can be performed without visibly leaving the context of an existing ParagraphBlock widget - for example, pressing enter should create a new ParagraphBlock, without the need to explicitly insert one from a menu. To do this, ParagraphBlock will make use of the StreamField 'capabilities' mechanism to identify that it is contained within a parent block that manages a sequence of children and exposes various API methods for splitting and inserting blocks. When these API methods are available, ParagraphBlock will configure itself with additional keyboard controls and menu items to take advantage of them.

### Keyboard interactions - navigation

ParagraphBlock will expose API methods that allow the block to be focused and the caret placed at the start or end of the text - a capability known as "end-focusable". ListBlock, StreamBlock and StructBlock can also easily implement this capability, if their child blocks are end-focusable themselves - by making the corresponding API call to the first or last of their children as appropriate.

ListBlock, StreamBlock and StructBlock will provide capabilities that allow child blocks to check the capabilities of the previous and next blocks in the sequence.

If the up or left cursor key is pressed while the caret is at the start of a ParagraphBlock, and the previous block in the sequence is end-focusable, then the previous block will be given focus with the caret placed at the end.

If the down or right cursor key is pressed while the caret is at the start of a ParagraphBlock, and the next block in the sequence is end-focusable, then the next block will be given focus with the caret placed at the start.

### Keyboard interactions - block insertion and deletion

If the enter key is pressed while a ParagraphBlock is focused, and the parent block allows insertion of new blocks (as StreamBlock and ListBlock do, but not StructBlock), the ParagraphBlock will be split at the caret position into two ParagraphBlocks of the same type, and the second one will be given focus with the caret placed at the start.

If the backspace key is pressed while the caret is at the start of a ParagraphBlock, and the parent block allows deletion of blocks, and the previous block in the sequence is also a text-based block, then the content of the current block will be appended to the previous block, the current block will be deleted, and the previous block will be given the focus at the start of the newly-moved text.

(These rules are not exhaustive - others may be added, such as the ability to delete an embedded image/video by backspacing from the paragraph after it.)

### Toolbars and changing block type

ParagraphBlock will provide a toolbar for inserting inline styles, inline entities and blocks. The design for this is yet to be determined. However, it is proposed that it should be permanently visible while the block is focused, positioned at the top of the block - or if the ParagraphBlock is one of a sequence (within a StreamBlock or ListBlock), at the top of the first block in the sequence. If this would result in the toolbar being off-screen, it will be anchored to the top of the screen instead.

A StreamBlock will allow a block within it to read the list of available block types; ParagraphBlock will use this to populate the toolbar with buttons for inserting those block types.

On clicking one of those toolbar buttons - or some other equivalent action such as entering the '/' or '#' shortcuts - if the chosen block type is text-based (e.g. a heading or blockquote), the active ParagraphBlock will be replaced with a block of that type, populated with the previous ParagraphBlock's content.

If the chosen block type is not text-based (e.g. an image), the active ParagraphBlock will be split into two at the caret position (deleting any selected text), and a new instance of the chosen block will be inserted between them and focused.

### Undo/redo

The StreamField as a whole (or potentially the whole edit form) will maintain an undo / redo buffer so that block deletions and insertions can be undone, rather than just edits within a single paragraph block.

### Copy and paste

Content pasted into a ParagraphBlock will be intercepted, where browser capabilities allow, and split into paragraphs. If the parent block allows block insertion, as many new blocks will be inserted as necessary to fit the content. Where possible, the markup / style of each pasted paragraph will be matched to the most suitable block type out of all the ParagraphBlock types available on the container - headings, blockquote and so on. (If the parent block does not allow block insertion, only a single paragraph will be pasted - either cutting off after the first paragraph, or concatenating everything into one paragraph.)

### Data representation

This change will mean that the in-database representation of a RichTextField or RichTextBlock will change from the current HTML-like string to the StreamField JSON format.

The data format for an individual ParagraphBlock is also up for consideration: while it could feasibly adopt the existing HTML-like string format, there is probably a strong case for taking this opportunity to switch to a JSON format like contentState. Originally the HTML-like format was chosen to minimise the processing required to transform it into real front-end HTML, but over time additions such as commenting and `data-block-key` attributes have indicated a need to capture information within rich text that isn't reflected in the front-end rendering, and simple regexp replacement increasingly feels like too blunt an instrument for this. It's also arguably a good thing for developers to move away from the mental model of rich text as a "flavour" of HTML - so that features with only a loose relation to HTML (e.g. footnotes) do not have to be approached from the angle of HTML, and common questions of the "how do I enable `<button>` as an allowed element in rich text" variety are asked at the correct level of abstraction.

Retrieving the value of a RichTextField or RichTextBlock in user code will now yield a StreamValue. For simple template rendering cases, this should work with no code changes - a tag such as `{{ page.body }}` will return the StreamValue's `__str__` representation, which will be the HTML rendering as desired. The `|richtext` filter will become redundant, and we would encourage authors of new code to use `{% include_block page.body %}` instead. The native value type of ParagraphBlock will also be a custom Python type with a `__str__` method returning an HTML rendering.

It's very likely that there will be some incompatibilities with legacy code that expects a string value (e.g. code that performs string replacement or slicing on it, or queries the field with an `__icontains__` filter), and so this will most likely entail a major version bump of Wagtail.

To accommodate existing API clients that consume the legacy HTML-like string format, we will provide a DRF serialiser to translate the StreamField JSON format back to the HTML-like format - and, if possible, configure our API endpoints to use this by default for `/api/v2/` endpoints. It may be worthwhile to introduce a v3 API at this point that serves rich text fields in the new JSON format.

## Open Questions

...
