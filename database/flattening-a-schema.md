# Flattening a schema

## Basics

A simplified data structure for SQL schemas might look something like this:

```typescript
// Primitive types for column values. We don't actually care what these are
type GroundType = ...;

// Or could be something else
type Identifier = string;

type RowType = {
  columns: Map<Identifier, GroundType>;
};

// Represents a table with a primary key row and other columns
//
// Primary key and other columns must use disjoint column names
type TableType = {
  primary_key: RowType;
  other_columns: RowType;
};

// Represents a foreign key constraint
type ForeignKey = {
  local_table: Identifier;
  foreign_table: Identifier;
  // Assign foreign table's primary key columns to local table's columns
  columns: Map<Identifier, Identifer>;
};

// Represents the entire schema
type Schema = {
  tables: Map<Identifier, TableType>;
  // Table and column names must be valid in `tables`.
  foreign_keys: ForeignKey[];
};
```

It is hopefully somewhat clear how one would translate certain a collection types (say also in TypeScript) to schemas.
Here is an example:

> **Example**:
>
> ```typescript
> { example_table: Set<{ id: integer }> }
> ```
>
> is isomorphic to
>
> ```typescript
> { example_table: Map<{ id: integer }, {}> }
> ```
>
> which is translated to
>
> ```typescript
> {
>   tables: new Map([
>     [
>       "example_table",
>       {
>         primary_key: {
>           columns: new Map([["id", "INTEGER"]]),
>         },
>         other_columns: {
>           columns: new Map(), // No other columns
>         },
>       },
>     ],
>   ]),
>   foreign_keys: [], // No foreign keys
> }
> ```

The thing to note is that we're starting from a type for the *whole* schema.
The typical "ORM" appraoch is to translate record types to table types: which conflates the type of a row in a table with the type of table itself.
This is sloppy, and makes it harder to talk about the more advanced cases we will later get to, so we will not do that.

> **Example**:
>
> ```typescript
> {
>   person: Map<{ id: integer }, { name: string }>;
>   building: Map<{ id: integer }, { address: string }>;
> }
> ```
>
> which is translated to
>
> ```typescript
> {
>   tables: new Map([
>     [
>       "person",
>       {
>         primary_key: {
>           columns: new Map([["id", "INTEGER"]]),
>         },
>         other_columns: {
>           columns: new Map([["name", "STRING"]]),
>         },
>       },
>     ],
>     [
>       "building",
>       {
>         primary_key: {
>           columns: new Map([["id", "INTEGER"]]),
>         },
>         other_columns: {
>           columns: new Map([["address", "STRING"]]),
>         },
>       },
>     ],
>   ]),
>   foreign_keys: [], // No foreign keys
> }
> ```

## Harder case: sum types

These types all themselves are in a form which corresponds to the type we gave for SQL schemas, so they are easy to translate.
But there are other TypeScript types which do not fit that form.

To start with, let's tackle *sum types* (tagged unions).

When the sum type is in the key, this is not too hard.
We can split the table into multiple until there are no sum types remaining:

> **Example**:
>
> ```typescript
> {
>   table: Map<
>     { foo: { x: string }} | { bar: { y: integer }},
>     { baz: string }
>   >;
> }
> ```
>
> is isomorphic to
>
> ```typescript
> {
>   table_foo: Map<{ foo: { x: string } }, { baz: string }>;
>   table_bar: Map<{ bar: { y: integer } }, { baz: string }>;
> }
> ```
>
> This second type longer contains any sum types, so we are ready to translate it to a schema:
>
> ```typescript
> {
>   tables: new Map([
>     [
>       "table_foo",
>       {
>         primary_key: {
>           columns: new Map([["x", "STRING"]]),
>         },
>         other_columns: {
>           columns: new Map([["baz", "STRING"]]),
>         },
>       },
>     ],
>     [
>       "table_bar",
>       {
>         primary_key: {
>           columns: new Map([["y", "INTEGER"]]),
>         },
>         other_columns: {
>           columns: new Map([["baz", "STRING"]]),
>         },
>       },
>     ],
>   ]),
>   foreign_keys: [], // No foreign keys
> }
> ```

