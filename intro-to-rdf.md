# What is RDF and what is it good for?

Joshua Tauberer

July 2006

This is an introduction to RDF (Resource Description 
Framework), the standard for encoding knowledge that forms the basis
of the Semantic Web, in which computer applications make use of
distributed, decentralized, structured information spread throughout
the current web.  RDF is often 
thought of as an XML format, but this article is from the view of RDF as 
an abstract data structure, independent of how it is written.
This article is geared toward those familiar with XML and programming.

* * *

1.  [Why we need a new standard for the Semantic Web](#Why we need a new standard for the Semantic Web)
2.  [Introducing RDF](#Introducing RDF)
3.  [Triples of knowledge](#Triples of knowledge)
    1.  [The abstract RDF model defined](#The abstract RDF model defined)
    2.  [RDF as a Graph](#RDF as a Graph)
    3.  [Blank Nodes and Literal Values](#Blank Nodes and Literal Values)
4.  [Reading and writing RDF: Serialization syntaxes](#Reading and writing RDF: Serialization syntaxes)
    1.  [The RDF/XML format](#The RDF/XML format)
    2.  [Notation 3](#Notation 3)
5.  [Distributed Information](#Distributed Information)
    1.  [Choosing the Right Predicates](#Choosing the Right Predicates)
    2.  [Meshing Information: A Real-World Example](#Meshing Information: A Real-World Example)
6.  [Comparing RDF with XML](#Comparing RDF with XML)
7.  [RDF about RDF](#RDF about RDF)
    1.  [RDF Schema (RDFS)](#RDF Schema)
    2.  [Web Ontology Language (OWL)](#Web Ontology Language)
8.  [Closing Remarks](#Closing Remarks)

This document was originally written in October 2005.
In July 2006 it was revised and extended with material
from my [xml.com](http://www.xml.com/pub/a/2001/01/24/rdf.html)
article "What is RDF".

<a name="Why we need a new standard for the Semantic Web"> </a>

## Why we need a new standard for the Semantic Web

On the [Semantic Web](http://www.w3.org/2001/sw/), computers do the browsing for us.  The SemWeb enables computers to seek out knowledge distributed throughout the Web, mesh it, and then take action based on it.  To use an analogy, the current Web is a decentralized platform for distributed _presentations_ while the SemWeb is a decentralized platform for distributed _knowledge_.  [RDF](http://www.w3.org/RDF/) is the W3C standard for encoding knowledge.

There of course is knowledge on the current Web, but it's off limits to computers.  Consider a [Wikipedia](http://www.wikipedia.org) page, which might convey a lot of information to the human reader, but to the computer displaying the page all it sees is presentation markup.  To the extent that computers make sense of HTML, images, Flash, etc., it's almost always for the purpose of creating a presentation for the end-user.   The real content, the knowledge the files are conveying to the human, is opaque to the computer.

What is meant by “[semantic](http://en.wikipedia.org/wiki/Semantics)” in the Semantic Web is not that computers are going to understand the meaning of anything, but that the logical pieces of meaning can be mechanically manipulated by a machine to useful ends.

So now imagine a new Web where the real content can be manipulated by computers.  For now, picture it as a web of databases.  One “semantic” website publishes a database about a product line, with products and descriptions, while another publishes a database of product reviews.  A third site for a retailer publishes a database of products in stock.  What standards would make it easier to write an application to mesh distributed databases together, so that a computer could use the three data sources together to help an end-user make better purchasing decisions?

There's nothing stopping anyone from writing a program now to do those sorts of things, in just the same way that nothing stopped anyone from exchanging data before we had XML.  But standards facilitate building applications, especially in a decentralized system.  Here are some of the things we would want a standard about _distributed knowledge_ to consider:

1.  Files on the Semantic Web need to be able to express information flexibly.  Life can't be neatly packed into tables, as in relational databases, or hierarchies, as in XML.  The information about movies and TV shows contained in the graph below is really best expressed _as a graph_:

	![](whatisrdf_1.gif)

_Knowledge as a Graph_

Of course, we can't be drawing our way through the Semantic Web, so instead we will need a tabular notation for these graphs a bit like this:

<table>
	<tr><th>Start Node</th> <th>Edge Label</th> <th>End Node</th></tr>
	<tr><td>vincent_donofrio</td> <td>starred_in</td> <td>law_&amp;_order_ci</td></tr>
	<tr><td>law_&amp;_order_ci</td> <td>is_a</td> <td>tv_show</td> </tr>
	<tr><td>the_thirteenth_floor</td> <td>similar_plot_as</td> <td>the_matrix</td> </tr>
	<tr><td>...</td></tr>
</table>

Each row of the table specifies an edge from one node in the graph to another. More on this later.

2.  Files on the Semantic Web need to be able to relate to each other.  A file about product prices posted by a vendor and a file with product reviews posted independently by a consumer need to have a way of indicating that they are talking about the same products.  Just using product names isn't enough.  Two products might exist in the world both called "The Super Duper 3000," and we want to eliminate ambiguity from the SemWeb so that computers can process the information with certainty.  The SemWeb needs globally unique identifiers that can be assigned in a decentralized way.

3.  We will use vocabularies for making assertions about things, but these vocabularies must be able to be mixed together.  A vocabulary about TV shows developed by TV aficionados and a vocabulary about movies independently developed by movie connoisseurs must be able to be used together in the same file, to talk about the same things, for instance to assert that an actor has appeared in both TV shows and movies.

These are the requirements that RDF, Resource Description Framework, provides a standard for, as we'll see in the next section.  Before getting too abstract, here are actual RDF examples of the information from the graph above, first in the Notation 3 format, which closely follows the tabular encoding of the underlying graph:

_Notation 3 Example_
<pre>
@prefix rdf: &lt;http://www.w3.org/1999/02/22-rdf-syntax-ns#&gt; .
@prefix ex: &lt;http://www.example.org/&gt; .

ex:vincent_donofrio ex:starred_in ex:law_and_order_ci .
ex:law_and_order_ci rdf:type ex:tv_show .
ex:the_thirteenth_floor ex:similar_plot_as ex:the_matrix .
</pre>

And in the standard RDF/XML format, which may have a more intuitive feel but tends to obscure the underlying graph:

_RDF/XML Example_
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

RDF was [originally](http://www.w3.org/TR/1999/REC-rdf-syntax-19990222/) created in 1999 as a standard on top of XML for encoding metadata — literally, data about data.  Metadata is of course things like who authored a Web page, what date a blog entry was published, etc., information that is in some sense _secondary_ to some other content already on the regular Web.  Since then, and perhaps even after [the updated RDF spec in 2004](http://www.w3.org/TR/rdf-primer/), the scope of RDF has really evolved into something greater.  The most exiting uses of RDF aren't in encoding information about Web resources, but information about and relations between things in the real world: people, places, concepts, etc.

<a name="Introducing RDF"> </a>

## Introducing RDF

Unless you know Resource Description Framework (RDF) well, it's 
probably best if you try to forget what you already know about it.  RDF 
exists at the intersection of a few different technologies, so it's easy 
to be lead into thinking that it is merely a particular XML data format or a 
tool for blog feeds.  Forget what you know.  Here is RDF from the 
beginning.

RDF is a general method to decompose knowledge into small pieces, with some
rules about the semantics, or meaning, of those pieces. The 
point is to have a method so simple that it can express any fact, and 
yet so structured that computer applications can do useful things with knowledge 
expressed in RDF. I say "method" in particular, rather than format, because one 
can write down those pieces in any number of ways and still preserve the 
original information and structure, just like how one can express the same meaning in 
different human languages or implement the same data structure in
multiple ways.

In some ways, RDF can be compared to XML.  XML also is designed to be 
simple and applicable to any type of data.  XML is also more than a file 
format. It is a foundation for dealing with hierarchical, self-contained
documents, whether they be stored on disk in the usual brackets-and-slashes format, 
or held in memory.

What sets RDF apart from XML is that RDF is designed to represent 
_knowledge_ in a _distributed_ world. That RDF is designed for 
knowledge, and not data, means RDF is particularly concerned with 
meaning.  Everything at all mentioned in RDF means something.  It may be 
a reference to something in the world, like a person or movie, or it may 
be an abstract concept, like the state of being friends with someone 
else.  And by putting three such entities together, the RDF standard says 
how to arrive at a fact.  The meaning of the triple (John, Bob, the state of being 
friends) is that John and Bob are friends.  By putting a lot of facts 
together, one arrives at some form of knowledge.  Standards built on top of
RDF, including RDFS and OWL, add to RDF semantics for drawing logical
inferences from data.

For comparison, XML itself is not very much concerned with meaning.
XML nodes don't need to be associated with particular concepts, and
the XML standard doesn't indicate how to derive a fact from a document.
For instance, if you were presented with a few XML documents whose root nodes
were in a foreign language you don't understand, you couldn't do anything
useful with the documents but display them.  RDF documents with nodes
you can't understand could still actually be usefully processed because RDF specifies
some basic level of meaning.  Now, this isn't to say that you couldn't
develop your own standard on top of XML that says how to derive
the set of facts in an XML document, but you'll find you've probably
just reinvented something like RDF.

The second key aspect of RDF is that it works well for distributed
information.  That is, RDF applications can put together RDF files
posted by different people around the Internet and easily learn from them new
things that no single document asserted.  It does this in two ways,
first by linking documents together by the common vocabularies they use,
and second by allowing any document to use any vocabulary.  This allows
enormous flexibility in expressing facts about a wide range of things,
drawing on information from a wide range of sources.

> For the official documentation on RDF, start with the [RDF Primer](http://www.w3.org/TR/rdf-primer/).

<a name="Triples of knowledge"> </a>

## Triples of knowledge

RDF provides a general, flexible method to decompose any knowledge into small pieces, called triples, with some rules about the semantics (meaning) of those pieces.

The foundation is breaking knowledge down into a labeled, directed graph. Each edge in the graph represents a fact, or a relation between two things.  The edge in the example from the node <tt>vincent_donofrio</tt> labeled <tt>starred_in</tt> to the node <tt>the_thirteenth_floor</tt> represents the fact that actor Vincent D'Onofrio starred in the movie "The Thirteenth Floor."  A fact represented this way has three parts: a subject, a predicate (i.e. verb), and an object.  The subject is what's at the start of the edge, the predicate is the type of edge (its label), and the object is what's at the end of the edge.

The six documents composing [the RDF specification](http://www.w3.org/TR/rdf-primer/) tell us two things.  First, it outlines [the abstract model](http://www.w3.org/TR/2004/REC-rdf-concepts-20040210/), i.e. how to use triples to represent knowledge about the world.  Second, it describes [how to encode those triples in XML](http://www.w3.org/TR/rdf-syntax-grammar/).

<a name="The abstract RDF model defined"> </a>

### The abstract RDF model defined

RDF is nothing more than a general method to decompose information into 
pieces.  The emphasis is on general here because the same method can be 
used for any type of information.  And the method is this: **Express 
information as a list of statements in the form SUBJECT PREDICATE 
OBJECT.**  The _subject_ and _object_ are names for two things in the world, 
and the _predicate_ is the name of a relation between the two.  You can 
think of predicates as verbs.

Here's how I would break down information about my apartment into RDF 
statements:

<table style="text-align: center">
<tr><th>SUBJECT</th> <th>PREDICATE</th> <th>OBJECT</th></tr>
<tr><td>I</td> <td>own</td> <td>my_apartment</td></tr>
<tr><td>my_apartment</td> <td>has</td> <td>my_computer</td></tr>
<tr><td>my_apartment</td> <td>has</td> <td>my_bed</td></tr>
<tr><td>my_apartment</td> <td>is_in</td> <td>Philadelphia</td></tr>
</table>

These four lines express four facts.  Each line is called a **statement**
or **triple**.

The subjects, predicates, and objects in RDF are always simple
names for things: concrete things, like _my_apartment_, or abstract concepts,
like _has_.  These names don't have internal structure or significance
of their own.  They're like proper names or variables.  It doesn't matter
what name you choose for anything, as long as you use it consistently
throughout.

Names in RDF statements are said to **refer to** or **denote** things in
the world.  The things that names denote are called **resources** (dating back
to RDF's use for metadata for web resources),
**nodes** (from graph terminology), or **entities**.  For instance, the name
_my_apartment_ denotes my actual apartment, which is an entity in the
real world.  The distinction between names and the entities they denote
is minute but important because two names can be used to refer to the
same entity.

Predicates are always relations between two things.  _Own_ is a 
relation between an owner and an 'ownee'; _has_ is a relation between the 
container and the thing contained; _is_in_ is the inverse relation, 
between the contained and the container.  In RDF, the order of the subject
and object is very important.

The next aspect of RDF almost goes without saying, but I want to put 
everything down in print: **If someone refers to something as X in one place 
and X is used in another place, the two X's refer to the same entity.**
When I wrote _my_apartment_ in the first line, it's the 
same apartment that I meant when I wrote it in the other three 
lines.

The rules so far already get us a lot farther than you might realize.  Given this 
table of statements, I can write a simple program that can answer 
questions like "who own my_apartment" and "my_apartment has what."  The 
question itself is in the form of an RDF statement, except the program 
will consider wh-words like who and what to be wild-cards.  A simple question-answering 
program can compare the question to each row in the table.  Each 
matching row is an answer.  Here's the pseudocode:

_Pseudocode for Question-Answering_
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

The computer doesn't need to know what _has_ actually means in English for 
this to be useful.  That is, it's left up to the application writer to 
choose appropriate names for things (e.g. my_apartment) and to use the 
right predicates (own, has).  RDF tools are ignorant of what these names 
mean, but they can still usefully process the information.  (I'll get to 
more useful things later.)

RDF information is meant to be published on the Internet, and so the 
names I used above have a problem.  I shouldn't name something 
my_apartment because someone else might use the name my_apartment for 
their apartment too.  Following from the last fact about RDF, RDF tools 
would think the two instances of my_apartment referred to the same thing 
in the real world, whereas in fact they were intended to refer to two 
different apartments.  The last aspect of RDF is that names must be 
global, in the sense that you must not choose a name that someone else 
might conceivably also use to refer to something different.  Formally, 
**names for subjects, predicates, and objects must be Uniform Resource 
Identifiers (URIs)**.

URIs can have the same syntax or format as website addresses, so you will 
see RDF files that contain URIs like 
<tt>http://www.w3.org/1999/02/22-rdf-syntax-ns#type</tt>, where that URI is the 
global name for some entity.  The fact that it looks like a web address is totally 
incidental.  There may or may not be an actual website at that address, 
and it doesn't matter.  There are other types of URIs besides <tt>http:</tt>-type
URIs.  URNs are a subtype of URI used for things like identifying books
by their ISBN number, e.g. <tt>urn:isbn:0143034650</tt>.  [TAGs](http://www.taguri.org/07/draft-kindberg-tag-uri-07.html) are a general-purpose
type of URI.  They look like <tt>tag:govtrack.us,2005:congress/senators/frist</tt>.  This article will use a mix of these three URI types.

Whatever their form, URIs you see in RDF documents are merely 
verbose names for entities, nothing more.

URIs are used as global names because they provide a way to break down 
the space of all possible names into units that have obvious owners.  
URIs that start with <tt>http://www.govtrack.us/</tt> are implicitly controlled 
by me, or whoever is running the website at that address.  By 
convention, if there's an obvious owner for a URI, no one but that owner 
will "mint" a new resource with that URI.  This prevents name clashes.  
If you create a URI in the space of URIs that you control, you can rest 
assured no one will use the same URI to denote something else.

Since URIs can be quite long, in various RDF notations they're 
usually abbreviated using the concept of namespaces from XML.  As in 
XML, a namespace is generally declared at the top of an RDF document and 
then used in abbreviated form later on.  Let's say I've declared the 
abbreviation <tt>taubz</tt> for the URI <tt>tag:tauberer@for.net,2005:</tt>.  
In many RDF notations, I can then abbreviate URIs like 
<tt>tag:tauberer@for.net,2005:my_apartment</tt> by replacing the namespace 
URI exactly as it is given in the declaration with the abbreviation and 
a colon, in this case simply as <tt>taubz:my_apartment</tt>. The precise 
rules for namespacing depend on the RDF serialization syntax being 
used.

I might re-write the table about my apartment as it is below, replacing 
the simple names I first used above with abritrary URIs:

<table style="text-align: center; font-size: 85%; font-family: Arial" cellspacing="5">
<tr><th>SUBJECT</th> <th>PREDICATE</th> <th>OBJECT</th></tr>
<tr><td>tag:tauberer@for.net,2005:me</td> <td>http://example.org/own</td> <td>tag:tauberer@for.net,2005:my_apartment</td></tr>
<tr><td>tag:tauberer@for.net,2005:my_apartment</td> <td>http://example.org/has</td> <td>tag:tauberer@for.net,2005:my_computer</td></tr>
<tr><td>tag:tauberer@for.net,2005:my_apartment</td> <td>http://example.org/has</td> <td>tag:tauberer@for.net,2005:my_bed</td></tr>
<tr><td>tag:tauberer@for.net,2005:my_apartment</td> <td>http://example.org/is_in</td> <td>http://example.org/Philadelphia</td></tr>
</table>

And I can simplify that further as:

_RDF about My Apartment_
<pre>
taubz:me            http://example.org/own    taubz:my_apartment
taubz:my_apartment  http://example.org/has    taubz:my_computer
taubz:my_apartment  http://example.org/has    taubz:my_bed
taubz:my_apartment  http://example.org/is_in  http://example.org/Philadelphia
</pre>

And that's RDF.  Everything else in the Semantic Web builds on those
three rules, repeated here to hammer home the simplicity of the system:

1.  A fact is expressed as a triple of the form (Subject, Predicate, Object).
2.  Subjects, predicates, and objects are given as names for entities, whether
	concrete or abstract, in the real world.
3.  Names are in the format of URIs, which are global and refer to
	the same entity in any RDF document in which they appear.

These concepts form most of the abstract RDF model for encoding knowledge.  It's analogous to the common API that most XML libraries provide.  If it weren't for us curious humans always peeking into files, the actual format of XML wouldn't matter so much as long as we had our <tt>appendChild</tt>, <tt>setAttribute</tt>, etc.  Of course, we do need a common file format for exchanging data, and in fact there are two for RDF, which we look at later.

<a name="RDF as a Graph"> </a>

### RDF as a Graph

There are two complementary ways of looking at RDF information.
The first is as a set of statements, like above.  Each statement
represents a fact.  The second way is as a graph.

A graph is basically a network.  Graphs consist of nodes interconnected
by edges.  In the Internet, for instance, the nodes are the computers, and the edges
are the ethernet wires connecting them.  In RDF, the nodes are names (_not_ actual entities)
and the edges are statements.  Here's an example:

![](rdfasagraph.png)

_RDF as a Graph_

Each arrow or edge is a RDF statement.  The name at the
start of the arrow is the statement's subject, the name at
the end of the arrow is the statement's object, and the
name that labels the arrow is the predicate.

RDF as a graph expresses exactly the same information as
RDF written out as triples, but the graph form makes it
easier for us humans to see structure in the data.  Not all RDF can be
expressed as a graph like this, however.  Predicates themselves can
be the subject or object of a statement (more on this later), but
this graph representation would not allow that because in this form
predicates are edges and not nodes.

<a name="Blank Nodes and Literal Values"> </a>

### Blank Nodes and Literal Values

There is actually a bit more to RDF than the three rules above. So 
far I've described three types of things in RDF: resources (things or 
concepts) that exist in the real world, global names for resources, and 
RDF statements.  There are two more things.

The first new thing is the **literal value**.  Literal values
are raw text that can be used instead of objects in RDF triples.
Unlike names (i.e. URIs) which are stand-ins for things in the real world,
literal values are just raw text data inserted into the graph.
Literals values could be used to relate people to their names,
books to their ISBN numbers, etc.

Then there are **anonymous** or **blank nodes**.  These
are nodes in an RDF graph that are unlabeled, either because we don't
know the global name for the node, or there isn't one for whatever
reason.  Not every thing in the real world needs to have its own
name, after all.

Without a name, it would be impossible to write out statements
about anonymous nodes since there would be no way to refer to them.
RDF notations solve this by using temporary, local names for
anonymous resources.  These names are only meaningful within
the document they're used in.

Here's one way literal values and anonymous nodes are used:

_Blank Nodes and Literal Values_
<pre>
taubz:me               foaf:name    "Joshua Tauberer"
taubz:me               ex:has_read  &lt;urn:isbn:0143034650&gt;
&lt;urn:isbn:0143034650&gt;  dc:title     "Free Culture : The Nature and Future of Creativity"
&lt;urn:isbn:0143034650&gt;  dc:author    _:anon123
_:anon123              foaf:name    "Lawrence Lessig"
</pre>

To distinguish between URIs, namespaced names (abbreviated URIs),
anonymous nodes, and literal values, I used the following common convention:

*   URIs are enclosed in angle brackets
*   Namespaced names are written plainly, but their colons give them away
*   Anonymous nodes are written like namespaced names, but in the reserved "_" namespace with an arbitrary local name after the colon
*   Literal values are enclosed in quotation marks

There is one blank node in this example, <tt>_:anon123</tt>.  What we know
about this resource is that it is the author of <tt>&lt;urn:isbn:0143034650&gt;</tt>
and it has the name _Lawrence Lessig_.  Because no global name is used
for this resource, we can't really be sure who we're talking about here (that is,
there's possibly more than one person whose name is _Lawrence Lessig_ in the world).  And,
if we wanted to say more about whatever is denoted by <tt>_:anon123</tt>, 
we would have to do it in this very RDF document because we would have no 
way to refer to this particular Lawrence Lessig outside of the document.

Anonymous nodes are often used like this to avoid having to assign
people URIs.  They're also often used in representing more complex relations:

_Blank Nodes for Complex Relations_
<pre>
taubz:me   ex:hasName     _:anon234
_:anon234  ex:firstName   "Joshua"
_:anon234  ex:middleName  "Ian"
_:anon234  ex:lastName    "Tauberer"
</pre>

Here the anonymous node was used as an intermediate step in the 
relation between me and the parts of my name.  The node represents my 
name in a structured way, rather than using a single opaque literal value
"Joshua Ian Tauberer".  RDF only allows binary relations, so it's 
necessary to express many-way relations like this (between a person, a 
first name, a middle name, and a last name) using intermediate nodes.
They could be named nodes with URIs, but there is a certain amount
of strangeness in giving things like people's names their own URIs.

<a name="Reading and writing RDF: Serialization syntaxes"> </a>

## Reading and writing RDF: Serialization syntaxes

<a name="The RDF/XML format"> </a>

### The RDF/XML format

In the previous section we covered the abstract RDF model.  Now we turn to how actually to write RDF in two formats.  The W3C specifications define an [XML format](http://www.w3.org/TR/rdf-syntax-grammar/) to encode RDF.  Here's an example:

_RDF/XML Example_
<pre>
&lt;rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
    xmlns:dc="http://purl.org/dc/elements/1.1/"
    xmlns:geo="http://www. w3.org/2003/01/geo/wgs84_pos#"
    xmlns:edu="http://www.example.org/"&gt;

    &lt;rdf:Description rdf:about="http://www.princeton.edu"&gt;
        &lt;geo:lat&gt;40.35&lt;/geo:lat&gt;
        &lt;geo:long&gt;-74.66&lt;/geo:long&gt;
        &lt;edu:hasDept rdf:resource="http://www.cs.princeton.edu"
            dc:title="Department of Computer Science"/&gt;
    &lt;/rdf:Description&gt;

&lt;/rdf:RDF&gt;
</pre>

In an RDF/XML document there are two types of nodes: resource nodes and property nodes.  Resource nodes are the subjects and objects of statements, and they usually have an <tt>rdf:about</tt> attribute on them giving the URI of the resource they represent.  In this example, the <tt>rdf:Description</tt> node is the only resource node, representing whatever was meant by the name &lt;http://www.princeton.edu&gt; (probably Princeton University itself, and _not_ the website at that address).

Resource nodes contain (only) property nodes, which represent statements.  There are three statements in this example, all with the subject &lt;http://www.princeton.edu&gt;, and with the predicates <tt>geo:lat</tt>, <tt>geo:long</tt>, and <tt>edu:hasDept</tt>.

Property nodes in turn contain literal values, like "40.35" and "-74.66", or a reference to an object resource using the <tt>rdf:resource</tt> attribute, or they may contain a full resource node as their object.

From the specification we are told how to take the XML document above and get out of it this table of statements:

_The Underlying Triples_
<pre>
            Subject            Predicate              Object
----------------------------- ----------- ------------------------
&lt;http://www.princeton.edu&gt;    edu:hasDept &lt;http://www.cs.princeton.edu&gt;
&lt;http://www.princeton.edu&gt;    geo:lat     "40.35"
&lt;http://www.princeton.edu&gt;    geo:long    "-74.66"
&lt;http://www.cs.princeton.edu&gt; dc:title    "Department of Computer Science"
</pre>

These triples are the bread and butter of RDF.  When applications use RDF in XML format, they see the triples. Note that the hierarchical structure of the XML and the order of the nodes is lost in the table of triples, which means that, like whitespace, it was not a part of the information meant to be encoded in the RDF.

<a name="Notation 3"> </a>

### Notation 3

[Notation 3](http://www.w3.org/DesignIssues/Notation3.html) (“N3”), or [Turtle](http://www.dajobe.org/2004/01/turtle/), is another system for writing out RDF.  Since it works under the same abstract model, the difference between it and RDF/XML is superficial — readability.

The same information in the RDF/XML file written in N3 looks like this:

_Notation 3 Example_
<pre>
@prefix rdf: &lt;http://www.w3.org/1999/02/22-rdf-syntax-ns#&gt; .
@prefix dc: &lt;http://purl.org/dc/elements/1.1/&gt; .
@prefix geo: &lt;http://www. w3.org/2003/01/geo/wgs84_pos#&gt; .
@prefix edu: &lt;http://www.example.org/&gt; .

&lt;http://www.princeton.edu&gt; geo:lat "40.35" ; geo:long "-74.66" .
&lt;http://www.cs.princeton.edu&gt; dc:title "Department of Computer Science" .
&lt;http://www.princeton.edu&gt; edu:hasDept &lt;http://www.cs.princeton.edu&gt; .
</pre>

In N3 and Turtle, statements are just written out as the subject URI (in brackets or abbreviated with 
namespaces), followed by the predicate URI, followed by the object URI or literal value, followed by a period.  
Also, namespaces are declared at the top with the <tt>@prefix</tt> directive.  Full URIs are reassembled
from the abbreviated notation, like <tt>geo:lat</tt>, just by concatenating the corresponding
namespace with the second part of the abbreviation, i.e. <tt>http://www. w3.org/2003/01/geo/wgs84_pos#</tt>
+ <tt>lat</tt> = <tt>http://www. w3.org/2003/01/geo/wgs84_pos#lat</tt>.

N3 has some syntactic sugar that allows further abbreviations.  If
many statements repeat the same subject and predicate, just separate
the objects with commas:

_N3 Syntactic Sugar: Commas_
<pre>
&lt;http://www.princeton.edu&gt; edu:hasDept &lt;http://www.cs.princeton.edu&gt; ,
	&lt;http://www.math.princeton.edu&gt; , &lt;http://history.princeton.edu&gt; .
</pre>

And if the same subject is repeated, but with different predicates,
one may use semicolons as in the example:

_N3 Syntactic Sugar: Semicolons_
<pre>
&lt;http://www.princeton.edu&gt; geo:lat "40.35" ; geo:long "-74.66" .
</pre>

N3 has a few other abbreviations as well.  The common predicate
<tt>rdf:type</tt> can be abbreviated simply as <tt>a</tt> (as in "is a").

[Turtle](http://www.ilrt.bris.ac.uk/discovery/2004/01/turtle/)
and NTriples are just like N3, but each with fewer of N3's syntactic
complexities.

Sometimes RDF/XML makes the information easier for humans to read,
but most of the time it just makes it more difficult to see exactly
what RDF statements are contained in there.  It's the statements
that count in the end, not the structure of the XML.

> See also: The authoratative definitions of [N3 
> syntax](http://www.w3.org/2000/10/swap/Primer.html) and [RDF/XML syntax](http://www.w3.org/TR/rdf-syntax-grammar/).

<a name="Distributed Information"> </a>

## Distributed Information

So far, RDF might seem unnecessarily complex.  For all of the examples
so far, the information could be written in plain XML just the same.
Taking a single document in isolation,
this is true.  But RDF excels when every document is thought
of as part of the bigger picture, part of a huge graph of
knowledge spread throughout the Internet.  In this "Semantic
Web," every RDF statement posted somewhere on the net contributes
a small piece to the greater body of knowledge.

But, not every piece of knowledge in the Semantic Web is
automatically interpretable.  Applications have to be told what
to do with particular predicates, or else the RDF is just URI salad.
In the next section, I tackle this further.

<a name="Choosing the Right Predicates"> </a>

### Choosing the Right Predicates

As you've seen so far, one can use RDF to model any type of knowledge
without having to use any centrally approved notions.  If no one
has coined a URI for something you want to describe, you can create 
your own URI for it.  This goes for not just subjects and objects but 
predicates as well.  In the examples above, I made up all of the URIs, 
and it's still perfectly valid RDF.

The trouble is that if I make up all of my own URIs, my RDF document
has no meaning to anyone else unless I explain what each URI is intended
to denote or mean. Two RDF documents with no URIs in common have no information 
that can be interrelated.  But, two documents that have some URIs in 
common are talking about some of the same things.  Someone else might 
want to publish information about Law &amp; Order.  By using the same URI for
the television show in the two documents, RDF tools will be able to recognize
that the two documents are describing the same thing.

Here is an example of using RDF to describe books.  Let's say, 
hypothetically, that the Library of Congress posted an RDF list of books 
and Amazon.com did the same.

_RDF from the Library of Congress (hypothetical)_
<pre>
&lt;urn:isbn:0143034650&gt;  dc:title  "Free Culture : The Nature and Future of Creativity"
&lt;urn:isbn:0613917472&gt;  dc:title  "Code and Other Laws of Cyberspace"
&lt;urn:isbn:B00005U7WO&gt;  dc:title  "The Future of Ideas"
</pre>

_RDF from Amazon.com (hypothetical)_
<pre>
&lt;urn:isbn:0143034650&gt;  amazon:price  "$15.00" .
&lt;urn:isbn:0613917472&gt;  amazon:price  "$26.35" .
&lt;urn:isbn:B00005U7WO&gt;  amazon:price  "$9.95" .
</pre>

The URIs for the books (<tt>urn:isbn:...</tt>) 
are what tie the two files together.  An RDF application using these 
files would be able to report that "The Future of Ideas" is "$9.95" at 
Amazon because both the title and the price are related to a common 
resource, denoted by the URI <tt>urn:isbn:B00005U7WO</tt>.  If the files did not 
have the same URI for that book, nothing would indicate that the titles 
went with particular prices.

It's also important to use the same predicates others are using when 
the predicates you want already exist.  This allows existing 
applications to make use of your information, without needing to be 
modified by developers to recognize your own URIs. For instance, if you're 
describing documents, you should use the existing [Dublin Core (DC)](http://dublincore.org/documents/dces/) title and 
description predicates so RDF applications that already use those 
predicates will be able to extract the titles and descriptions from your 
data.  If you're describing people, use existing [Friend of a Friend (FOAF)](http://www.foaf-project.org/) predicates so that 
FOAF-based applications will be able to take advantage of the 
information.

I used <tt>dc:title</tt> in my books example above because the predicate already 
existed to convey the information I wanted to put into the files: a 
relation between a book and its title.  No standard predicate exists for 
giving the Amazon.com price of a book, so I made one up.  (Remember 
<tt>amazon:price</tt> is an abbreviated form of a full URI, but I've left out the 
declaration of <tt>amazon:</tt> for the sake of brevity.)

Be careful of respecting the meanings of existing predicates, though.  
RDF applications expect the <tt>dc:title</tt> predicate to relate a document to 
its title.  You wouldn't want to use it to relate a telephone to its 
phone number, for instance, because existing RDF applications that get
a hold of your data will use it in ways that simply won't make sense.

<a name="Meshing Information: A Real-World Example"> </a>

### Meshing Information: A Real-World Example

Most of the time in today's world you get information from one place.  
For instance, a single database might contain the information for an 
entire product line.  There isn't much on the web in the way of 
distributed information because it's usually hard to put information 
together from multiple sources, each of which may have its own data 
format and conventions.

Here's a scenario where distributed information makes a lot of sense: a 
database of products from multiple vendors and reviews of those products 
from multiple reviewers.  No one vendor is going to want to be 
responsible for maintaining a central database for this project, 
especially since it will contain information for competing products and 
negative reviews.  Likewise, no one reviewer may have the resources to 
keep such a database up to date.  How can this become a reality?

RDF is particularly suited for this project.  Each vendor and reviewer 
will publish a file in RDF on their own websites.  The vendors will 
choose URIs for their products, and the reviewers will use those URIs 
when composing their reviews.  Vendors don't need to agree on a common 
naming scheme for products, and reviewers aren't tied to a 
vendor-controlled data format.  RDF allows the vendors and reviewers to 
agree on what they need to agree on, without forcing anyone to use one 
particular vocabulary.

_Vendor 1_
<pre>
vendor1:productX	dc:title	"Cool-O-Matic"
vendor1:productX	retail:price	"$50.75"
vendor1:productX	vendor1:partno	"TTK583"
vendor1:productY	dc:title	"Fluffertron"
vendor1:productY	retail:price	"$26.50"
vendor1:productY	vendor1:partno	"AAL132"
</pre>

_Vendor 2_
<pre>
vendor2:product1	dc:title	"Can Closer"
vendor2:product1	retail:price	"$28.11"
vendor2:product1	vendor2:warranty_code	"None."
vendor2:product2	dc:title	"Dust Unbuster"
vendor2:product2	retail:price	"$33.21"
vendor2:product2	vendor2:warranty_code	"X12"
</pre>

_Reviewer 1_
<pre>
vendor1:productX	dc:description	"This product is good buy!" .
</pre>

_Reviewer 2_
<pre>
vendor2:product2  dc:description  "Who needs something to unbust dust? 
                                  A dust buster would be a better idea."
vendor2:product2  review:rating   review:Excellent
</pre>

It's an open question just how an application will retrieve these files, 
but I'll put that aside.  Once an application has these files, it has 
enough information to relate products to reviews to prices, and even to 
vendor-specific information like <tt>vendor1:partno</tt> and 
<tt>vendor2:warranty_code</tt>.  What you should take away from this example is 
how unconstraining RDF is, while still allowing applications to 
immediately be able to relate information together.

And, RDF applications don't need to know about the nature of the data in 
these files to be able to make use of it.  If an application already knows what 
the <tt>dc:title</tt> and <tt>dc:description</tt> predicates are for,
and nothing else, then it is at least able to present the titles and 
reviews of the four products.  Note that the presence of predicates
the application doesn't understand, like <tt>review:rating</tt>, doesn't
impact the application at all.  It can simply ignore it without worrying
that it has misunderstood the rest of the data.

In addition, the vendors and reviewers did not have to agree on much to 
make this happen.  They had to agree to use RDF, but they didn't have to 
agree on any specific data format or even on specific URIs.  It helps 
that they agreed on URIs for indicating titles and prices, although even 
that wasn't strictly necessary.  But, crucially, they didn't have to 
enumerate everything any vendor would want to include about their 
products.  When a vendor needed something that wasn't already agreed on 
(product numbers and warranty codes), they were able to create a new 
predicate without disrupting any existing systems.  Likewise, the 
reviewers aren't tied to a vendor-controlled vocabulary.  Reviewers were 
free to add their own relations, such as a ratings, to their RDF files.

Another way to look at this from the standpoint of interoperability.  
Vendor 1's format is entirely interoperable with anyone else's format, 
even though Vendor 1 didn't hash out a common format with anyone.  When 
someone comes along and wants to be interoperable with Vendor 1's 
information, they don't need a new format, they just need to choose the 
right subjects, predicates, and objects.

Is this any better than XML?  I'll take a look at that later on.

<a name="Comparing RDF with XML"> </a>

## Comparing RDF with XML

Earlier I showed how RDF could be used to create a 
decentralized database for product information and reviews.  Here is how 
a similar system would be accomplished using non-RDF XML.

First, though, I'll look at how a single vendor would approach this 
alone.  The vendor, if it was so inclined, might publish an XML file 
with a node for each product, and within that a node for its name and 
some vendor-specific information.

_Vendor 1's XML File_
<pre>
&lt;products&gt;
  &lt;product title="Cool-O-Matic"&gt;
    &lt;price&gt;50.75&lt;/price&gt;
    &lt;partno&gt;TTK583&lt;/partno&gt;
    ...
</pre>

What can be done with this file?  An application to display this 
information would have to be specifically programmed to know that 
<tt>&lt;product&gt;</tt> nodes are for products, with titles in the <tt>title</tt>
attribute, etc.  And, if a reviewer wanted to post a review XML file, 
the only way to relate reviews to products would be by name.  Two 
vendors might have products with the same name, so vendors would have to 
use IDs of some sort to keep their products separate.

The first problem arises.  Vendors will need to come together to 
establish a product ID system so that IDs are unique within the local ID 
space of this vendor consortium.  RDF solves this problem by requiring 
that all IDs be globally unique, and by using URIs for IDs, allowing 
individuals to create IDs in a local space that they control.

_Vendor 1's XML File - Revision 1_
<pre>
&lt;products&gt;
  &lt;product title="Cool-O-Matic" id="vendor1_product1"&gt;
    &lt;price&gt;50.75&lt;/price&gt;
    &lt;partno&gt;TTK583&lt;/partno&gt;
    ...
</pre>

_Reviewer 1's XML File_
<pre>
&lt;reviews&gt;
  &lt;review product-title="Cool-O-Matic" id="reviewer1_review1"
	productid="vendor1_product1"&gt;
    &lt;description&gt;This product is just too cool.&lt;/description&gt;
    ...
</pre>

With IDs in the XML files, reviewers will be able to identify 
products in their review files, but applications still won't be able to 
relate products to reviews.  IDs aren't enough.  The applications 
themselves have to be told where to find the IDs.  In Vendor 1's file, 
it's in the product node's <tt>id</tt> attribute.  In the review files, 
the same attribute is to give an identifier to each review.  To connect 
the review to the product, another attribute is used.  Even if the 
vendors and reviewers agreed where to put the IDs, the application still 
needs to know where it is.  RDF solves this problem by making everything 
a global ID (except literals), so basically anything the RDF application sees is 
an ID that means something.

The vendors and reviewers next have to decide what constitutes a 
valid product or review XML file, and how the nodes of these files 
should be interpreted by software.  If these files are defined by a DTD 
or Schema, the files will not be extensible.  Before adding anything new 
into these files, such as vendor-specific information, all of the 
vendors and reviewers will need to agree to a DTD or Schema change.

On the other hand, the vendors and reviewers can go without a DTD or Schema.
There are no rules for what elements go where, which provides the
flexibility that the vendors and reviewers need.  But, then there must be
some guide to what the elements of the XML files mean, and thus a
central authority for deciding these things.  Unless, the vendors and
reviewers use namespaces to allow them to develop their own
vocabularies.

I could go on, but you should see now that XML isn't particularly suited for 
distributed, extensible information _unless_ that XML looks a lot
like RDF!

<a name="RDF about RDF"> </a>

## RDF about RDF

So far I've shown how RDF can be used to describe the relationships
between entities in the world.  RDF can be used at a higher level, too,
to describe RDF predicates and classes of resources.  Ontologies, schemas,
and vocabularies, which all mean roughly the same thing, are RDF
information about... other RDF information.

RDF ontologies play a vaguely similar role as XML Document Type 
Definitions and XML Schema.  But they are as different as they are the 
same.  DTDs and XML Schema specify what constitutes a valid document.  
They don't indicate how a document should be interpreted, and they only 
restrict the set of elements that can be used in any given file.  RDF 
ontologies, which are themselves written in RDF, provide relations 
between higher-level things, entirely for the purpose of indicating to 
applications how some information should be interpreted.  RDF 
ontologies also don't restrict at all which predicates are valid where.  
Any statement is valid anywhere, as before.

[RDF Schema](http://www.w3.org/TR/rdf-schema/) (RDFS)
and [Web Ontology Language](http://www.w3.org/TR/owl-features/) (OWL) define
a few _classes_ and predicates that are, merely by convention,
used to provide higher-level descriptions of data.

> The classes and predicates below are some of the standard tools that
> are available to you when you write ontologies.  For some examples
> of complete ontologies, including the standard RDF, RDFS, and OWL
> ontologies, see [SchemaWeb](http://www.schemaweb.info/).

<a name="RDF Schema"> </a>

### RDF Schema (RDFS)

RDF Schema (RDFS) introduces the notion of a class.  A class is a 
type of thing.  For instance, you and I are members of the class Person.  
Computers are members of the class Machine.  Person and Machine are 
classes.  That is to say, they are themselves members of the type Class.  The first 
higher-level predicate is the <tt>rdf:type</tt> predicate. (<tt>rdf</tt> 
is the usual namespace abbreviation for 
<tt>http://www.w3.org/1999/02/22-rdf-syntax-ns#</tt>.)  It relates an 
entity to another entity that denotes the class of the entity. The 
purpose of this predicate is to indicate what kind of thing a resource 
is. But, as with anything else in RDF, the choice of class is either by 
convention or arbitrary.

To add class information into the vendor files from a few sections ago,
a vendor would simply add this:

_Adding Type Information_
<pre>
vendor1:productX	rdf:type	general:Product
</pre>

As with choosing predicates, it's helpful to choose URIs for classes
that are used by others.  Agreement among different parties for classes,
and for the other things in this section, is very important.

One interesting class is <tt>rdf:Property</tt>.  Any entity
used as a predicate is an <tt>rdf:Property</tt>.  Thus, from the
examples above, we can conclude:

_The rdf:Property class_
<pre>
&lt;http://example.org/own&gt; rdf:type  rdf:Property
dc:subject               rdf:type  rdf:Property
amazon:price             rdf:type  rdf:Property
</pre>

To be explicit, one might include these statements in an RDF
ontology describing the predicates used in the data.

Other RDFS predicates are used to provide even more information about
predicates.  The <tt>rdfs:domain</tt> and <tt>rdfs:range</tt> predicates
relate a predicate to the class of resources that can
serve as the subject or object of the predicate, respectively.  Here's an example:

_Domain and Range_
<pre>
vendor2:warranty_code	rdfs:domain	general:Product
vendor2:warranty_code	rdfs:range	rdfs:Literal
</pre>

These statements say that the subjects of <tt>vendor2:warranty_code</tt>
are things typed as <tt>general:Product</tt> and the objects of this
predicate are literals (raw text).  That's true.  Recall:

_warranty_code_
<pre>
vendor2:product1  vendor2:warranty_code  "None."
</pre>

<tt>vendor2:product1</tt> is a product, and <tt>"None."</tt> is a 
literal value.

Specifying domains and ranges for predicates
serves two purposes.  First, it allows applications to make inferences
from statements about the types of things.  If it sees something that is
the subject of <tt>vendor2:warranty_code</tt>, it can infer that it is
a <tt>general:Product</tt>.  Second, these specifications serve
as documentation of a vocabulary for people.  The RDF itself is used
to indicate how predicates should be used.

Two RDFS predicates are used to give relations between classes
and predicates.  The <tt>rdfs:subClassOf</tt> relation indicates
that one class is a sub-class of another.  For instance, the class
Mammal is a sub-class of the class Animal.  Anything true of
the Animal class is also true of the Mammal class, and applications
are able to make such inferences once this predicate is present.  The
<tt>rdfs:subPropertyOf</tt> is similar, but for predicates.  For
example, the friend predicate is a sub-property of the knows
predicate.  Any friend is someone you know.

> [RDF Schema](http://www.w3.org/TR/rdf-schema/) details
> the semantics of these properties.

<a name="Web Ontology Language"> </a>

### Web Ontology Language (OWL)

Web Ontology Language (OWL) defines more classes that let RDF authors
define more of the meaning of their predicates within RDF.  Four
classes of predicates defined by OWL include:
<tt>owl:SymmetricProperty</tt>, <tt>owl:TransitiveProperty</tt>,
<tt>owl:FunctionalProperty</tt>, and <tt>owl:InverseFunctionalProperty</tt>.
(The OWL namespace is <tt>http://www.w3.org/2002/07/owl#</tt>.)
Each of these classes is <tt>rdf:subClassOf rdf:Property</tt>.

Applications can use these classes, by convention, to make inferences 
about data. You would use these classes in an ontology like this:

_Defining amazon:price_
<pre>
amazon:price	rdf:type	owl:FunctionalProperty
</pre>

Because these classes are defined in the OWL ontology as being sub-classes
of <tt>rdf:Property</tt>, applications can infer the following:

_Defining amazon:price_
<pre>
amazon:price	rdf:type	rdf:Property
</pre>

That's the same statement as earlier.  So, when you use a sub-class
in place of the 'parent' class, you're being strictly more informative.
Anything the application knew before it still knows, and it knows more
because the sub-class is more specific.

OWL symmetric properties tell applications that the
following inference is valid.  If the application sees the statement
S P O, and if P is typed as a symmetric property, then O P S is also
true.  For instance, we think of the has-friend relation are being symmetric.
If you're my friend (ME HAS_FRIEND YOU), I'm your friend (YOU HAS_FRIEND ME).

OWL transitive properties work like this.  If the application sees 
the statements X P Y and Y P Z, and if P is typed as a transitive 
property, then X P Z is also true.  <tt>rdfs:subClassOf</tt> is a transitive 
relation.  If Mammal is a sub-class of Animal and Animal is a sub-class 
of Organism, then Mammal is a sub-class of Organism.

OWL functional and inverse-functional properties indicate how many 
times a property can be used for a given subject or object. A functional 
property is one that has just one value for any particular subject.  An 
example is the <tt>hasBirthday</tt> relation between a person and his or 
her birthday.  Everyone has just one birthday, so for any given subject 
(person), there can be just one object (birthday).  But, the 
<tt>owns</tt> relation between an owner and ownee is not functional.  
Anyone can own more than one thing.

Inverse functional properties do the same in reverse.  For any 
object, there is only one subject for a particular inverse functional 
property.  The ISBN relation is inverse functional.  For any ISBN, 
there is only one book that has that ISBN.

Functional and inverse-functional properties can be used by
applications to infer things like two entities denote the same
thing.  For instance, take the following input:

_Inverse-Functional Properties_
<pre>
ex:isbn  rdf:type  owl:InverseFunctionalProperty
book:a   ex:isbn   "12345-67890"
book:b   ex:isbn   "12345-67890"
</pre>

If this data is consistent, then the fact that <tt>ex:isbn</tt> is
marked as an inverse-functional property lets the application conclude
<tt>book:a</tt> and <tt>book:b</tt> denote the very same book.  Recall
that two names can refer to the same thing.

> [Web Ontology Language](http://www.w3.org/TR/owl-features/) 
> defines the semantics of these predicates and classes.

<a name="Closing Remarks"> </a>

## Closing Remarks

There is a lot more to say about RDF.  Two recommended steps from here
	are to look into RDF inferencing (with [cwm](http://www.w3.org/2000/10/swap/doc/cwm.html)) and RDF querying
	(with [SPARQL](http://www.w3.org/TR/rdf-sparql-query/)).

Other good starting points are linked [from here](http://dannyayers.com/archives/2005/10/03/semantic-web-starting-points/).

If you want to program with RDF, grab [one
	of the many toolkits available](http://www.wiwiss.fu-berlin.de/suhl/bizer/toolkits/index.htm) in many languages.

Many thanks to everyone on the semantic-web W3 mail list
	who provided feedback on the initial post of this article!
