Reading and Writing RDF
=======================

In this section, we describe two standard ways of writing out RDF.
Because RDF is the abstract graph-like model, we call these written
formats serialization syntaxes for RDF.

::

   <a name="Notation 3"> </a>

Notation 3
~~~~~~~~~~

`Notation 3 <http://www.w3.org/DesignIssues/Notation3.html>`__ ("N3"),
or the subset called `Turtle <http://www.dajobe.org/2004/01/turtle/>`__,
is a de facto standard for writing out RDF. It is not a W3C standard,
but it is widely deployed, commonly used in email discussions between
SemWeb developers, and the most important RDF notation to understand
because it most clearly captures the abstract graph.

Here is an example, and it should be mostly clear what *triples* are
encoded by this document.

*Notation 3 Example*

.. raw:: html

   <pre>
   @prefix dc: &lt;http://purl.org/dc/elements/1.1/&gt; .
   @prefix geo: &lt;http://www.w3.org/2003/01/geo/wgs84_pos#&gt; .
   @prefix edu: &lt;http://www.example.org/&gt; .

   &lt;http://www.princeton.edu&gt; geo:lat "40.35" ; geo:long "-74.66" .
   &lt;http://www.cs.princeton.edu&gt; dc:title "Department of Computer Science" .
   &lt;http://www.princeton.edu&gt; edu:hasDept &lt;http://www.cs.princeton.edu&gt; .
   </pre>

In tabular form (actually also a standard called NTriples), this is:

*Tabular or NTriples Form*

.. raw:: html

   <pre>
   &lt;http://www.princeton.edu&gt; &lt;http://www.w3.org/2003/01/geo/wgs84_pos#lat&gt; "40.35" .
   &lt;http://www.princeton.edu&gt; &lt;http://www.w3.org/2003/01/geo/wgs84_pos#long&gt; "-74.66" .
   &lt;http://www.cs.princeton.edu&gt; &lt;http://purl.org/dc/elements/1.1/title&gt; "Department of..." .
   &lt;http://www.princeton.edu&gt; &lt;http://www.example.org/hasDept&gt; &lt;http://www.cs.princeton.edu&gt; .
   </pre>

This is meant to express the geographic latitude and longitude
coordinates of Princeton University, that it has a department, and that
that department is named "Department of Computer Science".

| In N3 and Turtle, statements are just written out as the subject URI
  (in brackets or abbreviated with namespaces), followed by the
  predicate URI, followed by the object URI or literal value, followed
  by a period.
| Also, namespaces are declared at the top with the ``@prefix`` directive.
  Full URIs are reassembled from the abbreviated notation, like ``geo:lat``,
  just by concatenating the corresponding namespace with the second part
  of the abbreviation, i.e. http://www.w3.org/2003/01/geo/wgs84_pos
   
+ ``lat`` = http://www.w3.org/2003/01/geo/wgs84_pos#lat.

N3 has some syntactic sugar that allows further abbreviations. If many
statements repeat the same subject and predicate, just separate the
objects with commas:

*N3 Syntactic Sugar: Commas*

.. raw:: html

   <pre>
   &lt;http://www.princeton.edu&gt; edu:hasDept &lt;http://www.cs.princeton.edu&gt; ,
       &lt;http://www.math.princeton.edu&gt; , &lt;http://history.princeton.edu&gt; .
   </pre>

And if the same subject is repeated, but with different predicates, one
may use semicolons as in the example:

*N3 Syntactic Sugar: Semicolons*

.. raw:: html

   <pre>
   &lt;http://www.princeton.edu&gt; geo:lat "40.35" ; geo:long "-74.66" .
   </pre>

N3 has a few other abbreviations as well. The common predicate rdf:type
can be abbreviated simply as the letter ``a`` (as in "is a").

Further, blank nodes are represented in N3 in two ways. In the first
way, they are given local names in the reserved underscore "_" prefix:

*Blank Nodes in Notation 3 - Method One*

