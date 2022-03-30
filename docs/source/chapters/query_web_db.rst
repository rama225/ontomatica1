Querying Semantic Web Databases
===============================

A Semantic Web or RDF database, also called a **triple store** because
it stores triples (which, if you recall, is another word for an RDF
statement), is generally any repository of RDF statements that supports
some form of querying operation. This is important, of course, because
once you have a document, you have to *ask* what is inside.

The process of asking abstracts over how the triples were originally
stored, such as in XML or Notation 3. As a result, triple stores often
cannot tell you what order statements appeared in in the original
document, what namespace, prefix, and local name were used for any given
resource, or what local name was used to identify blank nodes within
documents. These are all incidental artifacts of serializing RDF that
are not a part of the information content of the documents, after all,
and are accordingly meant to be forgotten.

What a triple store *can* tell you is what statements are inside (in an
unordered fashion). Usually a triple store can answer the question of
whether a particular statement that you provide is contained in the
store, and some will be able to provide a set of statements that match a
given pattern.

Triple stores can roughly be divided into two types. On the one hand,
there are pure, low-level triple stores that maintain a repository of
triples and answer questions about the existence of triples in the
repository. On the other hand, there are triple stores that can answer
more complicated questions that rely on the semantics of RDFS and OWL
(described above), or on other rules of logic provided specially. We
will discuss the former type only.

   For a list of triple store applications, see `LargeTripleStores on
   the ESW wiki <http://esw.w3.org/topic/LargeTripleStores>`__.

As in the land of RDBMSs and XML, there is for RDF also a standard query
language, and it is called SPARQL. The lexicographic similarity to SQL
is not an accident. SPARQL is a query language for RDF modeled roughly
after SQL, although they are quite different. Further, the SPARQL
standard is only concerned with asking queries. No part of the SPARQL
standard addresses adding statements into a triple store or otherwise
modifying the contents of a triple store. Triple stores of course may
support these operations, but not through the SPARQL standard.

As with the introductions of Notation 3 and RDF/XML above, this section
is meant to provide only the very basics of the SPARQL language.

Let's start with some example data set in a hypothetical RDF repository:

*RDF about My Apartment*

.. raw:: html

   <pre>
   @prefix ex: &lt;http://example.org/&gt; .
   @prefix taubz: &lt;http://razor.occams.info/index.html#&gt; .
   taubz:me            ex:own       taubz:my_apartment .
   taubz:me            ex:own       taubz:my_computer .
   taubz:my_apartment  ex:contains  taubz:my_computer .
   taubz:my_apartment  ex:contains  taubz:friends_junk .
   taubz:my_apartment  ex:location  &lt;http://example.org/Philadelphia&gt; .
   taubz:me            ex:own       taubz:my_desk .
   taubz:my_desk       ex:contains  taubz:my_pens_and_pencils .
   </pre>

The SPARQL language has four types of queries: SELECT, ASK, DESCRIBE,
and CONSTRUCT.

The SELECT query is most similar to SQL in that you provide a query and
you get back a tabular response. However, the similarity pretty much
ends there. In SPARQL's SELECT query, you are asking to find
*resources*. For instance, a query might ask for which resources "own"
"my apartment". That would be to say, which resources *x* can be found
in statements in the repository of the form \ ``x ex:own
taubz:my_apartment``. The answer, according to the example repository data
above, would be just ``taubz:me``. Or, which resources are in my apartment?
That is, which resources are the objects *x* of statements like
``taubz:my_apartnemtn ex:contains x``. This differs from SQL, in which
each row of results is a row from a table. Here, each row is a resource.

SPARQL, like SQL, has a WHERE clause. However, the WHERE clause in
SPARQL is a graph pattern. A graph pattern is like a graph, in Notation
3 format, except that besides URIs, bnodes, and literals, you are also
permitted to use *variables*. Variables are placeholders, precisely like
the *x* above. The variables represent the resources you are looking
for. The second question above would be written in SPARQL as:

*SPARQL Query Example 1*

.. raw:: html

   <pre>
   PREFIX ex: &lt;http://example.org/&gt;
   PREFIX taubz: &lt;http://razor.occams.info/index.html#&gt;
   SELECT ?what
   WHERE {
       taubz:my_apartment ex:contains ?what .
   }
   </pre>

The results would be:

*SPARQL Query Example 1 Results*

