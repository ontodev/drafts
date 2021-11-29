# Building OBO Ontologies: Future

## A Vision for a Web-Based Future

1. GitHub: make greater use of API, Actions
2. OWL: RDFTab, Gizmos; Wiring, LDTab
3. templates: COGS, VALVE; Sprocket
4. automation: DROID


### 1. GitHub

GitHub is web-based, and we already take advantage of many of its features.
We propose better use of branch-based development,
and of centralized permissions management,
including GitHub Apps.

Our experience with SourceForge and Google Code makes us wary
of comitting too deeply to any single platform.
The decentralized nature of git already provides advantages over Subversion.
While GitHub is the market leader,
it has several major competitors
with comparable feature sets.
GitLab is a competitor which touts an open source core
and free self-hosting.

Although we make wide use of GitHub,
we are always careful to keep our options open.


### 2. OWL

OWL is a complex standard.
OWLAPI has been the only complete implementation of it.
[TODO: how to explain Horned OWL?]
[TODO: how to introduce OwlReady, Funowl?]
OWLAPI is written in Java,
and thus is available to programming languages and tools
with access to the Java Virtual Machine.

OWLAPI represents ontologies as objects in memory.
Loading an ontology can require significant amounts of memory.
Large but logically simple ontologies
such as OBO's translation of the NCBI Taxonomy
and the Protein Ontology,
can require 12, 16, or more gigabytes of memory,
and many minutes to load or save.
The lack of a disk-based storage option for OWLAPI
limits the size of ontologies that can be handled on a give machine,
and thus the deployment scenarios available.

A wider range of OBO-format libraries are available,
supporting a wider range of programming languages and platforms,
but OBO-format is much more limited than OWL
in the logical axioms it can express.

Since OWL has an RDF serialization,
RDF triplestores can be used to store OWL ontologies,
and SPARQL can be used to query them.
The Ontobee ontology browser is an example of such a system.
But complex OWL axioms are represented in RDF
using nested blank-node structures,
which makes it difficult to view OWL using SPARQL,
and even more difficult to edit it.

Graph databases such as Neo4j are an alternative to RDF triplestores,
but face the same problems
of view and editing deeply nested graph structures
representing complex OWL axioms.

When viewing or editing an ontology in Protege,
two operations are by far the most common.
First is browsing and selecting a term from the subclass hierarchy
(often on the left-hand side of the Protege window),
displayed with labels from some predicate
such as `rdfs:label`.
Second is displaying all the annotations and axioms for the selected term
(often on the right-hand side of the Protege window).
Working with a RDF representation of the ontology,
SPARQL makes the first task fairly straightforward,
but the second tasks is quite difficult in the general case.
Since OWL axioms can be arbitrarily complex,
you need to recursively "chase" blank nodes to an unknown depth.
Users also expect the resulting subgraph
to be translated into a human-readable form
such as Manchester syntax.

Related to the complexity of working with OWL in RDF,
we have had two major problems using triplestores for ontology editing.
The first is difficulty optimizing triplestore performance.
The second is deployment limitations,
since there are few options for an "embedded" triplestore.

It's natural to contrast RDF triplestores with relational databases.
Relational database systems in general,
which are practically synonymous with SQL,
are very mature technology.
They provide a wide range of options for optimization,
including various datatypes and indexes.
There is also a wide range of relational database options
for both "server" and "embedded" use cases.
Common open source choices include PostgreSQL for a database server,
and SQLite for an embedded server.
For ontology build tasks,
embedded databases such as SQLite are very convenient:
we can start from the source (text) files,
populate the database,
perform fast query and update operations,
perhaps save the results back to a text file,
and then discard the database by deleting one file.

#### 2.1 RDFTab

