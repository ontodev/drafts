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


