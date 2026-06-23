# RFC 28: Tagging improvements

* RFC: 28
* Author: Karl Hobley
* Status: Draft
* Created: 2018-06-01
* Last Modified: 2018-06-01

## Abstract

At the NHS Wagtail sprint, we looked at the some issues their editors have when creating content for a large site in Wagtail. Some issues were raised around tagging and the search-by-tag UI in the image and document choosers.

They have thousands of images and they use the “tags” field to taxonomise them in various ways (such as illness, age group and gender). This RFC proposes some changes we can make to Wagtail which should improve the experience for users who tag content in this way.

## Tag management

Currently, it’s really easy for users to add new tags, they just type the name in the “tags" field and hit enter. Suggestions for names are given but it’s easy to ignore this.

This means that you can easily end up with duplicate tags which fragments the content making it harder to search using that tag. The only way to resolve this is to find all the incorrectly tagged items and manually assign the correct tag, but this does not remove the incorrect tag from suggestions which could cause it to be assigned to something else by mistake.

Also, it’s possible that a site would want to predefine tags to prevent inconsistency. Using age brackets as an example, one editor might tag an image “Ages 18-25” while another editor might tag an image with with “Ages 20-25” and yet another might use “Young adult”.

Here’s some ideas of how this can be improved:

**Make it (slightly) harder to add new tags**

Instead of allowing the user to just hit enter to add a new tag, add an “Add tag” button within the suggestions. This should trigger the user to consider using an existing tag rather than create a new one.

This is how Github allows adding “Labels” to an issue:

![](/media/028-tagging-improvements/Github_add_label.png)

**Add a “Tag management” interface**

Add a separate interface in Wagtail to allow admins/moderators to manage the available tags. This would allow defining tags before inserting content and renaming tags for consistency. This could also include merging duplicate tags as well.

This could be used in conjunction with free tagging or free tagging could be disabled entirely.

## Image/Document choosers

There’s some shortcomings in the image/document choosers which could be improved to make finding images with tags more efficient for the user.

Here’s a screenshot of the image chooser from the bakery demo (I had to make up some tags). Currently, the “Bread” tag is selected.

![](/media/028-tagging-improvements/Image_chooser_current.png)

There are some problems here:

1. It’s not possible to filter by a tag that isn’t in the top ten “Popular tags”
2. It’s not possible to filter by more than one tag at a time
3. The only way to deselect the tag is to close and reopen the chooser
4. There is no indication to the user that the “Bread” tag is actually selected

**Add “Tags” field to the chooser**

The obvious solution to this is to add a “Tags” field, like what we have on the model, underneath the “Collections” field. This provides a solution to all of the points above:

![](/media/028-tagging-improvements/Image_chooser_with_tags_field.png)

**Change the layout of the chooser form**

Adding a new tags field to the chooser increases the amount of vertical space used by the form so we should tweak the layout to reduce the amount of vertical space that’s used.

Since the labels are redundent, we can remove them which allows us to put two fields on the same row:

![](/media/028-tagging-improvements/Image_chooser_new_layout.png)