.. raw:: html

   <pre>
   &lt;http://www.princeton.edu&gt; edu:hasDept _:deptOne, _:deptTwo .
   _:deptOne dc:title "Department of Computer Science" .
   _:deptTwo dc:title "Department of Psychology" .
   </pre>

The second way uses square brackets, and is recursive. Square brackets
represent a blank node, and inside the brackets you can put
predicate-object pairs separated by semicolons to encode properties of
that blank node. For example:

*Blank Nodes in Notation 3 - Method Two*

.. raw:: html

   <pre>
   &lt;http://www.princeton.edu&gt; edu:hasDept 
       [ dc:title "Department of Computer Science" ] ,
       [ dc:title "Department of Psychology" ].
   </pre>

The two blank node N3 examples encode precisely the same information as
in the previous example. Keep in mind that it's not the form of the N3
document that ever matters. Whether someone uses commas, semicolons,
brackets, or underscores is always besides the point. What matters is
what triples are encoded by the document.

   You should read the `Primer using
   N3 <http://www.w3.org/2000/10/swap/Primer.html>`__ for more examples.

::

   <a name="RDF/XML"> </a>

RDF/XML
~~~~~~~

The W3C specifications define an `XML
format <http://www.w3.org/TR/rdf-syntax-grammar/>`__ to encode RDF.
Since it works under the same abstract model as Notation 3, the
differences between the formats are superficial — readability.

The same information in the first N3 example above looks like this in
RDF/XML:

*RDF/XML Example*

.. raw:: html

   <pre>
   &lt;rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
       xmlns:dc="http://purl.org/dc/elements/1.1/"
       xmlns:geo="http://www. w3.org/2003/01/geo/wgs84_pos#"
       xmlns:edu="http://www.example.org/"&gt;

       &lt;rdf:Description rdf:about="http://www.princeton.edu"&gt;
           &lt;geo:lat&gt;40.35&lt;/geo:lat&gt;
           &lt;geo:long&gt;-74.66&lt;/geo:long&gt;
           &lt;edu:hasDept rdf:resource="http://www.cs.princeton.edu"/&gt;
       &lt;/rdf:Description&gt;

       &lt;rdf:Description rdf:about="http://www.cs.princeton.edu"&gt;
           &lt;dc:title&gt;Department of Computer Science&lt;/dc:title&gt;
       &lt;/rdf:Description&gt;

   &lt;/rdf:RDF&gt;
   </pre>

The questions you should be asking yourself are: What are the
standardized rules for interpreting the nodes and attributes in this
document as an abstract RDF graph, and what are the standardized rules
for producing a document in this form given an abstract RDF graph?

In an RDF/XML document there are two types of XML nodes:

1) resource XML nodes and 
2) property XML nodes. 
   
Resource XML nodes are the subjects and
objects of statements, and they usually are ``rdf:Description`` tags that
have an ``rdf:about`` attribute on them giving the URI of the resource they
represent. In this example, the ``rdf:Description`` nodes are the resource
nodes.

Resource XML nodes contain within them property XML nodes (and nothing
else). Each property XML node represents a single statement. The subject
of the statement is the outer resource XML node that contains the
property. There are four statements in this example, the first three
with the subject http://www.princeton.edu and the fourth with the
subject http://www.cs.princeton.edu. The URIs of the predicates in the
four statements are (abbreviated) ``geo:lat``, ``geo:long``, ``edu:hasDept``, and
``dc:title``.

Statements can have either resources or literal values as their objects,
as discussed earlier in this article. To put a literal value as the
object of a property XML node, the value simply goes inside the XML
element. See how "40.35" and "-74.66" are used above.

Using a resource as an object can be done in two ways. The first way is
illustrated in the example, by using a ``rdf:resource`` attribute in which
you put the URI of the object. You can describe properties of that
object elsewhere, as is done above in a separate ``rdf:Description``
resource XML node at the top-level of the document.

