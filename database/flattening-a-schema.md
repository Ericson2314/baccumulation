# Flattening a schema

## Basics

A simplified data structure for SQL schemas might look something like this:

```typescript
// Primitive types for column values. We don't actually care what these are
type GroundType = ...;

// Or could be something else
type Identifier = string;

type RowType = {
  columns: Map<Identifier, GroundType>
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
  tables: Map<Identifier, TableType>
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
>   person : Map<{ id: integer }, { name: string }>,
>   building : Map<{ id: integer }, { address: string }>,
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
>   table: Map<{ id: integer }, { foo: { x: string } } | { bar: { y: integer } }>;
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

There's simply no good solution here with how SQL currently works --- we need something new.
The new feature would be unique indices that *span multiple tables*.

## Nested containers

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
type PersonId = integer;

type Schema = {
  person: Map<
    { id: PersonId },
    {
      name: string,
    }>,
  pets: Map<
    { person: PersonId },
    Set<{ name: string }>
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
