# Web Publications Explainer


## Introduction

What is a web publication? The spec [says](https://w3c.github.io/wpub/#intro):

> A Web Publication is a discoverable and identifiable collection of resources. Information about the Web Publication is expressed in a machine-readable document called a manifest, which is what enables user agents to understand the bounds of the Web Publication and the connection between its resources.

Two things make web publications different from the “ordinary” web we know and love. First, a web publication may consist of multiple resources that form a logical whole. Moby-Dick might consist of 136 HTML files in a specified order, but it’s still a single work. So searching should search all 136 chapters.

More importantly, users have a set of expectations about how such content should be presented in order to make it easy to read and understand. Users need to personalize the presentation, using the font and font size that make it easiest for them to read. They want it to be easy to go to the next chapter without interrupting the reading experience by hunting for a link to click. They might need a high- or low-contrast version of the content. They want to read while offline.

Thus the goal of web publications is to make these features ("affordances") available to readers. We hope that someday browsers will provide these features, but in the meantime we have to use scripting and polyfills. That's OK, because we'll learn a lot from trying, and that will help the spec get better.

## Goals

 
- Provide a mechanism for defining a collection of web resources as a publication

 - The resources may be images (comics/manga), audio files (audiobooks), or even data files which would not be rendered.
  
- Provide a mechanism for ascribing descriptive metadata to a collection of web resources.

- Provide a mechanism for defining an ordering of the resources in a publication

- Describe the affordances needed for reading publications on the web. Note that publications may take many forms, from journals to educational texts to novels. 

 
## Non-goals

 - Issues of layout, such as pagination or displaying different resources side-by-side.
 
 - Extending the DOM to include collections of document elements 
 
 - DRM
 
## Basic design

A web publication must have an *entry page*, which the HTML document returned by the URL of the publication. This page must have either a link to the manifest (`<link rel="publication" href="manifest.json">`, or an [embedded](https://github.com/w3c/wpub/issues/327) manifest. 

The term “manifest” originally described a list of the passengers or cargo on a ship. For web publications, a manifest lists the constituents of the publication—all the HTML files, stylesheets, images, scripts, etc.—needed to create the whole. One list, called the `readingOrder`, describes the default sequence of primary resources, so that we know that `chapter-02.html` comes after `chapter-01.html`. Another list, called simply `resources`, lists all the other associated files. The union of `readingOrder` and `resources` defines the bounds of the publication. 

The manifest is also the natural location for metadata that applies to the whole publication, rather than just one of the constituents. The metadata vocabulary is based on schema.org; the entire manifest is serialized as JSON-LD, as it is a syntax of choice for schema.org.


## Example Manifest

Here's a simple example of a web publication manifest, for a tiny version of *Moby-Dick* with only a few HTML files:

```json
{
    "@context": ["https://schema.org", "https://www.w3.org/ns/wp-context"],
    "type": "Book",
    "url": "https://publisher.example.org/mobydick",
    "author": "Herman Melville",
    "dateModified": "2018-02-10T17:00:00Z",

    "readingOrder": [
        "html/title.html",
        "html/copyright.html",
        "html/introduction.html",
        "html/epigraph.html",
        "html/c001.html",
        "html/c002.html",
        "html/c003.html",
        "html/c004.html",
        "html/c005.html",
        "html/c006.html"
    ],

    "resources": [
        "css/mobydick.css",
        {
            "type": "PublicationLink",
            "rel": "https://www.w3.org/ns/wp#cover-page",
            "url": "images/cover.jpg",
            "encodingFormat": "image/jpeg"
        },{
            "type": "PublicationLink",
            "url": "html/toc.html",
            "rel": "contents"
        },{
            "type": "PublicationLink",
            "url": "fonts/STIXGeneral.otf",
            "encodingFormat": "font/otf"
        },{
            "type": "PublicationLink",
            "url": "fonts/STIXGeneralBol.otf",
            "encodingFormat": "font/otf"
        },{
            "type": "PublicationLink",
            "url": "fonts/STIXGeneralBolIta.otf",
            "encodingFormat": "font/otf"
        },{
            "type": "PublicationLink",
            "url": "fonts/STIXGeneralItalic.otf",
            "encodingFormat": "font/otf"
        }
    ]
}

```

Note how we used `rel=contents` to identify that a particular resource is an HTML table of contents.

## Modularity

The Group is working toward a WP model that is very slim, focusing almost exclusively on the metadata aspects of a Web Publication. All information about affordances and behaviors will move to the [WP Use Cases and Requirements Documents](https://w3c.github.io/dpub-pwp-ucr/). Detailed information about specific types of publications will go into stand-alone modules as business needs arise. The first module will be audiobooks (see the [separate explainer](./audio-explainer.md)). Future modules might include comics/manga/bandes dessinees and scholarly publishing. 


## Design choices


The design of web publications has caused perhaps the greatest amount of discussion within the group, and centers around three key areas:

1. how best to identify the "bounds" of the publication;
2. how best to define the ordering of the primary resources; and
3. how best to express metadata for the publication as a whole.

This section outlines the current approaches that have been adopted in each of these areas, and lists alternative practices that have been discussed and/or influenced the decision-making process. The currently-adopted solutions may change as discussions evolve and external feedback is obtained.


### 1. What is part of the publication?

The Web Publications Working Group has chosen to use an explicit list of resources in the manifest to identify the bounds. Anything that is not part of the `resources` or `readingOrder` manifest members is considered to be outside the web publication.

Precedent for this design pattern can be found in EPUB, which includes a comprehensive listing of publication resources in the package document `manifest` element.

The group has also considered the patterns described below, but currently believes an explicit list is the best choice in terms of simplicity and flexibility (e.g., incorporating resources from across the Web).

- Using a `scope`, as was done in the [Web Application Manifest spec](https://w3c.github.io/manifest/).

- Not explicitly define the boundaries, similar to how web sites function.

- Using the notion of *transclusion*, where the contents of other documents can be incorporated into a parent document based on hypertext references (e.g., through iframes, HTML imports, custom elements, etc.). For example:

```html
<html>
  <head>
    <title>Moby-Dick</title>
  </head>
  <body>
    <web-publication>
      <iframe src="c001.html">
      <iframe src="c002.html">
      <iframe src="c003.html">
    </web-publication>
  </body>
</html>

```


### 2. Sequence of primary resources

Web Publications use an explicit list in the manifest to define the reading sequence &#8212; via the `readingOrder` member.

This design choice also has precedent in EPUB, which defines the order of content documents in the package document `spine` element.

Other methods considered to achieve sequencing include:

- `rel=prev` and `rel=next` in HTML, but these do not have UI support from the browsers (with notably rare exceptions) and are not available for most non-HTML media types.

- identifying sequence via a list of links in a `nav` element (e.g., the table of contents), but not all resources may be listed in the table of contents

- the DOM order of transcluded documents


### 3. Metadata

Web Publication metadata is expressed in the manifest using the schema.org vocabulary, with some extensions. The metadata is serialized as JSON-LD, one of the three syntaxes promoted by schema.org and understood by search crawlers (the others being microdata and RDFa). JSON-LD has also the advantage of being easy to use for both client and server side processing.

The encapsulation of metadata is not new or unique to Web Publications, so precedent can be found in EPUB, Web App Manifest, and other standards. Where Web Publications differs is in putting priority on conformance to schema.org, which makes the metadata more Web-friendly for a wider variety of processors.

Embedding metadata directly in the header and/or content of HTML files has also been considered, but such a design presents greater complexity in terms of identifying and harvesting the information.


## The User Experience

Reading something that takes a day or a week rather than a few minutes influences what sort of user experience works for the reader. We have a formal [use cases and requirements document](https://w3c.github.io/dpub-pwp-ucr/) in addition to lots of actual experience from ebook reading systems. Browsers have implemented "reading modes" to address some of these requirements. 


#### Navigating

- Easily move through the entire publication in order. In most ebook reading systems, a single gesture moves between pages and between sections. 

- The table(s) of contents should always be available. It should be possible to quickly navigate anywhere in the publication. 

- Retaining state. If I leave a publication while reading, I should be able to easily return to where I left off. Extra credit for implementing this across devices. 

- I may have hundreds or thousands of publications. How can they be organized so I can find them? 


#### Personalization

- ability to change font, font size, background color

- night mode

#### Offline

- Web publications should function while offline (Service Workers)

#### Annotation

- it should be possible to bookmark, highlight, and annotate web publications

- it should be possible to craft a URL to point anywhere within a publication

#### Search

- it should be possible to search the entire publication

## Unanswered questions


### Architecture

Andrew Betts, then on the TAG, [commented](https://github.com/w3c/wpub/issues/32#issuecomment-362273649):

> It seems to us on the TAG that the Readium manifest format is very unlikely to be considered for support by implementors of general purpose web browsers. Therefore, the question seems to be: is the goal of this group to make a new packaging format for specialist book reading software and devices, or is it to obtain first class support for missing book-related features in the web platform as a whole? If the latter, then it is an order of magnitude more likely to be achieved by building atop existing platform features - notably Web App Manifest and service worker, than creating a separate but similar concept.

It is true that the Web Publication Manifest appears to be very similar to the Web Application Manifest &#8212; both are JSON files that are linked to from HTML, and both provide metadata about a composite resource. Several [arguments](https://github.com/w3c/wpub/wiki/Options-for-Processing-a-Manifest) [have](https://github.com/w3c/wpub/issues/32) been made against building on top of WAM, however:

1. WP use cases are orthogonal to those of WAM. Nothing is stopping a creator of a web publication from also using a web application manifest, if the publication author desires for the publication to be installable, etc. 

2. A primary objective of web publications is to make publication metadata available for SEO. We feel that schema.org metadata is the best way to do this, but this implies using JSON-LD, which is not compatible with the syntax of WAM. 

3. There were concerns about the extensibility of web application manifest, especially with regards to the processing of new members. 

4. Web publications are fundamentally different from web apps, as the goal is for the user agent to provide the user interface. 

As a result, the working group has spent much of its time on a high-level data model, and work on identifying low-level primitives has been deferred.


### Addressability

A publication needs a permanent URL, and it should be possible to craft a URL to point anywhere within a publication. 


### Packaging

The working group has not yet formally addressed the issue of packaging, although it is central to many publishing use cases. We are carefully monitoring the [web packaging](https://github.com/WICG/webpackage) effort. 

## Acknowledgments

The initial draft was written by Dave Cramer. Thanks to Garth Conboy, Ivan Herman, Deborah Kaplan, Tzviya Siegman, and Avneesh Singh for valuable feedback. 
