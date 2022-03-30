
Comparing RDF with XML
======================

Earlier I showed how RDF could be used to create a decentralized
database for product information and reviews. Here is how a similar
system would be accomplished using non-RDF XML.

First, though, I'll look at how a single vendor would approach this
alone. The vendor, if it was so inclined, might publish an XML file with
a node for each product, and within that a node for its name and some
vendor-specific information.

*Vendor 1's XML File*

.. raw:: html

   <pre>
   &lt;products&gt;
     &lt;product title="Cool-O-Matic"&gt;
       &lt;price&gt;50.75&lt;/price&gt;
       &lt;partno&gt;TTK583&lt;/partno&gt;
       ...
   </pre>

What can be done with this file? An application to display this
information would have to be specifically programmed to know that
``<product>`` nodes are for products, with titles in the ``title`` attribute,
etc. And, if a reviewer wanted to post a review XML file, the only way
to relate reviews to products would be by name. Two vendors might have
products with the same name, so vendors would have to use IDs of some
sort to keep their products separate.

The first problem arises. Vendors will need to come together to
establish a product ID system so that IDs are unique within the local ID
space of this vendor consortium. RDF solves this problem by requiring
that all IDs be globally unique, and by using URIs for IDs, allowing
individuals to create IDs in a local space that they control.

*Vendor 1's XML File - Revision 1*

.. raw:: html

   <pre>
   &lt;products&gt;
     &lt;product title="Cool-O-Matic" id="vendor1_product1"&gt;
       &lt;price&gt;50.75&lt;/price&gt;
       &lt;partno&gt;TTK583&lt;/partno&gt;
       ...
   </pre>

*Reviewer 1's XML File*

.. raw:: html

   <pre>
   &lt;reviews&gt;
     &lt;review product-title="Cool-O-Matic" id="reviewer1_review1"
       productid="vendor1_product1"&gt;
       &lt;description&gt;This product is just too cool.&lt;/description&gt;
       ...
   </pre>

With IDs in the XML files, reviewers will be able to identify products
in their review files, but applications still won't be able to relate
products to reviews. IDs aren't enough. The applications themselves have
to be told where to find the IDs. In Vendor 1's file, it's in the
product node's ``id`` attribute. In the review files, the same attribute is
to give an identifier to each review. To connect the review to the
product, another attribute is used. Even if the vendors and reviewers
agreed where to put the IDs, the application still needs to know where
it is. RDF solves this problem by making everything a global ID (except
literals), so basically anything the RDF application sees is an ID that
means something.

The vendors and reviewers next have to decide what constitutes a valid
product or review XML file, and how the nodes of these files should be
interpreted by software. If these files are defined by a DTD or Schema,
the files will not be extensible. Before adding anything new into these
files, such as vendor-specific information, all of the vendors and
reviewers will need to agree to a DTD or Schema change.

On the other hand, the vendors and reviewers can go without a DTD or
Schema. There are no rules for what elements go where, which provides
the flexibility that the vendors and reviewers need. But, then there
must be some guide to what the elements of the XML files mean, and thus
a central authority for deciding these things. Unless, the vendors and
reviewers use namespaces to allow them to develop their own
vocabularies.

I could go on, but you should see now that XML isn't particularly suited
for distributed, extensible information *unless* that XML looks a lot
like RDF!
