
Linked Data for the Web
=======================

Earlier I left two points to be discussed later. The first was a remark
that when using ``http:``-type URIs, there are expectations that something
actually exists at that web URL. The second was the open question about
how Semantic Web clients are supposed to *find* RDF data on the web. A
new Semantic Web community movement under the name Linked Data seeks to
provide some answers to these questions.

The notion of Linked Data is to bring the concept and benefits of
hyperlinking between HTML documents on the World Wide Web to RDF
documents on the Semantic Web. The core principle is that ``http:``-type
URIs *should* be used for RDF resources, so that RDF documents can exist
at those locations describing the resources. When those documents
mention other resources, if they have ``http:``-type URIs then SemWeb
clients can jump from document to document finding more information as
it goes.

To take an example: I have minted the URI
http://www.rdfabout.com/rdf/usgov/geo/us/ny to represent the state of
New York in the United States. If you visit `that
URL <http://www.rdfabout.com/rdf/usgov/geo/us/ny>`__ you will get back
an RDF document describing New York. And it refers to some other
resources, which happen to have ``http:``-type URIs that you could retrieve
to get documents describing the other resources. Don't confuse the
documents you get back with the resources named by the URIs themselves.
There's no guarantee that the document you get back will even mention
the resource named by that address (though that would certainly defeat
the purpose).

Terminology
~~~~~~~~~~~

I chose to use a ``http:``-type URI so that it is "**dereferencable**".
Dereferencable is a term in the World Wide Web, not the Semantic Web,
which means a URI that is a URL, or in other words a URI with the ``http:``
scheme (among others) that specifies how to fetch a document at that
address. The ``tag:`` and ``urn:`` schemes do not specify how to find documents
with those types of URLs, so such URIs are not dereferencable. You can't
put them in your browser and get back a document. URIs that you can put
in your browser are dereferencable.

The distinction above between document — what you can get back from a
browser — and resource — something named by a URI — highlights some
common terminology people use. An "**information resource**" is
something that can be transmitted electronically. Documents, such as web
pages and RDF/XML documents, images, and binary files are all
information resources. "**Non-information resources**" are those
resources that can be named by a URI but which cannot be transmitted
electronically. Human beings, abstract concepts, etc. are
non-information resources. As we've seen, both information and
non-information resources can be named with URIs. However, browsers can
only display information resources. They can display **representations**
of non-information resources (such as pictures of people), but they are
(by definition) incapable of displaying a non-information resource
itself.

Linked Data Under the Hood
~~~~~~~~~~~~~~~~~~~~~~~~~~

Using HTTP GET
^^^^^^^^^^^^^^

According to web architecture standards, the HTTP 200 OK response to
requests is to be used for URIs denoting information resources only. So
when visiting the URI above for New York, a non-information resource,
the web server at the other end first sends back a HTTP 303 See Other
response, i.e. a redirect. This indicates that the URI is not something
the web server can provide directly, because it cannot transmit a
non-information resource over the wire. It sends back instead a URL
directing the user agent to an information resource, with the
implication that information about the original URI can be found in the
document at that URL.

If you use Linux, you can observe this with the ``curl`` or ``wget``
command-line tools. Running ``curl`` as shown below prints out the redirect
browsers get when going to the URI for New York.

*Using curl to follow the linked data*

.. raw:: html

   <pre>
   $ curl http://www.rdfabout.com/rdf/usgov/geo/us/ny
   ...
   &lt;p&gt;The answer to your request is located &lt;a href="http://rdfabout.com/sparql?query=DESCRIBE+%3Chttp://www.rdfabout.com/rdf/usgov/geo/us/ny%3E"&gt;here&lt;/a&gt;.&lt;/p&gt;
   ...
   </pre>

If you look carefully, you'll see that in this case the redirect happens
to take you to what looks like a dynamically-generated URL. More on this
URL later. Often, however, you will be redirected to a static RDF/XML
document (with a .rdf extension, for instance).

Use the ``-L`` option to follow the redirect:

*Using curl to follow the linked data*

.. raw:: html

   <pre>
   $ curl -L http://www.rdfabout.com/rdf/usgov/geo/us/ny

   &lt;rdf:RDF xmlns:rdf="http://www.w...
       &lt;usgovt:State rdf:about="http://www.rdfabout.com/rdf/usgov/geo/us/ny"&gt;
           &lt;ns:title&gt;New York&lt;/ns:title&gt;
           &lt;terms:isPartOf rdf:resource="http://www.rdfabout.com/rdf/usgov/geo/us" /&gt;
           &lt;wgspos:lat rdf:datatype="http://www.w3.org/2001/XMLSchema#double"&gt;42.155127&lt;/wgspos:lat&gt;
           ...
   </pre>

After following the redirect, an RDF/XML document that describes New
York is returned.

You may also want to try ``wget`` with the ``-S`` option to view the HTTP
response headers:

*Using wget to follow the linked data*

.. raw:: html

   <pre>
   $ wget -S -O /dev/null http://www.rdfabout.com/rdf/usgov/geo/us/ny
   </pre>

Content Negotiation and Link tags
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The HTTP protocol allows for different response documents depending on
some aspects of the request. In particular, the requesting HTTP client
can specify in the HTTP ``Accept`` header the MIME type of what it wants,
say an HTML (``text/html``) page, as is usually the case, or something else.
This allows the server to overload its response, and this is called
content negotiation.

A web-browsing client accessing an RDF URI might want an HTML
representation of a resource to display to the user (some textual
description of what the URI represents), whereas an RDF client may want
an RDF/XML description of the resource. In this case, the client
specifies ``Accept: application/rdf+xml``. We think of the HTML page and the
RDF/XML document as two information-resource representations of the same
non-information resource (i.e. what the URI denotes).

HTTP GET's on RDF URIs, as just discussed, is one way for RDF clients to
find triples about a particular resource. And, content negotiation
allows there to be both an HTML page and an RDF document at the same
URL. Another way to associate a URI with an HTML page to an RDF document
is to use the HTML ``link`` tag. This tag will help RDF clients discover
related RDF information from a page the user may already be browsing.

Place ``link`` tags in the ``head`` section of HTML pages:

*Link Tags*

.. raw:: html

   <pre>
   &lt;link rel="alternate" type="application/rdf+xml" href="moreinfo.rdf" /&gt;
   </pre>

In this example, the file ``moreinfo.rdf`` is expected to be an RDF/XML
document, hopefully either describing the URI of the page on which the
``link`` tag is found, or describing a resource of primary importance on the
HTML page.

   For more information on Linked Data and some best practices tips, see
   `How to Publish Linked Data on the
   Web <http://sites.wiwiss.fu-berlin.de/suhl/bizer/pub/LinkedDataTutorial>`__.
