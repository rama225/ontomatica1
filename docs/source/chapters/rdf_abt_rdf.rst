
RDF about RDF
=============

So far I've shown how RDF can be used to describe the relationships
between entities in the world. RDF can be used at a higher level, too,
to describe RDF predicates and classes of resources. Ontologies,
schemas, and vocabularies, which all mean roughly the same thing, are
RDF information about… other RDF information.

| RDF ontologies play a vaguely similar role as XML Document Type
  Definitions and XML Schema. But they are as different as they are the
  same. DTDs and XML Schema specify what constitutes a valid document.
| They don't indicate how a document should be interpreted, and they
  only restrict the set of elements that can be used in any given file.
  RDF ontologies, which are themselves written in RDF, provide relations
  between higher-level things, entirely for the purpose of indicating to
  applications how some information should be interpreted. RDF
  ontologies also don't restrict at all which predicates are valid
  where.
| Any statement is valid anywhere, as before.

`RDF Schema <http://www.w3.org/TR/rdf-schema/>`__ (RDFS) and `Web
Ontology Language <http://www.w3.org/TR/owl-features/>`__ (OWL) define a
few *classes* and predicates that are, merely by convention, used to
provide higher-level descriptions of data.

   The classes and predicates below are some of the standard tools that
   are available to you when you write ontologies. For some examples of
   complete ontologies, including the standard RDF, RDFS, and OWL
   ontologies, see `SchemaWeb <http://www.schemaweb.info/>`__.

RDF Schema (RDFS)
~~~~~~~~~~~~~~~~~

| RDF Schema (RDFS) introduces the notion of a class. A class is a type
  of thing. For instance, you and I are members of the class Person.