The second way is to tuck the ``rdf:Description`` element right inside the
property XML node and leave off ``rdf:resource``. The example above is
equivalently written as:

*RDF/XML Example Alternative*

.. raw:: html

   <pre>
   ...
   &lt;rdf:Description rdf:about="http://www.princeton.edu"&gt;
       &lt;geo:lat&gt;40.35&lt;/geo:lat&gt;
       &lt;geo:long&gt;-74.66&lt;/geo:long&gt;
       &lt;edu:hasDept&gt;
           &lt;rdf:Description rdf:about="http://www.cs.princeton.edu"&gt;
               &lt;dc:title&gt;Department of Computer Science&lt;/dc:title&gt;
           &lt;/rdf:Description&gt;
       &lt;/edu:hasDept&gt;
   &lt;/rdf:Description&gt;
   ...
   </pre>

From the specification we are told how to take the XML document above
and get out of it this table of statements. You can check that you
understand the format by comparing this document with the first N3
example and the explanation in the previous section. The two documents
encode exactly the same triples.

We'll discuss just one syntactic shortcut here because it is so commonly
used. Consider this example:

*RDF/XML Example With Types*

.. raw:: html

   <pre>
   &lt;rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
       xmlns:ex="http://www.example.org/"&gt;

       &lt;rdf:Description rdf:about="http://www.princeton.edu"&gt;
           &lt;rdf:type rdf:resource="http://www.example.org/University"/&gt;
       &lt;/rdf:Description&gt;
   &lt;/rdf:RDF&gt;
   </pre>

This example contains just a single triple. The ``rdf:type`` predicate is
used to say what type of thing a resource is. Here, the point is to say
that the resource denoted by http://www.princeton.edu is a university.
Because ``rdf:type`` is so common, rather than creating a property XML node
for it, it can be abbreviated by replacing the ``rdf:Description`` tag with
the type:

*RDF/XML Example With Types - Abbreviated*

.. raw:: html

   <pre>
   &lt;rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
       xmlns:ex="http://www.example.org/"&gt;

       &lt;ex:University rdf:about="http://www.princeton.edu"&gt;
       &lt;/ex:University&gt;
   &lt;/rdf:RDF&gt;
   </pre>

These two documents are completely equivalent. So, when something other
than ``rdf:Description`` is the name of a resource XML node, then there is a
``rdf:type`` triple encoded. ``rdf:Description`` is not itself a type. It is a
syntactic placeholder. It does not translate into a triple.

It is possible to use blank nodes in RDF/XML as well, and there are many
syntactic shortcuts that you will see commonly used. You will have to
consult the specifications to see these things, as it's time we move on.

Use a Validator!
~~~~~~~~~~~~~~~~

Triples are the bread and butter of RDF. When applications use RDF in
XML or N3 format, they see the triples. The hierarchical structure of
the XML and the order of the XML nodes is lost in the table of triples,
which means that, like whitespace, it was not a part of the information
meant to be encoded in the RDF. With XML and N3, the particular
syntactic or formatting choices are always besides the point. Make sure
you understand the triples encoded in any document!

When in doubt, you can use an online validator to convert RDF in one
format to another more triple-oriented format. Converting RDF/XML to N3
or, better yet, N-Triples is a good way to get a clear view of what
triples are encoded by the XML.

The W3C maintains a `RDF validator <http://www.w3.org/RDF/Validator/>`__
which converts an RDF document that you paste into a text box into a
table of RDF triples. This is good also to check your RDF syntax.

There is also a `validator on this site <https://github.com/JoshData/rdfabout/blob/gh-pages/demo/validator>`__ which can
convert back and forth between N3 and RDF/XML. You can use this to check
your RDF/XML or N3 syntax, and to double-check what triples are encoded
in N3 documents.

   See also: The authoratative definitions of `N3
   syntax <http://www.w3.org/2000/10/swap/Primer.html>`__ and `RDF/XML
   syntax <http://www.w3.org/TR/rdf-syntax-grammar/>`__.