RDFTab is a command-line tool for reading an RDFXML file into a SQLite database.
It is written in Rust and relies on oxigraph rio
to do the hard work of converting RDFXML to triples.
The key innovation of RDFTab
is that we modified rio to signal
when it has finished reading a top-level XML element
(i.e. a child of the root element).
Top-level elements usually represent ontology terms,
and this signal helps RDFTab to group the triples into "stanzas".
A "stanza" is the set of all the triples
associated with a top-level subject
(usually an ontology term),
including the nested blank node structures
that represent complex OWL logical axioms,
and also OWL annotation axioms.
We name the stanza after the top-level subject.
The result is a table with columns for:
stanza, subject, predicate, object, value, datatype, and language.

The upshot is that it is easy to collect all the triples
associated with an ontology term
(the second common task discussed above).
It is also relatively easy to collect the subclass hierarchy
using SQLite's WITH RECURSIVE feature.

#### 2.2 Gizmos

Gizmos is a Python library that builds on RDFTab
to provide common ontology viewing and editing operations.
Given a SQL database with RDF triples as generated by RDFTab,
Gizmos provides functionality for:

- extract: extracting a subset of terms from an ontology using the MIREOT method
- export: generating a tablular representation
- tree: generating HTML pages for an ontology browser
- search: generating JSON and HTML for an ontology seach box

Although Gizmos is not nearly as full-featured as ROBOT,
the extract and export functionality it provides is much faster,
and uses just a fraction of the memory that ROBOT (i.e. OWLAPI) does.
The tree and search functionality in Gizmos is not supported by ROBOT,
but also uses a fraction of the memory.

#### 2.3 Wiring

The combination of RDFTab and Gizmos provides a useful ontology browser
with a minimal memory footprint
suitable to many deployment scenarios.
However it does not solve the problem of editing complex OWL axioms.
The heart of this problem is blank nodes.
Two representations of the same RDF graph
will have their blank nodes labelled differently,
and finding a mapping between the blank nodes
is a variation on the labelled graph isomorphism problem,
which does not have a known general solution in polynomial time.

While RDFTab uses a "thin triples" approach,
where each triple is represented by a single row in the table,
Wiring adopts a "thick triples" approach,
where blank node structures are "folded"
into a nested datastructure
that removes blank nodes.
We choose to represent the nested datastructure in JSON,
because it is both familiar and well supported in SQL databases.
We build the JSON from a few pieces.

1. as in RDFTab, we represent IRIs with CURIEs when possible, and use `<>` wrapped IRIs when not possible
2. we represent an RDF object with a JSON object: `{"object": "foo", "datatype": "xsd:string"}`
3. we represented a set of RDF objects with a sorted JSON array: `[{"object": "foo", "datatype": "xsd:string"}, {"object": "bar", "datatype": "xsd:string"}]`
4. we represent a set of predicate-object pairs
   as JSON object with predicates as keys
   and JSON arrays (RDF object sets) as values:
   `{"rdfs:label": [{"object": "foo", "datatype": "xsd:string"}]}`
5. treating RDF reification and OWL annotations as essentially the same,
   we use the same set of predicate-object pairs
   with an additional "meta" key:
   `{"rdfs:comment": [{"object": "foo", "datatype": "xsd:string", "meta": "owl:Annotation"}]}`
6. we handle nested stuctures:
   `{"rdf:anon": [{"object": {"rdfs:label": [{"object": "foo", "datatype": "xsd:string"}], "datatype": "RDFJSON"}}]}`

In the RDFTab table schema, either the 'object' or 'value' column must be null,
and either the 'datatype' or 'language' columns (or both) must be null.
This results in a sparse table, which can cause some problems.
In Wiring we decided to use a single 'object' column to represent
both IRIs and literal values.
We "overload" the 'datatype' column
to include language codes starting with `@`,
and also keywords such as "IRI" and "Blank"
to indicate that the object column contains an IRI or blank node (respectively).
We also add support for RDF named graphs.
With thick triples, the stanza column is no longer needed.
The upshot is that we use these columns:
graph, subject, predicate, object, datatype, annotation.