| Computers are members of the class Machine. Person and Machine are
  classes. That is to say, they are themselves members of the type
  Class. The first higher-level predicate is the ``rdf:type`` predicate.
  (``rdf`` is the usual namespace abbreviation for
  http://www.w3.org/1999/02/22-rdf-syntax-ns.) It relates an entity to
  another entity that denotes the class of the entity. The purpose of
  this predicate is to indicate what kind of thing a resource is. But,
  as with anything else in RDF, the choice of class is either by
  convention or arbitrary.

To add class information into the vendor files from a few sections ago,
a vendor would simply add this:

*Adding Type Information*

.. raw:: html

   <pre>
   vendor1:productX    rdf:type    general:Product
   </pre>

As with choosing predicates, it's helpful to choose URIs for classes
that are used by others. Agreement among different parties for classes,
and for the other things in this section, is very important.

One interesting class is ``rdf:Property``. Any entity used as a predicate is
an ``rdf:Property``. Thus, from the examples above, we can conclude:

*The rdf:Property class*

.. raw:: html

   <pre>
   &lt;http://example.org/own&gt; rdf:type  rdf:Property
   dc:subject               rdf:type  rdf:Property
   amazon:price             rdf:type  rdf:Property
   </pre>

To be explicit, one might include these statements in an RDF ontology
describing the predicates used in the data.

Other RDFS predicates are used to provide even more information about
predicates. The ``rdfs:domain`` and ``rdfs:range`` predicates relate a predicate
to the class of resources that can serve as the subject or object of the
predicate, respectively. Here's an example:

*Domain and Range*

.. raw:: html

   <pre>
   vendor2:warranty_code   rdfs:domain general:Product
   vendor2:warranty_code   rdfs:range  rdfs:Literal
   </pre>

These statements say that the subjects of ``vendor2:warranty_code`` are
things typed as ``general:Product`` and the objects of this predicate are
literals (raw text). That's true. Recall:

*warranty_code*

.. raw:: html

   <pre>
   vendor2:product1  vendor2:warranty_code  "None."
   </pre>

``vendor2:product1`` is a product, and ``"None."`` is a literal value.

Specifying domains and ranges for predicates serves two purposes. First,
it allows applications to make inferences from statements about the
types of things. If it sees something that is the subject of
``vendor2:warranty_code``, it can infer that it is a ``general:Product``.
Second, these specifications serve as documentation of a vocabulary for
people. The RDF itself is used to indicate how predicates should be
used.

Two RDFS predicates are used to give relations between classes and
predicates. The ``rdfs:subClassOf`` relation indicates that one class is a
sub-class of another. For instance, the class Mammal is a sub-class of
the class Animal. Anything true of the Animal class is also true of the
Mammal class, and applications are able to make such inferences once
this predicate is present. The ``rdfs:subPropertyOf`` is similar, but for
predicates. For example, the friend predicate is a sub-property of the
knows predicate. Any friend is someone you know.

   `RDF Schema <http://www.w3.org/TR/rdf-schema/>`__ details the
   semantics of these properties.

Web Ontology Language (OWL)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Web Ontology Language (OWL) defines more classes that let RDF authors
define more of the meaning of their predicates within RDF. Four classes
of predicates defined by OWL include: ``owl:SymmetricProperty``,
``owl:TransitiveProperty``, ``owl:FunctionalProperty``, and
``owl:InverseFunctionalProperty``. (The OWL namespace is
http://www.w3.org/2002/07/owl.) Each of these classes is ``rdf:subClassOf``
``rdf:Property``.

Applications can use these classes, by convention, to make inferences
about data. You would use these classes in an ontology like this:

*Defining amazon:price*

.. raw:: html

   <pre>
   amazon:price    rdf:type    owl:FunctionalProperty
   </pre>

Because these classes are defined in the OWL ontology as being
sub-classes of ``rdf:Property``, applications can infer the following:

*Defining amazon:price*

.. raw:: html

   <pre>
   amazon:price    rdf:type    rdf:Property
   </pre>

That's the same statement as earlier. So, when you use a sub-class in
place of the ‘parent' class, you're being strictly more informative.
Anything the application knew before it still knows (if it has
inferencing capabilities and knows the OWL ontology), and it knows more
because the sub-class is more specific.

OWL symmetric properties tell applications that the following inference
is valid. If the application sees the statement S P O, and if P is typed
as a symmetric property, then O P S is also true. For instance, we think
of the has-friend relation are being symmetric. If you're my friend (ME
HAS_FRIEND YOU), I'm your friend (YOU HAS_FRIEND ME).

OWL transitive properties work like this. If the application sees the
statements X P Y and Y P Z, and if P is typed as a transitive property,
then X P Z is also true. ``rdfs:subClassOf`` is a transitive relation. If
Mammal is a sub-class of Animal and Animal is a sub-class of Organism,
then Mammal is a sub-class of Organism.

| OWL functional and inverse-functional properties indicate how many
  times a property can be used for a given subject or object. A
  functional property is one that has at most one value for any
  particular subject. An example is the ``hasBirthday`` relation between a
  person and his or her birthday. Everyone has just one birthday, so for
  any given subject (person), there can be just one object (birthday).
  But, the ``owns`` relation between an owner and ownee is not functional.
| People can own more than one thing.

Inverse functional properties do the same in reverse. For any object,
there is only one subject for a particular inverse functional property.
The has_ISBN relation is inverse functional. For any ISBN, there is only
one book that has that ISBN. The has_ISBN relation may not be
functional. Can a book have more than one ISBN number?

Functional and inverse-functional properties can be used by applications
to infer things like two entities denote the same thing. For instance,
take the following input:

*Inverse-Functional Properties*

.. raw:: html

   <pre>
   ex:isbn  rdf:type  owl:InverseFunctionalProperty
   book:a   ex:isbn   "12345-67890"
   book:b   ex:isbn   "12345-67890"
   </pre>

If this data is consistent, then the fact that ``ex:isbn`` is marked as an
inverse-functional property lets the application conclude ``book:a`` and
``book:b`` denote the very same book. They have the same ISBN, and since the
ISBN relation was marked as inverse-functional, then the two subjects
must denote the same book. Recall that two names can refer to the same
thing.

   `Web Ontology Language <http://www.w3.org/TR/owl-features/>`__
   defines the semantics of these predicates and classes.
