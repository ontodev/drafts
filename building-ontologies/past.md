# Building OBO Ontologies: Past

## History

The OBO Foundry was established circa ~2005 (Smith 2007 10.1038/nbt1346).
It brought together existing projects such as the Gene Ontology
and various model organism ontologies,
with the goal of building a suite of open, interoperable ontologies
to cover biology and biomedicine.

[TODO: Before OBO Edit?]

### OBO Format

OBO format is a text format for describing classes and their relations.
OBO Edit is a desktop application written in Java for viewing and editing OBO-format.
OBO format is relatively easy for humans to edit,
for simple programs to manipulate,
and is well-suited for version control.

[obo-format]: https://owlcollab.github.io/oboformat/doc/GO.format.obo-1_4.html
[obo-edit]: http://oboedit.org

### OWL Transition

The W3C developed the Web Ontology Language (OWL).
OWL builds on RDFS, which builds on RDF.
OWLAPI has been the only implementation of the OWL standard.
Protege (desktop) is the most widely used OWL viewer and editor
in the OBO community.

The OBO community adopted OWL,
to take advantage of Semantic Web tools such as OWLAPI and Protege,
and embrace interoperability with the wider Semantic Web.
A mapping from OBO format semantics into OWL was established.
OBO format support was added to OWLAPI in recent years,
making it straightforward to to edit OBO format files in Protege.

New OBO projects were encouraged to use OWL from the start.
Most established OBO projects continued to use OBO-format
and their existing workflows.

OBO format is deprecated, but still in wide use.
It offers several practical benefits.
OBO format is easier for humans to read and write
than RDF formats such as RDFXML and Turtle.
OBO format is also easier to read and write with software.
OWL is a complex standard, and
and OWLAPI has been the only library to fully implement it.
Since OWLAPI is written in Java,
its use is largely limited to the JVM platform.
OBO format is much simpler,
and libraries are available in several languages,
include Python and Rust.

OBO format has also been more amenable to version control.
Until ~2015, OWLAPI serialization to RDFXML was not "stable"
in the sense that reading and then writing an ontology
(without changing the content)
would result in an RDFXML file
with elements in a different order.

OBO Edit has been largely replaced by Protege.
However there are key features of OBO Edit
that are not supported by Protege.
The most important of these is the ability to view
both the subclass and partonomy relations
in the same tree.

### Open Source Ontologies

For many years, "OWL native" OBO project such as OBI
would operate as follows.

Over the years,
OBO projects have taken advantage
of a series of free open source software project hosting platforms.
Many OBO projects used Source Forge.
Google Code was popular for a short period,
before it was shut down.
GitHub has been the most popular option in recent years.
GitHub has several competitors,
such as Bitbucket and GitLab,
which are used by some OBO projects.
OBI used Source Forge using Subversion as the version control system,
then migrated to GitHub using git as the version control system.

In addition to hosting a version control repository
with authentication and authorization systems,
these services also provide issue management.
GitHub also provides many more features,
including 
Pull Requests,
continuous integration,
GitHub Actions,
and a powerful HTTP API.
OBO projects such as OBI
only made the most basic use of these systems
for version control and issue tracking.

The standard workflow for OBI at this time was:

1. email the OBI mailing list saying "OBI is locked"
2. edit `obi.owl` in Protege
3. commit changes to SVN master branch
4. email "Unlocked" or "Done editing"

With the move to GitHub
OBI took more advantage of branched editing.
However it was still difficult to merge branches
when all changes were made in a single `obi.owl` file.

To a limited extent,
"branches" of OBI development
were pursued in a small number of different OWL files,
also stored on the master branch of the version control system.
Importing and merging these branch files was often problematic.

### OWL Automation

The OBO community aims to create an orthogonal suite
of interoperable, open ontologies.
Reuse of terms from other OBO ontologies is crucial.

