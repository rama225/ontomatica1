What is RDF and what is it good for?
====================================

This is an introduction to RDF ("Resource Description Framework"), which
is the standard for encoding metadata and other knowledge on the
Semantic Web. In the Semantic Web, computer applications make use of
structured information spread in a distributed and decentralized way
throughout the current web. RDF is an abstract model, a way to break
down knowledge into discrete pieces, and while it is most popularly
known for its RDF/XML syntax, RDF can be stored in a variety of formats.
This article discusses the abstract RDF model, two concrete
serialization formats, how RDF is used and how it differs from plain
XML, higher-level RDF semantics, best practices for deployment, and
querying RDF data sources.

--------------

Translations

The first chapter (Really Quick Introduction) has been translated into:

-  `Russian <http://xmlhack.ru/texts/06/rdf-quickintro/rdf-quickintro.html>`__
   by A.Skrobov
-  `Spanish <http://www.seofreelance.es/que-es-rdf-introduccion-a-rdf/>`__
   by Ricard Menor

--------------

This document was originally written in October 2005. In July 2006 it
was revised and extended with material from my
`xml.com <http://www.xml.com/pub/a/2001/01/24/rdf.html>`__ article "What
is RDF". In January 2008 it was revised with more on N3 and RDF/XML and
extended with the new sections Linked Data for the Web and Querying
Semantic Web Databases

Really Quick Intro to RDF
-------------------------

This section is a really brief introduction to Resource Description
Framework (RDF). You might also be interested in…

-  My more in-depth guide, which you'll find after this quick part wraps
   up.
-  Two video introductions to `the Semantic
   Web <http://wiki.digitalbazaar.com/en/Semantic-web-intro>`__ and `RDF
   and RDFa <http://wiki.digitalbazaar.com/en/Rdfa-basics>`__ by Manu
   Sporny are very good.
-  Ian Davis's `RDF
   Tutorial <http://research.talis.com/2005/rdf-intro/>`__ slides are
   also very good.

RDF is a method for expressing knowledge in a decentralized world and is
the foundation of the Semantic Web, in which computer applications make
use of distributed, structured information spread throughout the Web.
Just to get it out of the way, RDF isn't strictly an XML format, it's
not just about metadata, it has little to do with RSS, and it's not as
complicated as you think.

The Big Picture
~~~~~~~~~~~~~~~

RDF is a general method to decompose any type of knowledge into small
pieces, with some rules about the semantics, or meaning, of those
pieces. The point is to have a method so simple that it can express any
fact, and yet so structured that computer applications can do useful
things with it. Here's some RDF:

.. raw:: html

   <pre class="code">
   @prefix : &lt;http://www.example.org/&gt; .
   :john    a           :Person .
   :john    :hasMother  :susan .
   :john    :hasFather  :richard .
   :richard :hasBrother :luke .
   </pre>

The meaning is obvious. We'll get to the details later.

If you know XML, here's a brief comparison. Like RDF, XML also is
designed to be simple and general-purpose. XML can be abstracted beyond
its written brackets-and-slashes notation to something more abstract, a
"DOM" for tree-structured data. Similarly, RDF isn't just about how you
write it. It's about representing network- or graph-structured
information. You *can* write RDF in XML, and many people do. Here's what
it might look like:

.. raw:: html

   <pre class="code">
   &lt;rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
       xmlns:ns="http://www.example.org/#"&gt;
     &lt;ns:Person rdf:about="http://www.example.org/#john"&gt;
       &lt;ns:hasMother rdf:resource="http://www.example.org/#susan" /&gt;
       &lt;ns:hasFather&gt;
         &lt;rdf:Description rdf:about="http://www.example.org/#richard"&gt;
           &lt;ns:hasBrother rdf:resource="http://www.example.org/#luke" /&gt;
         &lt;/rdf:Description&gt;
       &lt;/ns:hasFather&gt;
     &lt;/ns:Person&gt;
   &lt;/rdf:RDF&gt;
   </pre>

But you don't have to use XML. I don't. The first format above, called
N3, is just as good.

What really sets RDF apart from XML and other things is that RDF is
designed to represent knowledge in a distributed world. This means RDF
is particularly concerned with meaning. Everything at all mentioned in
RDF means something, whether a reference to something concrete in the
world, an abstract concept, or a fact. Standards built on RDF describe
logical inferences between facts and how to search for facts in a large
database of RDF knowledge.

What makes RDF suited for distributed knowledge is that RDF applications
can put together RDF files posted by different people around the
Internet and easily learn from them new things that no single document
asserted. It does this in two ways, first by linking documents together
by the common vocabularies they use, and second by allowing any document
to use any vocabulary. This flexibility is fairly unique to RDF.

Consider this second document of RDF:

.. raw:: html

   <pre class="code">
   @prefix : &lt;http://www.example.org/&gt; .
   :richard :hasSister :rebecca
   { ?a :hasFather ?b . ?b :hasSister ?c . } =&gt; { ?a :hasAunt ?c } .
   </pre>

