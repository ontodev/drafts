# Building OBO Ontologies: Present

## OBO Status Quo

The status quo for OBO projects in 2021 is
a GitHub repository
that contains a mixture of OWL files
and TSV or CSV files for templates,
and a Makefile defining a range of tasks
for building, testing, and releasing the project.
Potential contributors need four main sets of skills and tools

1. version control
2. OWL
3. templates
4. automation

### 1. Version Control

Contributors must be able to check out a working copy of the project,
switch to a new development branch,
make their changes,
commit their changes to the branch,
push the branch to GitHub,
and open a pull request.
It is also very helpful if the contributor can run automation tasks
before committing.

Distributed version control systems such as git are complex,
but they are widely used for software development
and many related fields.
There is extensive documentation and training material on the Web.

GitHub also provides their own documentation
and the free GitHub Desktop,
which many users prefer to the `git` command-line tool.

GitHub also provides various options to manage and restrict PRs,
integration with third party continuous integration tools,
and their own GitHub Actions system
for executing a wide variety of tasks
triggered by various version control operations.

### 2. OWL

Contributors must view and edit OWL files.
Protege Desktop is by far the most common tool for this in the OBO community,
although alternatives such as Top Braid exist.

It is possible to build an ontology entirely with templates.
This reduces the need for an ontology editor,
but an ontology viewer is still indispensable,
and Protege usually fills this role.


### 3. Templates

Template files are usually stored as TSV or CSV text files,
which work well with (text-based) version control systems.
Ontology contributors usually edit these tables using spreadsheet software,
such as Microsoft Excel.
For the most part, this works,
but there are several common problems,
such as inconsistent line-ending characters,
cell quoting and escaping conventions,
special characters such as `'`.

### 4. Automation

Key automation tasks include:

- fetching import dependencies
- building import subsets
- converting templates to OWL modules
- merging OWL files
- reasoning
- automated testing
- releases

Many OBO projects use GNU Make to define and execute these tasks.
Many of the tasks use ROBOT,
while others use standard Unix coreutils
and custom Python scripts.

Many OBO projects use ODK to simplify installation of tools
and execution of the Makefile.
ODK also standardizes build tasks
and the locations of project files.

### Limitations

The key limitation to all the systems described here as the status quo
is that they require desktop applications.
This makes it difficult to contribute to ontology projects
using mobile devices, such as increasingly powerful table devices.
Desktop-centric workflows also provide fewer opportunities
for close collaboration than web-based workflow can,
and increased support costs related to software installation and maintenance.

Software development more generally includes powerful trends
towards web-based development,
such as cloud IDEs and pervasive use of the web technology stack.

Many of the tools discussed above do have web-based alternatives.

1. GitHub: very usable with the website alone
2. OWL: there are ontology browsers and Web Protege
3. templates: there are web-based spreadsheets such as Google Sheets
4. automation: provided by GitHub Actions and other tools

Ontology browsers are not ontology editors.
Web Protege does support ontology editing,
but it does not integrate into the workflows we are discussing,
involving version control, templates, and automation.
[TODO: Is this true?]
Google Sheets is very good for editing tables,
but the tables we are discussing are stored as TSV files
in the repository.
It can be tricky to synchronize a Google Sheet with TSV files.
More importantly, we want to impose many constraints on our templates,
which can be hard to express using validation rules in Google Sheets.
Our automated testing tools report problems,
but users can have difficulty understanding and acting on those messages
unless they are effectively integrated into the Google Sheet.
Finally, while command-line tools and workflow automation
allow a virtually unlimited range
of operations and transformations of an ontology project,
most ontology contributors should only need to perform
a very limited subset of these operations.
Ideally they should have easy access to a curated list of automation tasks.

In short, the status quo for developing OBO ontologies
is largely limited to desktop applications.
While there are web-based alternatives
to many of the individual tasks that ontology contributors must perform,
those alternatives do not combine into a coherent whole.