OBI has used the Ontofox service.
We store Ontofox configuration files in version control,
which include the sets of terms to import
and various configuration options.
Periodically we submit these files to the Ontofox web service,
which returns an OWL file with the terms to import.
We store these OWL files in version control,
and we use `owl:import` to import these OWL files into `obi-edit.owl`.

Protege has a steep learning curve,
but the key advantage of the OBI workflow described above
was that learning Protege was the key skill
required to make changes to OBI.
Very few features of Subversion were required:
mainly just (1) fetch the latest files from master,
and (2) commit changes to master.
The result was most of the people participating in OBI discussions
were also editing OBI with their changes.

The key disadvantage is every editor had their own approach
to writing logical axioms.
It would take several minutes to reason over OBI,
so editors often forgot to check their changes before committing them.
There were no tools to check this automatically.
It was common for incoherencies to be introduced
and committed to the version control system,
which could be difficult to track down.

OBI "releases" would undergo more thorough manual checking
to ensure consistency and an appropriate subclass hierarchy.
Creating these releases was painful.
A single release manager would often spend two days
debugging problems with `obi.owl`
and coordinating fixes with the editors who made those changes.
Because the process was slow and difficult,
months would often pass between releases,
making it even harder to debug and fix problems.

My first major contribution to the OBI project
was to write release automation tools.
Alan Ruttenberg had written some tools in Armed Bear Common Lisp,
which ran on the JVM and called the OWLAPI.
I rewrote and extended these in Java calling the OWLAPI.

In 2015 Chris Mungall and I started the ROBOT project.
ROBOT is a Java codebase consisting of two parts:
`robot-core` is a library of operations common to OBO projects,
which builds on OWLAPI (for OWL) and Apache Jena (for RDF and SPARQL);
`robot-command` is a command-line interface which calls
operations from `robot-core`.
The ROBOT CLI is usually called from a workflow system such as GNU Make.

By automating the OBI release process,
we reduced a two-day job to an hour or two.
Many other OBO projects adopted ROBOT to automate tasks.

The key disadvantage of this automation
is that it requires a wider range of skills.
Many OBI project members who were happy to edit in Protege
are not comfortable working in a terminal to execute automation tasks,
and cannot directly contribute to the development of those tools.

Two more disadvantages of OBO automation in practise
are that it requires a larger number of tools to install,
and that it tends to be Unix-specific:
it is relatively easy to use under Linux or macOS,
but more difficult to use under Windows.
This is largely due to reliance on GNU Make.
The Ontology Development Kit (ODK) provides a Docker image
which builds on Ubuntu Linux
and includes ROBOT, DOS-DP, and many other common tools.
ODK makes it easier to install and use our Unix-specific workflows
on all the major desktop platforms.


### Templates 

Building on the increased use of automation
was an increased use of templates for defining ontology terms.
OBO projects predominantly use DOS-DP or ROBOT templates,
although these were predated by several systems
including Populous and Ontodog.
While there are important differences among these systems,
the essential feature is that ontology developers use tables
rather than editing in Protege,
then some tool converts the tables to OWL.

There are several benefits to templates.
Perhaps the most important is that they enforce strong design patterns for terms,
which encourages consistency and reduces errors.
Templates also reduce the learning curve for ontology contributions,
since practically everyone is already familiar
with the basic functionality of spreadsheet software.


### Continuous Integration

Automation of ontology projects also facilitated automated testing.
Software can check for a wide range of common problems with ontologies,
including logical problems such as incoherence,
and stylistic problems such as required annotations, capitalization, and whitespace.

The ROBOT `report` command uses SPARQL to run a wide range of checks
common to OBO projects,
and also allows custom checks to be specified.

OBO projects using GitHub often require
that all commits to the `master` or `main` branch first be Pull Requests,
and that Pull Requests must pass the automated tests
before they are merged.
This helps to ensure that the `master` branch is always "green",
i.e. passes all automated tests.

OBO has gone further and created the OBO Dashboard,
which runs a range of automated checks for all OBO projects,
covering as many OBO Principles as possible,
displays the results in a table,
and provides instructions to fix common problems.