This RDF document defines what it means to be an aunt, in terms of two
other relations. You could imagine an application putting this document
together with the first RDF document to determine that   ``:rebecca`` is
``:john``'s aunt. What makes this work is that the names of entities are
global. That is, when ``:john`` and ``:hasFather`` are used in one document,
applications can assume they have the same meaning in any other RDF
document with the same ``@prefix``.

So why use RDF? Here are use cases, as described by Richard Cyganiak on
the `W3C's Semantic Web mail
list <http://www.w3.org/2001/sw/interest/>`__:

-  You want to integrate data from different sources without custom
   programming.
-  You want to offer your data for re-use by other parties\* You want to
   decentralize data in a way that no single party "owns" all the data.
-  You want to do something fancy with large amounts of data (browse,
   query, match, input, extract, …), so you develop (or re-use) a
   generic tool that allows you to do this on top of the RDF data model
   (which has the advantage of not being tied to a proprietary data
   storage/representation technology, like a database dialect).

RDF Defined
~~~~~~~~~~~

RDF can be defined in three simple rules:

1. A fact is expressed as a triple of the form (Subject, Predicate,
   Object). It's like a little English sentence.
2. Subjects, predicates, and objects are names for entities, whether
   concrete or abstract, in the real world. Names are either 
   1) global and refer to the same entity in any RDF document in which they appear, or
   2) local, and the entity it refers to cannot be directly refered to
   outside of the RDF document.

3. Objects can also be text values, called literal values.

You've seen facts already. Each line below was a fact:

.. raw:: html

   <pre class="code">
   :john    a           :Person .
   :john    :hasMother  :susan .
     ...
   </pre>

| Names come in two types. Global names, which have the same meaning
  everywhere, are always Uniform Resource Identifiers (URIs). URIs can
  have the same syntax or format as website addresses, so you will see
  RDF files that contain URIs like
  ``http://www.w3.org/1999/02/22-rdf-syntax-ns#type``, where that URI is the
  global name for some entity. The fact that it looks like a web address
  is totally incidental. There may or may not be an actual website at
  that address, and it doesn't matter. There are other types of URIs
  besides http:-type URIs. URNs are a subtype of URI used for things
  like identifying books by their ISBN number, e.g.
| ``urn:isbn:0143034650``. TAGs are a general-purpose type of URI. They look
  like ``tag:govtrack.us,2005:congress/senators/frist``. URIs are used as
  global names because they provide a way to break down the space of all
  possible names into units that have obvious owners. URIs that start
  with ``http://www.rdfabout.net/`` are implicitly controlled by me.

This point is important and needs repeating: Whatever their form, URIs
you see in RDF documents are merely verbose names for entities, nothing
more. Forget that it has anything to do with the web.

Since URIs can be quite long, in various RDF notations they're usually
abbreviated using the concept of namespaces from XML. That's what the
colons are doing in ``:john``, ``:hasMother``, and the other entities in the
example. The colons indicate the name is an abbreviated form. In these
cases, the names were ``http://www.example.org/#john``,
``http://www.example.org/#hasMother``, etc.

When written out, URIs are generally enclosed in brackets to distinguish
them from namespaced-abbreviated names.

Literal values allow text to be included in RDF. This is used heavily
when RDF is used for metadata:

.. raw:: html

   <pre class="code">
   &lt;http://www.rdfabout.net/&gt; a :Website .
   &lt;http://www.rdfabout.net/&gt; dc:title "rdf:about" .
   &lt;http://www.rdfabout.net/&gt; dc:description "A website about
       Resource Description Framework." .
   </pre>

And that's basically RDF.

RDF As A Graph
~~~~~~~~~~~~~~

There are two complementary ways of looking at RDF information. The
first is as a set of statements, like above. Each statement represents a
fact. The second way is as a graph.

A graph is basically a network. Graphs consist of nodes interconnected
by edges. In the Internet, for instance, the nodes are the computers,
and the edges are the ethernet wires connecting them. In RDF, the nodes
are names (not actual entities) and the edges are statements. Here's an
example:

.. raw:: html

   <center>

.. image:: /_static/images/rdfasagraph.png

.. raw:: html

   </center>

Each arrow or edge is a RDF statement. The name at the start of the
arrow is the statement's subject, the name at the end of the arrow is
the statement's object, and the name that labels the arrow is the
predicate. RDF as a graph expresses exactly the same information as RDF
written out as triples, but the graph form makes it easier for us humans
to see structure in the data.

A Quick Example
~~~~~~~~~~~~~~~

So how is RDF useful? It's the technology for the job when you want to
mesh together distributed information.

Here's a scenario where distributed information makes a lot of sense: a
database of products from multiple vendors and reviews of those products
from multiple reviewers. No one vendor is going to want to be
responsible for maintaining a central database for this project,
especially since it will contain information for competing products and
negative reviews. Likewise, no one reviewer may have the resources to
keep such a database up to date.