What about if the sum type is in the value?
Let's attempt another example, but, watch out --- there will be a problem.

> **Incorrect Example**:
>
> ```typescript
> {
>   table: Map<
>     { id: integer },
>     { foo: { x: string } } | { bar: { y: integer } }
>   >;
> }
> ```
>
> Split it into two separate maps based on the possible value types:
>
> ```typescript
> {
>   table_foo: Map<{ id: integer }, { x: string }>;
>   table_bar: Map<{ id: integer }, { y: integer }>;
> }
> ```
>
> Once again, the second type longer contains any sum types, so we are ready to translate it to a schema:
>
> ```typescript
> {
>   tables: new Map([
>     [
>       "table_foo",
>       {
>         primary_key: {
>           columns: new Map([["id", "INTEGER"]]),
>         },
>         other_columns: {
>           columns: new Map([["x", "STRING"]]),
>         },
>       },
>     ],
>     [
>       "table_bar",
>       {
>         primary_key: {
>           columns: new Map([["id", "INTEGER"]]),
>         },
>         other_columns: {
>           columns: new Map([["y", "INTEGER"]]),
>         },
>       },
>     ],
>   ]),
>   foreign_keys: [], // No foreign keys
> }
> ```

OK --- don't scroll further!
Can you spot the problem?

----

The problem is in the first step, rewriting the TypeScript type so it has two maps.
The new type allows for a key to be repeated accross the two maps, and this breaks the desired isomorphism.
Converting from the first type to the second type is fine: we just partition the map in two, and then are done.
But converting the second type to the first type is partial, because we might have the same key in both maps.
we have to either throw one of the values away, or combine them somehow (which will not be injective in the general case, breaking the isomophism).

> *Example*
>
> ```typescript
> {
>   table_foo: new Map([[{ id: 42 }, { x: "pick me!" }]]),
>   table_bar: new Map([[{ id: 42 }, { y: 0xdeadbeef }]]),
> }
> ```
>
> This cannot be converted as:
>
> ```typescript
> {
>   table: new Map([[{ id: 42 }, { foo: { x: "pick me!" }}]]),
> }
> ```
>
> because we would lose the second row, and likewise cannot be converted as:
>
> ```typescript
> {
>   table: new Map([[{ id: 42 }, { bar: { y: 0xdeadbeef }}]]),
> }
> ```
>
> because we would lose the first row. And finally,
>
> ```typescript
> {
>   table: new Map([[
>     { id: 42 },
>     {
>       foo: { x: "pick me!" },
>       bar: { y: 0xdeadbeef },
>     }
>   ]]),
> }
> ```
>
> preserves both, but doesn't match the type!

### Multi-table unique constraints

There's simply no good[^bidirectional-constraint] solution here with how SQL currently works --- we need something new.
The new feature would be unique constraint that *span multiple tables*.

