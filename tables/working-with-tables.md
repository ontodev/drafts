# Trouble with Tables

## Outline

- need to handle "good data" efficiently
  - version control
  - large number of rows
  - nulls
  - types
  - indexes
  - linked tables
  - nested structures
- need to handle "bad data"
  - incomplete
  - inconsistent
  - exceptions
- efficient workflows
  - fast tools
  - quick iterations
  - cross-platform
- need to explain why data is considered good or bad
  - clear expression of rules, exceptions
    - tables about tables; bootstrapping problem
  - clear list of how rules were applied
- need to fix data
  - normalize: case, whitespace
  - suggest fix
    - automatically apply fixes
    - manually confirm fixes
  - near misses: did you mean?
  - mass replacement
- summaries
  - number of problems by table, column, type?
- change
  - update summaries
  - update data
    - mass replacement
    - schema changes
    - update defaults
- data integration
  - ontologies
  - databases
  - RDF, OWL, modelling

## Working with Tables

Now that we move freely back and forth
between TSVs, spreadsheets, and SQL,
have all our dreams come true?
Unfortunately, no.
We have a powerful and flexible system,
but we still have a long way to go
to make it as useful as possible.

1. handle good data efficiently
2. handle bad data effectively
3. explain why data is good or bad
4. fix bad data

The best we can usually hope for
is that *most* of the data in a given dataset is good:
syntactically correct,
semantically correct,
conforming to all our data types and rules.
We need to handle this good data efficiently.
Our SQL tables should enforce strong constraints,
such as datatypes, primary keys, and foreign keys.
Strong constraints will help with efficient indexes,
which will result in fast queries.

But there will always be "bad" data:
formatting problems,
required values that are missing,
sets of values that make no sense,
ambiguous NULLs, etc.
There's no limit on the number of ways our data can be bad,
so we can't hope to handle all of it optimally,
but we should be able to handle bad data *effectively*,
by keeping track of it
letting our users know what the problems are,
and how to fix those problems.

In the discussion above,
the '_meta' columns and '_conflict' table
give us the flexibility to handle bad data,
while allowing good data to have all the SQL constraints we desire.

In addition to being able to distinguish good data from bad data,
we also need to *explain* to our users *why* specific data are good or bad.
Users need to understand the validation rules
and also be confident in their understanding.
Ideally, most users should be able to modify validation rules
and write their own.
This is why we propose configuring validation rules with *tables*
rather than with scripts or database schemas.

Most validation systems use programming languages,
and output results in logs,
leaving users with the task of reading the logs
and figuring out where the problem is.
Spreadsheets do better when they highlight invalid cells
and support filtering for them.
This is the good example that we will follow.

Most validation systems also stop short at *identifying* the problem.
This is not good enough.
In most cases, it should also be possible
to suggest how the user can *fix* the problem.
In many cases it should be possible to *automatically* fix the problem,
either with the press of a button or silently.


### Datatypes

Different SQL databases support different datatypes.
SQLite supports relatively few "storage classes":
null, text, integer, real, blob.
These few storage classes are used to support
a slightly larger number of datatypes:
boolean (integer), date (text, real, or integer), JSON (text).
PostgreSQL supports dozens of datatypes.

No matter how many SQL datatypes are supported,
they will fall short of representing the semantics
of all but the simplest dataset.
While we may represent an ID as an integer
or a label as text,
integer and text do not capture our intended meaning of ID and label.

SQL lets us specify additional constraints in several ways.
We can enforce CHECKs on values,
to insist that IDs are seven digit integers,
or that labels must not begin or end with whitespace.
We can enforce uniqueness, primary keys, and foreign keys
on a single column or spanning multiple columns.
We will take advantage of these tools in our proposal.
The problem is that we want to express these constraints
in a language that is accessible to our users.
We want to capture the semantics of datatypes,
not just their syntactic structure.
In particular, we want to express these constraints using tables,
that users can read and edit alongside their tables of data.

We propose a 'datatype' table
where each row represents a datatype that we might enforce on a column.
The datatypes will be organized as a tree.
The root datatype is 'text',
accepting any string,
and it will be the ultimate ancestor of all other datatypes.
We use text as our root precisely because TSV cells
are represented by text.
All other datatypes will inherit from 'text'.
While we could specify a 'label' datatype as a direct child of 'text',
instead we choose to build a hierarchy of increasingly specific datatypes:

- line: text without newline characters
- trimmed_line: a line that does not begin or end with whitespace
- label: a trimmed_line

For many of our datatypes
we will specify a rule for matching or rejecting a value.
Most of these rules will build on regular expressions.
While regular expressions can be difficult to read and write in general,
by using a hierarchy we can break complicated rules into simple parts,
and thus use a series of simple regular expressions
that are easy to read and write
and form a coherent whole.

If a value should have datatype 'label',
we determine the list of ancestors
from the root 'text' to 'label' --
in this case: text, line, trimmed_line, label.
We check the value against each of these datatypes in order.
If the value fails any of these checks
then it is invalid.
We attach a message based on the datatype check that failed.

Every datatype except for the root 'text'
must have a parent datatype.
Every datatype must have a name, an error level, and a message.
Datatypes may also include a suggested fix.
For trimmed_line the suggestion might be the value
with leading and trailing whitespace removed

### Column Table

We also propose a 'column' table,
which will be used to generate SQL table schemas
and configure which datatypes apply to each column.
The columns of the column table must include:

- table
- column
- nulltype
- datatype
- structure

Other columns may be desirable, such as 'description'.

The combination of 'table' and 'column' will identify the target column.
The nulltype specifies which strings will be interpreted as NULLs.
The datatype column specifies which datatype from the 'datatype' table
applies to this column.
The 'structure' specifies additional constraints:
unique, primary key, foreign key, and more discussed below.

### Nulltype Validation

NULLs are an important and confusing part of SQL databases.


In TSV files, NULLs are most often represented by empty strings.
They can also be represented by the string "NULL",
or "NA", "na", "nd", etc.

The nulltype column specifies a datatype from the 'datatype' table.
If the specified nulltype matches (and all its ancestors)
then this value will be interpreted as a NULL.
We represent a cell with matched nulltype using this JSON:

```
{
  "value": nil,
  "raw": "",
  "nulltype": "empty"
}
```

If a nulltype is specified then the value is considered "optional".
If no nulltype is specified then the value considered "required".
The next step is to check the datatype.

### Datatype Validation



A successful datatype check is represented with this JSON:

```
{
  "value": "foo",
  "datatype": "label"
}
```

An unsuccessful datatype check is represented by this JSON:

```
{
  "value": "foo ",
  "datatype": "line",
  "messages": [
    {
      "rule": "datatype:trimmed_line",
      "level": "error",
      "message": "Must be a line without leading or trailing whitespace"
    }
  ]
}
```

With a lower logging level,
a successful datatype check could include more messages:

```
{
  "value": "foo",
  "datatype": "label",
  "messages": [
    {
      "rule": "datatype:text",
      "level": "debug",
      "message": "Must be text"
    },
    {
      "rule": "datatype:line",
      "level": "debug",
      "message": "Must be text without newlines"
    },
    {
      "rule": "datatype:trimmed_line",
      "level": "error",
      "message": "Must be a line without leading or trailing whitespace"
    },
    {
      "rule": "datatype:label",
      "level": "debug",
      "message": "Must be a trimmed line"
    }
  ]
}
```

These messages will help users understand
why a given datatype applies or does not apply to a given value.

The nulltype and datatype checks are focused on a single value at a time.


### Structure

TODO: call this 'constraint' column?

The structure column lets us specify constraints
that apply to full columns.

The 'primary' and 'unique' structures correspond directly to SQL constraints.
The 'primary' constraint specifies
that all the values in a column must be distinct.
Each table can have zero or one primary constraint,
but it can include one or more columns.
The 'unique' constraint also specifies
that all the values in a column must be distinct,
but a table can have zero or more unique constraints.

The 'from()' structure specifies a foreign key constraint.

The 

In general, when a structure constraint fails for a given row
this is a SQL conflict.
That row cannot be inserted into the primary table
and so must be inserted into the _conflict table.

### Rule Table

The nulltype and datatype constrain individual values.
The structure column constrains columns.
In order to specify constraints
that connect multiple columns in the same row
we propose the 'rule' table.
The 'rule' table requires these columns:

- table
- when column
- when condition
- then column
- then condition
- level
- message

The rule table builds on datatype functionality.
The 'when condition' and 'then condition'
are specified as in datatypes,
and apply to the 'when column' and 'then column' respectively.
It may also include a 'suggestion' column
which will function like datatype suggestions.

While when-then conditions are very simple
compared to the conditionals offered by most programming languages,
they are also simple to understand.

### Validate Rows

The validate_rows function requires:

- general configuration, including 'datatype', 'column', and 'rule' tables
- database read access
- a table name
- a list of rows, where each row is represented as a map from column name to string value

The order of validation is as follows:

- verify config
- find table name in config
- verify rows input is a list<map<string, string>>
- foreach row
  - foreach cell
    - lookup column
    - if column is not found then warn? TODO
    - check nulltype
      - if nulltype is not specified, check datatype
      - get list of ancestors for nulltype
      - check that all ancestors and nulltype match
      - if nulltype matches, set JSON and continue to next cell
    - check datatype
  - foreach rule
    - if when condition applies then check then condition
- check structure
  - foreach column
    - check unique/primary
      - check for unique values within rows
      - check for unique values against database
    - check foreign
    - check under

Note that foreign key constraints between tables
require that tables are validated in dependency order.

`validate_rows` returns a list of rows,
where each row is a map from column name to a JSON object
representing the validation result (as discussed above).

### Creating Tables

For each table of data, we create two SQL tables:
the "primary" table and the "_conflict" table.
For each column of the original table
we create a "value" column and a corresponding "_meta" column to hold JSON.
We use the datatype for each column
to set the SQL type of the "value" column.

In the primary table we use primary, unique, and foreign key constraints
from the 'structure' table.
In the _conflict table we do not specify these constraints.
Rows which would fail these constraints
and therefore cannot be inserted into the primary table
can still be inserted into the _conflict table.

### SQL Insert

Given the result of `validate_rows()`
there are two main steps to insert into SQL tables.

First, if the cell is valid,
then the value must be extracted from the JSON
and put into the value column.
If the cell is invalid, the value column will be NULL.
The remaining JSON will be inserted into the _meta column.

Second, rows with conflicts must be inserted into the _conflict table
while all others are inserted into the primary table.

### SQL Update

The update case is somewhat more complex than the insert case.

- need to address rows
  - primary key
  - rowid
- may need to move rows from primary table to _conflict table and vice versa

s## Conclusions

TODO