RDF is particularly suited for this project. Each vendor and reviewer
will publish a file in RDF on their own websites. The vendors will
choose URIs for their products, and the reviewers will use those URIs
when composing their reviews. Vendors don't need to agree on a common
naming scheme for products, and reviewers aren't tied to a
vendor-controlled data format. RDF allows the vendors and reviewers to
agree on what they need to agree on, without forcing anyone to use one
particular vocabulary.

Here are the RDF files they publish:

.. raw:: html

   <pre class="code">
   **Vendor 1:**
   vendor1:productX    dc:title    "Cool-O-Matic" .
   vendor1:productX    retail:price    "$50.75" .
   vendor1:productX    vendor1:partno  "TTK583" .
   vendor1:productY    dc:title    "Fluffertron" .
   vendor1:productY    retail:price    "$26.50" .
   vendor1:productY    vendor1:partno  "AAL132" .

   **Vendor 2:**
   vendor2:product1    dc:title    "Can Closer" .
   vendor2:product2    dc:title    "Dust Unbuster" .

   **Reviewer 1:**
   vendor1:productX    dc:description  "This product is good buy!" .

   **Reviewer 2:**
   vendor2:product2  dc:description  "Who needs something to unbust dust? 
                                     A dust buster would be a better idea,
                                     and I wish they posted the price." .
   vendor2:product2  review:rating   review:Excellent .
   </pre>

It's an open question just how an application will retrieve these files,
but I'll put that aside. Once an application has these files, it has
enough information to relate products to reviews to prices, and even to
vendor-specific information like vendor1:partno. What you should take
away from this example is how unconstraining RDF is, while still
allowing applications to immediately be able to relate information
together.

The vendors and reviewers didn't have to agree on much to make this
happen. They had to agree to use RDF, but they didn't have to agree on
any specific data format or even on specific URIs. Crucially, they
didn't have to enumerate everything any vendor would want to include
about their products, and they can't lock out reviewers from posting
related information.

Another way to look at this from the standpoint of interoperability.
Vendor 1's format is entirely interoperable with anyone else's format,
even though Vendor 1 didn't hash out a common format with anyone. When
someone comes along and wants to be interoperable with Vendor 1's
information, they don't need a new format, they just need to choose the
right subjects, predicates, and objects.

Conclusion
~~~~~~~~~~

If you thought RDF was complicated, I hope you see now that it doesn't
have to be. RDF is easy to write, flexible, and unconstraining. It makes
it easy to model knowledge and to mesh distributed knowledge sources.

Why we need a new standard for the Semantic Web
-----------------------------------------------

On the `Semantic Web <http://www.w3.org/2001/sw/>`__, computers do the
browsing for us. The "SemWeb" enables computers to seek out knowledge
distributed throughout the Web, mesh it, and then take action based on
it. To use an analogy, the current Web is a decentralized platform for
distributed *presentations* while the SemWeb is a decentralized platform
for distributed *knowledge*. `RDF <http://www.w3.org/RDF/>`__ is the W3C
standard for encoding knowledge.

There of course is knowledge on the current Web, but it's off limits to
computers. Consider a `Wikipedia <http://www.wikipedia.org>`__ page,
which might convey a lot of information to the human reader, but to the
computer displaying the page all it sees is presentation markup. To the
extent that computers make sense of HTML, images, Flash, etc., it's
almost always for the purpose of creating a presentation for the
end-user. The real content, the knowledge the files are conveying to the
human, is opaque to the computer.

What is meant by "`semantic <http://en.wikipedia.org/wiki/Semantics>`__"
in the Semantic Web is not that computers are going to understand the
meaning of anything, but that the logical pieces of meaning can be
mechanically manipulated by a machine to useful ends.

So now imagine a new Web where the real content can be manipulated by
computers. For now, picture it as a web of databases. One "semantic"
website publishes a database about a product line, with products and
descriptions, while another publishes a database of product reviews. A
third site for a retailer publishes a database of products in stock.
What standards would make it easier to write an application to mesh
distributed databases together, so that a computer could use the three
data sources together to help an end-user make better purchasing
decisions?

There's nothing stopping anyone from writing a program now to do those
sorts of things, in just the same way that nothing stopped anyone from
exchanging data before we had XML. But standards facilitate building
applications, especially in a decentralized system. Here are some of the
things we would want a standard about *distributed knowledge* to
consider:

1. Files on the Semantic Web need to be able to express information
   flexibly. Life can't be neatly packed into tables, as in relational
   databases, or hierarchies, as in XML. The information about movies
   and TV shows contained in the graph below is really best expressed
   *as a graph*:

   .. image:: /_static/images/whatisrdf_1.gif

   *Knowledge as a Graph*

   Of course, we can't be drawing our way through the Semantic Web, so
   instead we will need a tabular notation for these graphs. Compare the
   table below to the figure above. Each row represents an arrow (an
   "edge") in the figure. The first column has the name of the "node" at
   the start of the edge. The second column has the label of the edge
   itself (the kind of edge). The third column has the name of the node at
   the end of the arrow.

   .. table::
      :align: center
      :width: 100%
      :class: align-center
      :widths: auto

      +-------------------------+-----------------------+-----------------------+
      | **Start Node**          | **Edge Label**        | **End Node**          |
      +=========================+=======================+=======================+
      | vincent_donofrio        | starred_in            | law_&_order_ci        |
      +-------------------------+-----------------------+-----------------------+
      | law_&_order_ci          | is_a                  | tv_show               |
      +-------------------------+-----------------------+-----------------------+
      | the_thirteenth_floor    | similar_plot_as       | the_matrix            |
      +-------------------------+-----------------------+-----------------------+
      | ...                                                                     |
      +-------------------------------------------------------------------------+

   Whether we represent the graph as a picture or in a table, we're talking
   about the same thing. Both describe what is abstractly called a graph.
   More on this later.

2. Files on the Semantic Web need to be able to relate to each other. A
   file about product prices posted by a vendor and a file with product
   reviews posted independently by a consumer need to have a way of
   indicating that they are talking about the same products. Just using
   product names isn't enough. Two products might exist in the world
   both called "The Super Duper 3000," and we want to eliminate
   ambiguity from the SemWeb so that computers can process the
   information with certainty. The SemWeb needs globally unique
   identifiers that can be assigned in a decentralized way.

3. We will use vocabularies for making assertions about things, but
   these vocabularies must be able to be mixed together. A vocabulary
   about TV shows developed by TV aficionados and a vocabulary about
   movies independently developed by movie connoisseurs must be able to
   be used together in the same file, to talk about the same things, for
   instance to assert that an actor has appeared in both TV shows and
   movies.

These are some of the requirements that RDF, Resource Description
Framework, provides a standard for, as we'll see in the next section.
Before getting too abstract, here are actual RDF examples of the
information from the graph above, first in the Notation 3 format, which
closely follows the tabular encoding of the underlying graph:

*Notation 3 Example*

.. raw:: html

   <pre>
   @prefix rdf: &lt;http://www.w3.org/1999/02/22-rdf-syntax-ns#&gt; .
   @prefix ex: &lt;http://www.example.org/&gt; .

   ex:vincent_donofrio ex:starred_in ex:law_and_order_ci .
   ex:law_and_order_ci rdf:type ex:tv_show .
   ex:the_thirteenth_floor ex:similar_plot_as ex:the_matrix .
   </pre>

And in the standard RDF/XML format, which may have a more intuitive feel
and is more explicit about hierarchical structure in the graph, but in
most cases it tends to obscure the underlying graph:

*RDF/XML Example*

.. raw:: html

   <pre>
   &lt;rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
       xmlns:ex="http://www.example.org/"&gt;

       &lt;rdf:Description rdf:about="http://www.example.org/vincent_donofrio"&gt;
           &lt;ex:starred_in&gt;
               &lt;ex:tv_show rdf:about="http://www.example.org/law_and_order_ci" /&gt;
           &lt;/ex:starred_in&gt;
       &lt;/rdf:Description&gt;

       &lt;rdf:Description rdf:about="http://www.example.org/the_thirteenth_floor"&gt;
           &lt;ex:similar_plot_as rdf:resource="http://www.example.org/the_matrix" /&gt;
       &lt;/rdf:Description&gt;

   &lt;/rdf:RDF&gt;
   </pre>

RDF was `originally <http://www.w3.org/TR/1999/REC-rdf-syntax-19990222/>`__
created in 1999 as a standard on top of XML for encoding metadata —
literally, data about data. Metadata is of course things like who
authored a Web page, what date a blog entry was published, etc.,
information that is in some sense *secondary* to some other content
already on the regular Web. Since then, and perhaps even after `the
updated RDF spec in 2004 <http://www.w3.org/TR/rdf-primer/>`__, the
scope of RDF has really evolved into something greater. The most exiting
uses of RDF aren't in encoding information about Web resources, but
information about and relations between things in the real world:
people, places, concepts, etc.

Introducing RDF
---------------

Unless you know Resource Description Framework (RDF) well, it's probably
best if you try to forget what you already know about it as you read the
rest of this section. RDF exists at the intersection of a few different
technologies, so it's easy to be lead into thinking that it is merely a
particular XML data format or a tool for blog feeds. Forget what you
know. Here is RDF from the beginning.

RDF is a general method to decompose knowledge into small pieces, with
some rules about the semantics, or meaning, of those pieces. The point
is to have a method so simple that it can express any fact, and yet so
structured that computer applications can do useful things with
knowledge expressed in RDF. I say "method" in particular, rather than
format, because one can write down those pieces in any number of ways
and still preserve the original information and structure, just like how
one can express the same meaning in different human languages or
implement the same data structure in multiple ways.

