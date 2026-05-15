# RFC 19: 

* RFC: 19
* Author: Duco Dokter
* Status: Draft
* Created: 2018-03-14
* Last Modified: 2018-03-20

## Abstract

The current Wagtail API is limited in terms of resource coverage and
per-resource explicitness. Use cases for an extensive API range from a
full blown headless CMS to browsable objects for front-end selection
of, for example, pages, images and documents. Even though full
coverage may not be in the scope of development for now, the API needs
to be set up in a way that enables consumers to extract information in
a uniform way and that allows for future extension.


## Specification

The Wagtail API is implemented with the de facto REST implementation
of _django\_restframework_, or _DRF_. This framework however doesn't
provide the level of explicitness that is needed for a REST
implementation that is truly RESTful in terms of the original concepts
as defined by Roy Fielding. The fourth guiding constraint states,
among other things:

    ... hypermedia as the engine of application state.

or in other words: the state of the resource should be reflected in
it's representation. This phrase is usually shortened to
HATEOAS. Indeed, not all acronyms make things easier.

A typical example in the context of a CMS would be the representation
of a page. Let us assume that a page can either be public, or private,
and that a consumer with the proper authorisation can change this
state.  A representation of the page would then not only need to show
the state, but also the way to change it. HATEOAS does not dictate
*how* to achieve this. However, usually implementations use some kind
of meta information on a resource. So, this would be a resource representation
lacking in hypermedia:

    {
      'id': 666,
      'title': 'Designing great beers',
      'author': 'Bob Dobalina',
      'state': private'
    }

and this would be the HATEOAS version:

    {
      'id': 666,
      'title': 'Designing great beers',
      'author': 'Bob Dobalina',
      'state': private',
      '_links': {
        'self': 'http://be.er/api/pages/666/',
        'publish': 'http://be.er/api/pages/666/publish/'
      }
    }

and this would be the same page, for a consumer without proper
authorisation for publishing (but enough to see private pages...):

    {
      'id': 666,
      'title': 'Designing great beers',
      'author': 'Bob Dobalina',
      'state': private',
      '_links': {
        'self': 'http://be.er/api/pages/666/',
      }
    }

After publication, the page to the authorised consumer, would look like this:

    {
      'id': 666,
      'title': 'Designing great beers',
      'author': 'Bob Dobalina',
      'state': public',
      '_links': {
        'self': 'http://be.er/api/pages/666/',
        'unpublish': 'http://be.er/api/pages/666/unpublish/'
      }
    }

So: if the consumer can perform any kind of action other than the
standard CRUD methods for REST, it should be reflected in the
resource. This allows any consumer, be it human or robot, to navigate
through the API and to detect what actions may be performed on a given
resource. A page edit screen using the REST resource for example,
would show the 'publish' button only if the link is available.

This proposal consists of the following:

  1. implement a root view for the API, where all available resources
     are listed, so a consumer can start navigation from here;
  2. implement HATEOAS for all resources using the *_links* meta property;
  3. provide documentation for all views for human consumption on filtering
     or search options for the given resource.

This way, a consumer always only needs to know one URL, being the root
URL for the API. All further navigation is provided.


## Sometimes a working demo says more than a thousend words

... and usually is much less work to write. So you may want to check
out wagtail.campsix from Github, install it, andsee what I mean:

    https://github.com/ddokter/wagtail.campsix


## Open Questions