The result is that we have one row for each "top-level" triple,
where the graph, subject, and predicate are CURIEs.
Then we have several cases for the object:

- literal: the 'object' column contains the lexical content, and the 'datatype' column contains the datatype or the language tag
- IRI: the 'object' column contains the CURIE or IRI, and the 'datatype' column contains a keyword "IRI"
- simple blank node
- blank node structure: in the case of a nested blank node structure
  such as an RDF list or OWL logical axiom,
  we "collapse" the nested blank nodes
  into a single JSON object
  and store that in the 'object' column
  with 'datatype' set to "RDFJSON".
  [TODO: the "RDFJSON" name is already used]

It is still possible to have top-level blank nodes as subjects,
and to have "orphan" blank nodes as objects
that do not occur as subjects elsewhere in the graph.

We use this same technique to handle RDF reification and OWL annotations.
When the subject triple is <S, P, O>,
we use a JSON structure in the 'annotation' column
of the row for that target triple.

The upshot is a table with very few blank nodes.
There is no longer a need for stanzas.
We can query the 'subject' column for the subject we care about
and get rows for all the thick triples about it.
We can easily convert the JSON objects
into Turtle or Manchester syntax as appropriate.
This is generally more efficient to read,
but the key benefit is that it is much easier
to edit the thick triples representation,
since most edits will consist of changes to a single row.
It is also much easier to look for differences between RDF graphs
since blank nodes have been largely eliminated.

The JSON representation of RDF
is useful for storing nested data
in a single cell of a SQL database.
JSON is easy to use from any programming language,
but we admit that the nested JSON structures
are not a very convenient way to work with complex OWL axioms.

Wiring also provides a translation from RDFJSON
to "OWL Functional S-Expressions" syntax,
which is essentially OWL Functional syntax
represented as nested JSON arrays of strings.
So `ObjectSomeValuesFrom(ex:part-of, ex:bar)`
becomes `["ObjectSomeValuesFrom","ex:part-of","ex:bar"]`.
The advantage is that this syntax is "predigested" into JSON
and relatively easy to work with in any programming language
without an additional OWL Functional Syntax parser.

Wiring also provides parsing from and rendering to Manchester syntax,
both as strings and as HTML with links.
This is the only implementation of Manchester syntax outside of OWLAPI,
as far as we know.

In short, the Wiring library provides tools
for working with several representations of RDF,
including two new JSON representations.
These JSON representations let us collapse nested blank node structures into JSON,
and store and query them as "thick triples" in a SQL database.
This improves on the RDFTab/Gizmos approach
and makes it much easier to diff and edit ontologies
without using OWLAPI.


### 2.4 LDTab

LDTab is the successor to RDFTab.
While RDFTab uses oxigraph/rio and the thin triples representation,
LDTab uses Wiring and the thick triples representation.
LDTab is a command-line tool for loading ontologies into a SQL database.