In some ways, RDF can be compared to XML. XML also is designed to be
simple and applicable to any type of data. XML is also more than a file
format. It is a foundation for dealing with hierarchical, self-contained
documents, whether they be stored on disk in the usual
brackets-and-slashes format, or held in memory and accessed through a
DOM API.

What sets RDF apart from XML is that RDF is designed to represent
*knowledge* in a *distributed* world. That RDF is designed for
knowledge, and not data, means RDF is particularly concerned with
meaning. Everything at all mentioned in RDF means something. It may be a
reference to something in the world, like a person or movie, or it may
be an abstract concept, like the state of being friends with someone
else. And by putting three such entities together, the RDF standard says
how to arrive at a fact. The meaning of the triple "(John, Bob, the
state of being friends)" might be that John and Bob are friends. By
putting a lot of facts together, one arrives at some form of knowledge.
Standards built on top of RDF, including RDFS and OWL, add to RDF
semantics for drawing logical inferences from data.

For comparison, XML itself is not very much concerned with meaning. XML
nodes don't need to be associated with particular concepts, and the XML
standard doesn't indicate how to derive a fact from a document. For
instance, if you were presented with a few XML documents whose root
nodes were in a foreign language you don't understand, you couldn't do
anything useful with the documents but display them. RDF documents with
nodes you can't understand could still actually be usefully processed
because RDF specifies some basic level of meaning. Now, this isn't to
say that you couldn't develop your own standard on top of XML that says
how to derive the set of facts in an XML document, but you'll find
you've probably just reinvented something like RDF.

The second key aspect of RDF is that it works well for distributed
information. That is, RDF applications can put together RDF files posted
by different people around the Internet and easily learn from them new
things that no single document asserted. It does this in two ways, first
by linking documents together by the common vocabularies they use, and
second by allowing any document to use any vocabulary. This allows
enormous flexibility in expressing facts about a wide range of things,
drawing on information from a wide range of sources.

   For the official documentation on RDF, start with the `RDF
   Primer <http://www.w3.org/TR/rdf-primer/>`__.

Triples of knowledge
--------------------

RDF provides a general, flexible method to decompose any knowledge into
small pieces, called triples, with some rules about the semantics
(meaning) of those pieces.

The foundation is breaking knowledge down into basically what's called a
labeled, directed graph, if you know the terminology.

Each edge in the graph represents a fact, or a relation between two
things. The edge in the figure above from the node ``vincent_donofrio``
labeled ``starred_in`` to the node ``the_thirteenth_floor`` represents the fact
that actor Vincent D'Onofrio starred in the movie "The Thirteenth
Floor." A fact represented this way has three parts: a subject, a
predicate (i.e. verb), and an object. The subject is what's at the start
of the edge, the predicate is the type of edge (its label), and the
object is what's at the end of the edge.

The six documents composing `the RDF
specification <http://www.w3.org/TR/rdf-primer/>`__ tell us two things.
First, it outlines `the abstract
model <http://www.w3.org/TR/2004/REC-rdf-concepts-20040210/>`__,
i.e. how to use triples to represent knowledge about the world. Second,
it describes `how to encode those triples in
XML <http://www.w3.org/TR/rdf-syntax-grammar/>`__. We'll take each
subject in turn.

The abstract RDF model: Statements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

RDF is nothing more than a general method to decompose information into
pieces. The emphasis is on general here because the same method can be
used for any type of information. And the method is this: **Express
information as a list of statements in the form SUBJECT PREDICATE
OBJECT.** The *subject* and *object* are names for two things in the
world, and the *predicate* is the name of a relation between the two.
You can think of predicates as verbs.

Here's how I would break down information about my apartment into RDF
statements:

.. table::
   :align: center
   :width: 100%
   :class: align-center
   :widths: auto

   +-------------------------+-----------------------+-----------------------+
   | **SUBJECT**             | **PREDICATE**         | **OBJECT**            |
   +=========================+=======================+=======================+
   | I                       | own                   | my_apartment          |
   +-------------------------+-----------------------+-----------------------+
   | my_apartment            | has                   | my_computer           |
   +-------------------------+-----------------------+-----------------------+
   | my_apartment            | has                   | my_bed                |
   +-------------------------+-----------------------+-----------------------+
   | my_apartment            | is_in                 | Philadelphia          |
   +-------------------------+-----------------------+-----------------------+

These four lines express four facts. Each line is called a **statement**
or **triple**.

The subjects, predicates, and objects in RDF are always simple names for
things: concrete things, like *my_apartment*, or abstract concepts, like
*has*. These names don't have internal structure or significance of
their own. They're like proper names or variables. It doesn't matter
what name you choose for anything, as long as you use it consistently
throughout.

