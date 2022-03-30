Distributed Information
=======================

So far, RDF might seem unnecessarily complex. For all of the examples so
far, the information could be written in plain XML just the same. Taking
a single document in isolation, this is true. But RDF excels when every
document is thought of as part of the bigger picture, part of a huge
graph of knowledge spread throughout the Internet. In this "Semantic
Web," every RDF statement posted somewhere on the net contributes a
small piece to the greater body of knowledge.

But, not every piece of knowledge in the Semantic Web is automatically
interpretable. Applications have to be told what to do with particular
predicates, or else the RDF is just URI salad. In the next section, I
tackle this further.

Choosing the Right Predicates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As you've seen so far, one can use RDF to model any type of knowledge
without having to use any centrally approved notions. If no one has
coined a URI for something you want to describe, you can create your own
URI for it. This goes for not just subjects and objects but predicates
as well. In the examples above, I made up all of the URIs, and it's
still perfectly valid RDF.

The trouble is that if I make up all of my own URIs, my RDF document has
no meaning to anyone else unless I explain what each URI is intended to
denote or mean. Two RDF documents with no URIs in common have no
information that can be interrelated. But, two documents that have some
URIs in common are talking about some of the same things. Someone else
might want to publish information about Law & Order. By using the same
URI for the television show in the two documents, RDF tools will be able
to recognize that the two documents are describing the same thing.

Here is an example of using RDF to describe books. Let's say,
hypothetically, that the Library of Congress posted an RDF list of books
and Amazon.com did the same.

*RDF from the Library of Congress (hypothetical)*

.. raw:: html

   <pre>
   &lt;urn:isbn:0143034650&gt;  dc:title  "Free Culture : The Nature and Future of Creativity"
   &lt;urn:isbn:0613917472&gt;  dc:title  "Code and Other Laws of Cyberspace"
   &lt;urn:isbn:B00005U7WO&gt;  dc:title  "The Future of Ideas"
   </pre>

*RDF from Amazon.com (hypothetical)*

.. raw:: html

   <pre>
   &lt;urn:isbn:0143034650&gt;  amazon:price  "$15.00" .
   &lt;urn:isbn:0613917472&gt;  amazon:price  "$26.35" .
   &lt;urn:isbn:B00005U7WO&gt;  amazon:price  "$9.95" .
   </pre>

The URIs for the books (``urn:isbn:…``) are what tie the two files together.
An RDF application using these files would be able to report that "The
Future of Ideas" is "$9.95" at Amazon because both the title and the
price are related to a common resource, denoted by the URI
``urn:isbn:B00005U7WO``. If the files did not have the same URI for that
book, nothing would indicate that the titles went with particular
prices.

It's also important to use the same predicates others are using when the
predicates you want already exist. This allows existing applications to
make use of your information, without needing to be modified by
developers to recognize your own URIs. For instance, if you're
describing documents, you should use the existing `Dublin Core
(DC) <http://dublincore.org/documents/dces/>`__ title and description
predicates so RDF applications that already use those predicates will be
able to extract the titles and descriptions from your data. If you're
describing people, use existing `Friend of a Friend
(FOAF) <http://www.foaf-project.org/>`__ predicates so that FOAF-based
applications will be able to take advantage of the information.

I used ``dc:title`` in my books example above because the predicate already
existed to convey the information I wanted to put into the files: a
relation between a book and its title. No standard predicate exists for
giving the Amazon.com price of a book, so I made one up. (Remember
``amazon:price`` is an abbreviated form of a full URI, but I've left out the
declaration of ``amazon:`` for the sake of brevity.)

| Be careful of respecting the meanings of existing predicates, though.
| RDF applications expect the ``dc:title`` predicate to relate a document to
  its title. You wouldn't want to use it to relate a telephone to its
  phone number, for instance, because existing RDF applications that get
  a hold of your data will use it in ways that simply won't make sense.