[TODO: I'm still not sure how many Gizmos features will go in LDTab.]



## 3. Templates

OBO ontologies make increasing use of templates, which we encourage.
We propose three improved uses of tables for building ontologies:

1. use tables for imports
2. use Wiring and LDTab for translating templates to RDF/OWL
3. use better table editing and validation tools

### 3.1 Imports

As discussed above, OBI uses OntoFox configuration files
to configure our imports.
Many ODK projects extract IRIs for required imports from OWL files.
Other projects use tables to specify the terms to be imported.
Gizmos' `extract` operation includes new "import" functionality.
We propose building on this first version
to define a better standard table format
for specifying which terms to import, from where, and how.

### 3.2 Wiring and LDTab

The two main template systems use in OBO are DOS-DP and ROBOT.
Both build on OWLAPI.
As discussed above, Wiring and LDTab let us work with RDF and OWL without OWLAPI,
which gives us more options in terms of platforms and deployment scenarios.
Many lessons have been learned from DOS-DP and ROBOT templates,
and perhaps it is time to rethink our template tools.

Crucially, changing the template *tools*
should not require major changes to the *content* of template tables.

### 3.3 Better Table Tools

The ontology projects we are discussing use version control,
and so store tables as directories of TSV or CSV files.
It's convenient to edit these tables in spreadsheets such as Excel and Google Sheets.
In the previous discussion we have seen the benefits of SQL databases,
and these will be particularly useful for web-based editing of tables.

Tables are ubiquitous in biology, science, and everywhere.
The uses we make of tables in building ontologies
are not particularly special.
We have discussed general problems and opportunities for tables elsewhere:

<https://github.com/ontodev/drafts/tree/main/tables>

Our proposal is to use these better tools for ontologies as well.


## 4. Automation

Our proposal for automation is to add a web-based layer on top of existing tools.
Existing tools include:
GNU Make for defining tasks and dependencies;
ROBOT for ontology operations,
or their equivalents with Gizmos and LDTab;
scripts in Python and other programming languages;
GitHub for version control and GitHub Actions for running tasks;
Docker and ODK for packaging and isolation.

The new layer we propose is DROID.
DROID is a web-based interface for working with
(1) a build system,
managed by (2) a version control system,
optionally (3) running in a container.
The current version of DROID is designed to work with
(1) GNU Make, (2) GitHub, and (3) Docker.
Our goal is to make these systems accessible to a wider community of project contributors,
by exposing a curated set of functionality that is customized for each project.

DROID differs from Continuous Testing/Integration solutions
such as Jenkins or GitHub Actions
because DROID allows users to modify a working copy of a branch
and run tasks on an ad-hoc basis before committing changes,
optionally in a Docker container dedicated to the branch.
DROID differs from Web/Cloud IDEs
because users are limited to a specified set of files and tasks.

DROID can operate in either local or server mode.
When DROID is configured to run in local mode,
all GitHub authentication is performed on behalf of a single GitHub user account
that is identified via that user's personal access token.
When configured to run in server mode,
DROID will allow multiple distinct GitHub users
to be logged into the system simultaneously,
using GitHub App authentication to authenticate users.

In addition to running Make tasks,
DROID can also run CGI scripts
which can use Wiring, LDTab, and the better table tools discussed above
to view and edit tables and ontologies.


## Conclusions

Each of these tools is web-based
and they are all designed to work well together.
DROID talks to GitHub to authenticate users,
execute version control operations,
and create Pull Requests.
DROID uses Docker to run ODK
and ODK uses make, ROBOT, DOS-DP, and other tools.
DROID runs a curated set of tasks
from Makefiles we are already using.
Wiring gives us more options for how and where to use RDF and OWL.
LDTab gives us options to use RDF and OWL in SQL.
The better table tools let use move easily
between TSVs in version control, spreadsheets, and SQL databases,
and also validation and web-based editing.

These tools, like ROBOT, also follow a design pattern
of providing *both* libraries and tools.
The tools can be used by themselves,
but the libraries include the majority of the functionality,
and the libraries can be integrated into other software
such as custom tools.
More than this, these tools also provide *interfaces*
in the form of strong conventions on bog-standard JSON data structures.
These interfaces can be implemented by virtually any tool on any platform,
perhaps just implementing one part of the larger system
in some specialized way.
It should be relatively easy to write a Python script
that can write thick triples
or work with the better table tools.

These tools and workflows build on what has been working in the OBO community.
Each tool is largely orthogonal,
and can be adopted independently of each other,
although they work even better together.
Projects can adopt these pieces one-by-one, as they see fit.
When these tools are replaced by something better,
projects can drop them one-by-one in favour of their successors,
without undue restrictions and entanglements.

All these tools are open source.
They can be used as part of a web-based system,
but they can also be used locally.
They are open to extension.

These tools are also designed to be lightweight and scalable,
allowing larger ontologies to be built faster.