Names in RDF statements are said to **refer to** or **denote** things in
the world. The things that names denote are called **resources** (dating
back to RDF's use for metadata for web resources), **nodes** (from graph
terminology), or **entities**. These terms are generally all synonymous.
For instance, the name *my_apartment* denotes my actual apartment, which
is an entity in the real world. The distinction between names and the
entities they denote is minute but important because two names can be
used to refer to the same entity.

Predicates are always relations between two things. *Own* is a relation
between an owner and an ‘ownee'; *has* is a relation between the
container and the thing contained; *is_in* is the inverse relation,
between the contained and the container. In RDF, the order of the
subject and object is very important.

The next aspect of RDF almost goes without saying, but I want to put
everything down in print: **If someone refers to something as X in one
place and X is used in another place, the two X's refer to the same
entity.** When I wrote *my_apartment* in the first line, it's the same
apartment that I meant when I wrote it in the other three lines.

The rules so far already get us a lot farther than you might realize.
Given this table of statements, I can write a simple program that can
answer questions like "who own my_apartment" and "my_apartment has
what." The question itself is in the form of an RDF statement, except
the program will consider wh-words like who and what to be wild-cards. A
simple question-answering program can compare the question to each row
in the table. Each matching row is an answer. Here's the pseudocode:

*Pseudocode for Question-Answering*

.. raw:: html

   <pre>
   question = (my_apartment, has, what)
   knowledge = (
           (I, own, my_apartment),
           (my_apartment, has, my_computer),
           (my_apartment, has, my_bed),
           (my_apartment, is_in, Philadelphia)
       )
   for each statement in knowledge {
       if ((statement.subject == question.subject
               or question.subject == what) {
             and (statement.predicate == question.predicate
               or question.predicate == what)
             and (statement.object == question.object
               or question.object == what))
           call FoundAnswer(statement)
       }
   }

   Output:
       Answer: my_apartment has my_computer
       Answer: my_apartment has my_bed
   </pre>

The computer doesn't need to know what *has* actually means in English
for this to be useful. That is, it's left up to the application writer
to choose appropriate names for things (e.g. my_apartment) and to use
the right predicates (own, has). RDF tools are ignorant of what these
names mean, but they can still usefully process the information. (I'll
get to more useful things later.)

URIs to Name Resources
~~~~~~~~~~~~~~~~~~~~~~

RDF is meant to be published on the Internet, and so the names I used
above have a problem. I shouldn't name something my_apartment because
someone else might use the name my_apartment for their apartment too.
Following from the last fact about RDF, RDF tools would think the two
instances of my_apartment referred to the same thing in the real world,
whereas in fact they were intended to refer to two different apartments.
The last aspect of RDF is that names must be global, in the sense that
you must not choose a name that someone else might conceivably also use
to refer to something different. Formally, **names for subjects,
predicates, and objects must be Uniform Resource Identifiers (URIs)**.
(Technically, names can be Internationalized Resource Identifiers (IRIs)
but the distinction is not important.)

URIs can have the same syntax or format as website addresses, so you
will see RDF files that contain URIs like
http://www.w3.org/1999/02/22-rdf-syntax-ns#type, where that URI is the
global name for some entity. This happens to be the URI for the concept
of "is a" (if you recognize it).

Now, in the SemWeb world, URIs are treated in a somewhat inconsistent
way, so bear with me here. On the one hand, URIs are supposed to be
opaque. The fact that a URI looks like a web address is totally
incidental. There may or may not be an actual website at that address,
and it doesn't matter. There are other types of URIs besides ``http:``-type
URIs. URNs are a subtype of URI used for things like identifying books
by their ISBN number, e.g. ``urn:isbn:0143034650``.
`TAGs <http://www.taguri.org/07/draft-kindberg-tag-uri-07.html>`__ are a
general-purpose type of URI. They look like
``tag:govtrack.us,2005:congress/senators/frist``. Not all URIs name web
pages. That's the difference between a URI and a URL. URLs are just
those URIs that name things on the web that can be retrieved, aka
"**dereferenced**".

So the convention goes: whatever their form, URIs you see in RDF
documents are merely verbose names for entities, nothing more. Well, at
least, that's how people felt about URIs till around 2007.

Starting in recent years there actually *has* been an expectation that
if you create an ``http:`` URI — or, any dereferencable URI (a URL) — that
you actually put something at that address so that RDF clients can
access that page and get some information. Here's the bottom line: As
for what a URI *means* in a document, what the URI is simply doesn't
matter, but when you use dereferencable URIs, there may be an
expectation that you put something on the web at that address. We will
return to this in the section about Linked Data.

| URIs are used as global names because they provide a way to break down
  the space of all possible names into units that have obvious owners.
| URIs that start with http://www.govtrack.us/ are implicitly controlled
  by me, or whoever is running the website at that address. By
  convention, if there's an obvious owner for a URI, no one but that
  owner will "mint" a new resource with that URI. This prevents name
  clashes.
