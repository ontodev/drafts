# Table Types

We work with tabular data in several ways:

- TSV files stored in git
- spreadsheets: Excel, Google Sheets
- SQL databases
- data structures in programming languages: JSON

These data structures could be anything,
but let's take JSON as our lowest common denominator.

For much of our work
TSV files in git is the primary representation.
The key advantages are:

- version control
- keeping data side-by-side with code
- text-based workflows, often building on Unix coreutils or CLI tools

Spreadsheets are great.
SQL databases are great.
But if we want to version control our data,
TSV in git is the clear winner.

TSV is a very lax format.
It allows us to express "bad data".
That's a blessing and a curse.
It's a blessing because the world is full of
exceptions and missing information.
It's a curse because it's hard to work with "bad data".
It's really convenient to work with data
that has strict types.
SQL can be pretty strict,
which has a lot of nice advantages.
Spreadsheet carry a lot more information than TSV
but are also pretty lax.

We need to work with bad data.
We need to work with data efficiently.
So let's think about how to do both at the same time.

## Table Types

It should be easy to move back and forth
between TSV files, spreadsheets, SQL databases, and JSON.
Unfortunately, it is not.
Why not?

We can treat a text file as a sequence of strings (lines).
Let's adopt Java-style notation: `sequence<string>`.
We can treat a TSV file as a `sequence<sequence<string>>`.
We'll talk about header rows, column names, primary keys, etc. in a minute.
If we want to have multiple TSV files,
we can keep them together in a directory.
Then we have something like `map<string,sequence<sequence<string>>>`.

A row in a relational database (i.e. SQL)
is a "relation" from columns to values.
For our purposes, I think this is close enough
to a Python `dict` or a JSON `object` or a `map` in many other languages.
The column names are strings.
The values can be of several types.
So each row is a `map<string,type>`.
We often thing of SQL tables as ordered,
but strictly they are a set allowing duplicates.
These are sometimes called `bag` or `multiset`.
So our type for a SQL table is `bag<map<string,type>>`.
A SQL database can have multiple tables,
so our type is `map<string,bag<map<string,type>>>`.
Nice.

The cells in a spreadsheet are much more complex
than the "cells" in a TSV file or SQL table.
They can have type information, formatting, formulae, etc.
The tables of a spreadsheet do not have strong schemas.
The columns are named A, B, C, etc.
The values in a column may have different types.
So our type for a single sheet is something like `sequence<sequence<cell>>`.
The sheets usually have names,
in which case our type for a spreadsheet is something like
`map<string,sequence<sequence<cell>>>`.

## Grids

So we have three table types to reconcile:

- TSV `sequence<sequence<string>>`
- SQL `bag<map<string,type>>>`.
- Spreadsheet `sequence<sequence<string>>`

If we define a default sort
or use something like ROWNUM or ROWID,
then we can treat our SQL type as `sequence<map<string,type>>>`.
If we are very careful about our column order,
then we can make it `sequence<sequence<type>>`.
We'll talk about header rows in the next section.

This is essentially a two-dimensional array.
Let's call it a "grid".

## Cells

We are left to reconcile the three cell types:

- TSV: `string`
- SQL: `type` (including NULL)
- Spreadsheet: `cell`

Note that the SQL type must be the same
for all the cells in a column.
SQL defaults to NULLable types,
but you can specify NOT NULL.

We can't change these types,
but we can define a mapping between them.

A spreadsheet cell is the richest of these.
It can have lots of attributes, including:

- value (typed)
- string value
- datatype
- formatting
- comment
- formula
- conditional formatting rule

My proposal is to represent this as a JSON object
with several fields:

- value: required string
- valid: boolean
- datatype: optional string
- nulltype: optional foreign key
- format: optional foreign key
- messages: optional array of objects

## Spreadsheet Cell

The JSON is closest to a spreadsheet cell.
We won't try to capture all the information,
but we can capture the parts we care about.

## TSV Cell

The trick here is to "interpret" the TSV string
using a variety of rules.
The rules need to be specified somewhere.
Some of them can be baked in to the translation code,
but most of the rules should specified in other tables --
meta-tables.

TODO

## SQL Cells

This is the part I struggled with the most.
The solution I came up with is "shadow columns":
for each (typed) column containing the (typed) values,
we also create a column containing the JSON object.
So column A might have type INT (nullable),
but "shadow" column A_ contains JSON
which can capture the rest of the information we need.

In SQLite the column would have type TEXT
and we would use the various JSON functions to work with it,
In Postgres we would use a proper JSON type for the column.

The nice thing about this approach
is that it's easy to run "normal" SQL queries on the good data,
and not too hard to get the bad data.

## NULLs

In a TSV file we normally represent NULL with an empty string.
Let's take that as the default case
and try to handle it efficiently.
In SQL, let's represent that as NULL in the column
and also NULL and in shadow column.

## Headers

A TSV file without a header is very confusing.
Let's require our TSV files to have a header row.
Likewise, a table in a spreadsheet without a header is confusing.
Let's require our tables to have 

If we require headers
then there's all sorts of tables that we can't represent.
But we also get a lot of advantages.

If we require headers then out table type becomes
`sequence<map<string,cell>>`.


## Primary Keys

If we require primary keys, then out table type becomes
`map<string,<map<string,cell>>>`.