Meshing Information: A Real-World Example
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| Most of the time in today's world you get information from one place.
| For instance, a single database might contain the information for an
  entire product line. There isn't much on the web in the way of
  distributed information because it's usually hard to put information
  together from multiple sources, each of which may have its own data
  format and conventions.

Here's a scenario where distributed information makes a lot of sense: a
database of products from multiple vendors and reviews of those products
from multiple reviewers. No one vendor is going to want to be
responsible for maintaining a central database for this project,
especially since it will contain information for competing products and
negative reviews. Likewise, no one reviewer may have the resources to
keep such a database up to date. How can this become a reality?

RDF is particularly suited for this project. Each vendor and reviewer
will publish a file in RDF on their own websites. The vendors will
choose URIs for their products, and the reviewers will use those URIs
when composing their reviews. Vendors don't need to agree on a common
naming scheme for products, and reviewers aren't tied to a
vendor-controlled data format. RDF allows the vendors and reviewers to
agree on what they need to agree on, without forcing anyone to use one
particular vocabulary.

*Vendor 1*

.. raw:: html

   <pre>
   vendor1:productX    dc:title    "Cool-O-Matic"
   vendor1:productX    retail:price    "$50.75"
   vendor1:productX    vendor1:partno  "TTK583"
   vendor1:productY    dc:title    "Fluffertron"
   vendor1:productY    retail:price    "$26.50"
   vendor1:productY    vendor1:partno  "AAL132"
   </pre>

*Vendor 2*

.. raw:: html

   <pre>
   vendor2:product1    dc:title    "Can Closer"
   vendor2:product1    retail:price    "$28.11"
   vendor2:product1    vendor2:warranty_code   "None."
   vendor2:product2    dc:title    "Dust Unbuster"
   vendor2:product2    retail:price    "$33.21"
   vendor2:product2    vendor2:warranty_code   "X12"
   </pre>

*Reviewer 1*

.. raw:: html

   <pre>
   vendor1:productX    dc:description  "This product is good buy!" .
   </pre>

*Reviewer 2*

.. raw:: html

   <pre>
   vendor2:product2  dc:description  "Who needs something to unbust dust? 
                                     A dust buster would be a better idea."
   vendor2:product2  review:rating   review:Excellent
   </pre>

It's an open question just how an application will retrieve these files,
but I'll put that aside. Once an application has these files, it has
enough information to relate products to reviews to prices, and even to
vendor-specific information like ``vendor1:partno`` and
``vendor2:warranty_code``. What you should take away from this example is
how unconstraining RDF is, while still allowing applications to
immediately be able to relate information together.

And, RDF applications don't need to know about the nature of the data in
these files to be able to make use of it. If an application already
knows what the ``dc:title`` and ``dc:description`` predicates are for, and
nothing else, then it is at least able to present the titles and reviews
of the four products. Note that the presence of predicates the
application doesn't understand, like ``review:rating``, doesn't impact the
application at all. It can simply ignore it without worrying that it has
misunderstood the rest of the data.

In addition, the vendors and reviewers did not have to agree on much to
make this happen. They had to agree to use RDF, but they didn't have to
agree on any specific data format or even on specific URIs. It helps
that they agreed on URIs for indicating titles and prices, although even
that wasn't strictly necessary. But, crucially, they didn't have to
enumerate everything any vendor would want to include about their
products. When a vendor needed something that wasn't already agreed on
(product numbers and warranty codes), they were able to create a new
predicate without disrupting any existing systems. Likewise, the
reviewers aren't tied to a vendor-controlled vocabulary. Reviewers were
free to add their own relations, such as a ratings, to their RDF files.

| Another way to look at this from the standpoint of interoperability.
| Vendor 1's format is entirely interoperable with anyone else's format,
  even though Vendor 1 didn't hash out a common format with anyone. When
  someone comes along and wants to be interoperable with Vendor 1's
  information, they don't need a new format, they just need to choose
  the right subjects, predicates, and objects.

Is this any better than XML? I'll take a look at that later on.