[^bidirectional-constraint]: With enough extra tables, bidirectional foreign keys, and [deferred constraints](https://www.postgresql.org/docs/current/sql-set-constraints.html) it is possible to solve this problem with some SQL databases.
  But that is real pain.

We can adjust our type for schemas to account for this:

```typescript
// Primitive types for column values. We don't actually care what these are
type GroundType = ...;

// Or could be something else
type Identifier = string;

type RowType = {
  columns: Map<Identifier, GroundType>;
};

// Represents a table with a primary key row and other columns
//
// Primary key and other columns must use disjoint column names
type TableType = {
  // New: No primary key vs other columns distinction
  columns: RowType;
};

// Represents a foreign key constraint
type ForeignKey = {
  local_table: Identifier;
  foreign_table: Identifier;
  // Assign foreign table's primary key columns to local table's columns
  columns: Map<Identifier, Identifer>;
};

// Represents a unique constraint
type Unique = {
  // The *nth* referenced column must be of the same type in each table.
  // The number of columns referenced (lenth of the list) must also be
  // the same.
  tables_and_keys: Map<Identifier, Identifier[]>;
}

// Represents the entire schema
type Schema = {
  tables: Map<Identifier, TableType>;
  // Table and column names must be valid in `tables`.
  foreign_keys: ForeignKey[];
  uniques: Unique[];
};
```

With this change, we can now redo our example from above:

> **Example**:
>
> ```typescript
> {
>   table: Map<
>     { id: integer },
>     { foo: { x: string } } | { bar: { y: integer } }
>   >;
> }
> ```
>
> Can become
>
> ```typescript
> {
>   tables: new Map([
>     [
>       "table_foo",
>       {
>         columns: {
>           columns: new Map([
>             ["id", "INTEGER"],
>             ["x", "STRING"],
>           ]),
>         },
>       },
>     ],
>     [
>       "table_bar",
>       {
>         columns: {
>           columns: new Map([
>             ["id", "INTEGER"],
>             ["y", "INTEGER"],
>           ]),
>         },
>       },
>     ],
>   ]),
>   foreign_keys: [], // No foreign keys
>   uniques: [
>     {
>       tables_and_keys: new Map([
>         ["table_foo", ["id"]],
>         ["table_bar", ["id"]],
>       ]),
>     },
>   ],
> }
> ```
>
> The multi-table unique index ensures that no `id` can occur more than once in *either* `table_foo` or `table_bar`.
> When we convert the original value to the database value, we split into two tables, and when we convert back and recombine the two tables, we know that for any `id` we'll either take a row from `table_foo` or from `table_bar`.

This is exacty the semantics we want.
The case before we could not implement, where we had a row from each table, there was no good answer on what to do.
Now, we know that case cannot happen thanks to the unique constraint.

#### Implementation

So does any database this?
Not to my knowledge.

Has anyone else even asked for the feature?
I found a thread on the PostgreSQL mailing list [starting with this message](https://www.postgresql.org/message-id/BANLkTineoT4gKz8vybDQJarVhtxSf-84-g%40mail.gmail.com).
But this discussion was more about workarounds than implementation.

The [docs on inheritence](https://www.postgresql.org/docs/current/ddl-inherit.html#DDL-INHERIT-CAVEATS) mention essential the same missing feature:

> If we declared `cities.name` to be `UNIQUE` or a `PRIMARY KEY`, this would not stop the `capitals` table from having rows with names duplicating rows in `cities`. And those duplicate rows would by default show up in queries from `cities`.
> In fact, by default `capitals` would have no unique constraint at all, and so could contain multiple rows with the same name. You could add a unique constraint to `capitals`, but this would not prevent duplication compared to `cities`.

Postgresql's "inheritence" feature can be thought of as some sugar for a "bidirectional" view: rules ensure that both queries and updates to the view are "fanned out" to the correct underlying table.
The rules system supports operations that are needed (by rewriting requests from the client), but it doesn't help with the problem of *data at rest*.
For that, we really need generalizations to indices, which are the "constructive" data structure underlying unique constraints.

## Nested containers

> [!NOTE]
> This section is a work in progess

```typescript
{
  person: Map<
    { id: integer },
    {
      name: string,
      pets: Set<{ name: string }>,
    },
  >
}
```

Start flattening.

```typescript
// Stand-in for future foreign key constraints
type PersonId = integer;

type Schema = {
  person: Map<
    { id: PersonId },
    { name: string }
  >,
  pets: Map<
    { person: PersonId },
    NonEmptySet<{ name: string }>
  >,
};
```

Two nestings.

```typescript
{
  person: Map<
    { id: integer },
    {
      name: string,
      pets: Set<{
        name: string
        pets: Set<{
          name: string
        }>,
      }>,
    },
  >
}
```

Yes, we're giving our pets pets!
(Nevermind what the pet snake will do with its own pet mouse...)

### Another view: sub-databases

## Conclusion