| If you create a URI in the space of URIs that you control, you can
  rest assured no one will use the same URI to denote something else.
  (Of course, someone might use your URIs in a way that you would not
  appreciate, but this is a subject for another article.)

| Since URIs can be quite long, in various RDF notations they're usually
  abbreviated using the concept of namespaces from XML. As in XML, a
  namespace is generally declared at the top of an RDF document and then
  used in abbreviated form later on. Let's say I've declared the
  abbreviation ``taubz`` for the URI http://razor.occams.info/index.html.
| In many RDF notations, I can then abbreviate URIs like
  http://razor.occams.info/index.html#my_apartment by replacing the
  namespace URI exactly as it is given in the declaration with the
  abbreviation and a colon, in this case simply as ``taubz:my_apartment``.
  The precise rules for namespacing depend on the RDF serialization
  syntax being used.

Importantly, namespaces have no significant status in RDF. They are
merely a tool to abbreviate long URIs.

I might re-write the table about my apartment as it is below, replacing
the simple names I first used above with abritrary URIs:

*RDF about My Apartment*

.. raw:: html

   <pre>
   Let taubz: abbreviate http://razor.occams.info/index.html#

   taubz:me            http://example.org/own    taubz:my_apartment
   taubz:my_apartment  http://example.org/has    taubz:my_computer
   taubz:my_apartment  http://example.org/has    taubz:my_bed
   taubz:my_apartment  http://example.org/is_in  http://example.org/Philadelphia
   </pre>

The table above is just an informal table representing the graph of
information that exists at an abstract level, which could just as well
be described by the figure below. We will talk more about standard ways
of actually *writing out* RDF later on.

.. image:: /_static/images/rdfasagraph.png

*RDF as a Graph*

Wrapping It Up So Far
~~~~~~~~~~~~~~~~~~~~~

And that's RDF. Everything else in the Semantic Web builds on those
three rules, repeated here to hammer home the simplicity of the system:

1. A fact is expressed as a triple of the form (Subject, Predicate,
   Object).
2. Subjects, predicates, and objects are given as names for entities,
   whether concrete or abstract, in the real world.
3. Names are in the format of URIs, which are opaque and global.

These concepts form most of the abstract RDF model for encoding
knowledge. It's analogous to the common API that most XML libraries
provide. If it weren't for us curious humans always peeking into files,
the actual format of XML wouldn't matter so much as long as we had our
``appendChild``, ``setAttribute``, etc. Of course, we do need a common file
format for exchanging data, and in fact there are two for RDF, which we
look at later.

Blank Nodes and Literal Values
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There is actually a bit more to RDF than the three rules above. So far
I've described three types of things in RDF: resources (things or
concepts) that exist in the real world, global names for resources
(i.e. URIs), and RDF statements (triples, or rows in a table). There are
two more things.

Literals
^^^^^^^^

The first new thing is the **literal value**. Literal values are raw
text that can be used instead of objects in RDF triples. Unlike names
(i.e. URIs) which are stand-ins for things in the real world, literal
values are just raw text data inserted into the graph. Literal values
could be used to relate people to their names, books to their ISBN
numbers, etc.:

*Some Uses of Literals*

.. raw:: html

   <pre>
   taubz:me          foaf:name  "Joshua Ian Tauberer"
   book:HarryPotter  dc:title   "Harry Potter"
   book:HarryPotter  ex:price   "$18.75"
   </pre>

Blank/Anonymous Nodes
^^^^^^^^^^^^^^^^^^^^^

Then there are **anonymous nodes**, **blank nodes**, or **bnodes**.
These terms are all synonymous. The words anonymous or blank are meant
to indicate that these are nodes in a graph without a name, either
because the author of the document doesn't know or doesn't want to or
need to provide a name. In a sense, this is like saying "John is friends
with *someone*, but I'm not telling who." When we say these nodes are
nameless, keep in mind two things. First, the real-world thing that the
node denotes is not inherently nameless. John's friend, in the example,
has a name, after all. Second, when we say nameless here, we are
refering to the concept of naming things with URIs. Actual blank nodes
in documents may be given "local" identifiers so that they may be
referred to multiple times within a document. It is only that these
local identifiers are explicitly not global, and have no meaning outside
of the document in which they occur.

If you're familiar with formal semantics, blank nodes can often be
thought of as existentially bound variables.

Here's one way literal values and anonymous nodes are used. One literal
value in the example is ``"Joshua Tauberer"``, and the anonymous or blank
node is ``_:anon123``.

*Blank Nodes and Literal Values*

.. raw:: html

   <pre>
   taubz:me               foaf:name    "Joshua Tauberer"
   taubz:me               ex:has_read  &lt;urn:isbn:0143034650&gt;
   &lt;urn:isbn:0143034650&gt;  dc:title     "Free Culture : The Nature and Future of Creativity"
   &lt;urn:isbn:0143034650&gt;  dc:author    _:anon123
   _:anon123              foaf:name    "Lawrence Lessig"
   </pre>

