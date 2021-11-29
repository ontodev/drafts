## The Virtues of Tables

Tables are ubiquitous.
Focusing on biology and biomedicine alone,
we see tables in journal articles,
supplemental tables of data,
tables of data output from instruments,
tables of data configuring software,
spreadsheets for data cleaning and analysis,
scientific databases,
to name just a few.

Tables come in many forms,
and we frequently move back and forth between these forms.
Tables used as input or output of software or an instrument
are often stored in text files with a delimited format
such as comma-separated values (CSV)
or tab-separated values (TSV).
Often multiple textual tables are related to each other
and stored in a directory together.
Spreadsheet programs such as Microsoft Excel
keep multiple tables together as "sheets"
stored in a single file with a complex format.
Relational database systems usually use
SQL (Structured Query Language)
to define sets of tables
with complex constraints and connections.

Each of these formats has its virtues and limitations.
Text files are ideal inputs and outputs for software
because they are so simple,
and also work well with version control systems.
Any number of tools can read and edit a TSV file,
but many common operations are inconvenient.

Spreadsheets make it easy to edit tables as tables,
and also offer powerful visualization tools
and a certain degree of programming with formulae.
But it can be difficult to get data in or out of spreadsheets
with a simple script,
and their large, rich, binary file formats
are not well suited to version control systems.
Certain operations that are simple in SQL
can be difficult or impossible in a spreadsheet.

SQL databases are powerful tools that can manage vast amounts of data,
and provide fine-grained tools for analyzing and optimizing performance.
Various software makes it easy to view and edit SQL tables like a spreadsheet,
but SQL tables do not store rich formatting such as fonts and colours,
and they enforce a single datatype on all the values in a column.
Setting up and maintaining a relational database server
can complex and sometimes requires special skills.

Moving back and forth between TSV, spreadsheets, and SQL
is both extremely common and also error-prone.
An empty string in a cell of a TSV file
may or may not mean the same thing as a NULL.
A spreadsheets names columns with letters,
while a SQL table enforces column names.

Ideally, we would have the best of all three:

1. directories of TSV files in version control
2. straightforward editing in a spreadsheet
3. efficient storage and query in SQL

How can we accomplish this?