.. raw:: html

   <pre>
        ?what
   -----------------
   taubz:my_computer
   taubz:friends_junk
   </pre>

A few things are of note. First, the name of the variable ``?what`` is
incidental, as are names in general in RDF, except that it must begin
with a question mark. Second, while some things look a lot like N3, such
as the graph pattern syntax, other parts do not, such as the PREFIX
declarations. Lastly, the output of a SPARQL query is, abstractly, a
table. How you access the table depends on the SPARQL engine you use.
However, there is a standard XML output format. We won't look at that
format here, however.

Graph patterns can contain more that one statement. This has the effect
of either 1) filtering the resources that might be returned, or 2)
querying based on longer paths through the repository data. For
instance, to ask what is in my apartment *that I own*, that is, to
filter the results to contain only the resources that I own, add a
second statement to the graph pattern:

*SPARQL Query Example 2*

.. raw:: html

   <pre>
   PREFIX ex: &lt;http://example.org/&gt;
   PREFIX taubz: &lt;http://razor.occams.info/index.html#&gt;
   SELECT ?what
   WHERE {
       taubz:my_apartment ex:contains ?what .
       **taubz:me ex:own ?what .**
   }
   </pre>

The results would be:

*SPARQL Query Example 2 Results*

.. raw:: html

   <pre>
        ?what
   -----------------
   taubz:my_computer
   </pre>

Each resource returned must be able to substitute into all occurrences
of the variable, ``?what``, so that each statement in the graph pattern
exists in the repository. The resource ``taubz:friends_junk`` is ruled out
because when we substitute that resource into the graph pattern,
yielding

*Graph Pattern with Resource Substituted*

.. raw:: html

   <pre>
   taubz:my_apartment ex:contains taubz:friends_junk .
   **taubz:me ex:own taubz:friends_junk .**
   </pre>

one of those statements, namely the second one, does not exist in the
repository.

A query to find resources contained in *any* object I own (my computer
which is contained in my apartment, my friend's junk which is also
contained in my apartment, and my pens and pencils which are contained
in my desk) would use a graph pattern that has a longer path through
resources in the repository. We can rephrase this query as a conjunction
of two statements, as before, but this time using two variables. This
is: For any pair of resources *x* and *y* that we can find, such that A)
``taubz:me ex:own y``  and B) ``y ex:contains x`` , report resource
*x*. Here is the SPARQL query and expected output:

*SPARQL Query Example 3*

.. raw:: html

   <pre>
   PREFIX ex: &lt;http://example.org/&gt;
   PREFIX taubz: &lt;http://razor.occams.info/index.html#&gt;
   SELECT ?what
   WHERE {
       **taubz:me ex:own ?container .
       ?container ex:contains ?what .**
   }
   </pre>

*SPARQL Query Example 3 Results*

.. raw:: html

   <pre>
            ?what
   -------------------------
   taubz:my_computer
   taubz:friends_junk
   taubz:my_pens_and_pencils
   </pre>

To push this example further, rather than asking just for the list of
objects, we can ask for the pairs of ``?container`` and ``?what`` values that go
together. The only change is adding the ?container variable to the
SELECT line:

*SPARQL Query Example 4*

.. raw:: html

   <pre>
   PREFIX ex: &lt;http://example.org/&gt;
   PREFIX taubz: &lt;http://razor.occams.info/index.html#&gt;
   SELECT **?container** ?what
   WHERE {
       taubz:me ex:own ?container .
       ?container ex:contains ?what .
   }
   </pre>

*SPARQL Query Example 4 Results*

.. raw:: html

   <pre>
      ?container               ?what
   ------------------  -------------------------
   taubz:my_apartment  taubz:my_computer
   taubz:my_apartment  taubz:friends_junk
   taubz:my_desk       taubz:my_pens_and_pencils
   </pre>

Each row is verified by substituting for each variable in the graph
pattern the corresponding value listed in that row, one row at a time.
As before, if after substituting all of the values for the variables all
of the statements in the graph pattern occur in the repository, the row
is included in the result output.

Before we conclude, here is some more terminology. Within any particular
result row, a variable is said to be "**bound**" by the value for that
variable in that row. In the first row in the table for example 4 above,
the variable ``?container`` is bound by ``taubz:my_apartment``.

Until a future revision of this article, we won't go into SPARQL any
further.

   See the `SPARQL
   specification <http://www.w3.org/TR/rdf-sparql-query/>`__ for more on
   the query language.