To distinguish between URIs, namespaced names (abbreviated URIs),
anonymous nodes, and literal values, I used the following common
convention:

-  Full URIs are enclosed in angle brackets.
-  Namespaced names are written plainly, but their colons give them
   away.
-  Anonymous nodes are written like namespaced names, but in the
   reserved "_" namespace with an arbitrary local name after the colon.
-  Literal values are enclosed in quotation marks.

You should take a moment to try to visualize what graph is described by
the table. Picture arrows between nodes.

There is one blank node in this example, ``:anon123``. *What we know about
this resource is that it is the author of* ``<urn:isbn:0143034650>`` *and it
has the name Lawrence Lessig. Because no global name is used for this
resource, we can't really be sure who we're talking about here. And, if
we wanted to say more about whatever is denoted by* ``:anon123``, we would
have to do it in this very RDF document because we would have no way to
refer to this particular Lawrence Lessig outside of the document.

More on Literals: Language Tags and Datatypes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Literal values can be optionally adorned with one of two pieces of
metadata. The first is a *language tag*, to specify what language the
raw text is written in. The language tag should be viewed as a vestige
of how RDF was used in the early days. Today it is an ugly hack. You may
see " "chat"@en ", the literal value "chat" with an English language
tag, or " "chat"@fr ", the same with the French language tag.

Alternatively, a literal value can be tagged with a URI indicating a
*datatype*. The datatype indicates how to interpret the raw text, such
as as a number, a URI, a date or time, etc. Datatypes can be any URI,
although the datatypes defined in `XML
Schema <http://www.w3.org/TR/xmlschema-2/>`__ are used by convention.
The notation for datatypes is often the literal value in quotes followed
by two carets, followed by the datatype URI (possibly abbreviated):

*Datatypes*

.. raw:: html

   <pre>
   "1234"                This is an untyped literal value. No datatype.
   "1234"^^xsd:integer   This is a typed literal value using a namespace.
   "1234"^^&lt;http://www.w3.org/2001/XMLSchema#integer&gt;   The same with the full datatype URI.
   </pre>

Datatypes are a bit tricky. Let's think of the datatype for
floating-point numbers. At an abstract level, the floating-point numbers
themselves are different from the *text* we use to represent them on
paper. For instance, the text "5.1" represents the number 5.1, but so
does "5.1000" and "05.10". Here there are multiple textual
representations — what are called lexical representations — for the same
*value*. A datatype tells us how to map lexical representations to
values, and vice versa.

The semantics of RDF takes language tags and datatypes into account.
This means two things. First, a literal value without either a language
tag or datatype is different from a literal with a language tag is
different from a literal with a datatype. These four statements say four
different things and none can be inferred from the others:

*Literal Semantics*

.. raw:: html

   <pre>
   #john foaf:name "John Jones"              John's name is a lanaguage-less,
                                             datatype-less raw text value.
   #john foaf:name "John Jones"@en           John's name, in English, is John Jones.
   #john foaf:name "Jacque Jones"@fr         John's name, in French, is Jacque Jones.
   #john foaf:name "John Jones"^^xsd:string  John's name is a _string_.
   </pre>

So, an untyped literal with or without a language tag is *not the same*
as a typed literal. The second part of the semantics of literals is that
two typed literals that appear different *may be the same* if their
datatype maps their lexical representations to the same value. The
following statements *are* equivalent (at least for an RDF application
that has been given the semantics of the XSD datatypes):

*Datatype Semantics*

.. raw:: html

   <pre>
   #john ex:age "10"^^xsd:float
   #john ex:age "10.000"^^xsd:float
   </pre>

These mean John's age is 10. That is, the textual representation of the
number is besides the point and is not part of the meaning encoded by
the triples. Note that if the float datatype were not specified, the
triples would not be inherently equivalent, and the textual
representation of the 10 would be maintained as part of the information
content.

More on Blank Nodes: Some Caveats
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Unlike the rule for URIs stating that they are global, local identifiers
used to name blank nodes are explicitly *not* global. A local bnode
identifier used in two separate documents can refer to two things.
Still, however, the identifier itself is arbitrary, and the actual
identifier used in any particular case is not a part of the information
content of the document.

Anonymous nodes are often used to avoid having to assign people URIs, as
in the example above. They're also often used in representing more
complex relations:

*Blank Nodes for Complex Relations*

.. raw:: html

   <pre>
   taubz:me   ex:hasName     _:anon234
   _:anon234  ex:firstName   "Joshua"
   _:anon234  ex:middleName  "Ian"
   _:anon234  ex:lastName    "Tauberer"
   </pre>

Here the anonymous node was used as an intermediate step in the relation
between me and the parts of my name. The node represents my name in a
structured way, rather than using a single opaque literal value "Joshua
Ian Tauberer". RDF only allows binary relations, so it's necessary to
express many-way relations using intermediate nodes, and these nodes are
often anonymous.
