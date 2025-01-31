# Towards the next database query language

> [!NOTE]
> Everything here is a work in progress

I don't yet have a full vision for a new query language but here are some loosely connected notes.
Hopefully it will gel together over time.

The focus will be on reads / queries, and data structures / invariants.
Delete/updates are less interesting.
Migrations are interesting, but come after deciding what sorts of data structures we want.

The good thing about query languages is that it is basically just functional programming.
So there are lots of things we know that we should apply.
There is also Datalog, which is rightly venerated, and which I ought to consider, but I don't feel like getting into that yet.
So consider recursive queries, and various incremental stuff, a big giant "TODO" on this.

## Unrestricted data

In typical functional programming fashion, I'll start by trying to make more things first-class
--- "removing the weaknesses and restrictions that make additional features appear necessary", to quote the Scheme reference (as I usually like to do).

> Note that I am doing this *before* any attempt to formally model what I want.
> The hope is that actually be pushing on some "restrictions" and generalizing things, we'll something that is in fact easier to formalize / rigorously think about because there are fewer "wrong trees to bark up".
> By that I mean, restrictions sometimes are interesting things that yield interesting and worthwhile invariants --- contra the quote above.
> I am going to *assume* that isn't the case with SQL, and blast away the restrictions.

The place to start with that is the types.

### Ground types

Whatever you want, who cares! :)

### Tuples/records

SQL generally makes you name fields, and I like that.
Making these first-class is behind what standard SQL does, but it [already exists in PostgresSQL](https://www.postgresql.org/docs/current/rowtypes.html) like this:

```pgsql
CREATE TYPE inventory_item AS (
    name            text,
    supplier_id     integer,
    price           numeric
);

CREATE TYPE inventory_item AS (
    name            text,
    supplier_id     integer,
    price           numeric
);
```

So we aren't really breaking new ground here.

### Sum types

`NULL`-able is just a crude "Option" -- bad!
Don't stop there!

"Option" for whole tuples (rather than each column) is useful for proper one-sided outer joins:

```math
\mathrm{left\_join} : \forall A B.\, \mathrm{Set}(A) \to \mathrm{Set}(B) \to (A \to B \to \mathrm{Bool}) \to \mathrm{Set} (A \times \mathrm{Option}(B))
```
```math
\mathrm{right\_join} : \forall A B.\, \mathrm{Set}(A) \to \mathrm{Set}(B) \to (A \to B \to \mathrm{Bool}) \to \mathrm{Set} (\mathrm{Option}(A) \times B)
```

["These"](https://hackage.haskell.org/package/these/docs/Data-These.html#t:These) for full outer joins:

```math
\mathrm{full\_outer\_join} : \forall A B.\, \mathrm{Set}(A) \to \mathrm{Set}(B) \to (A \to B \to \mathrm{Bool}) \to \mathrm{Set} (\mathrm{These}(A,B))
```

### Set/Relation

This is where things get spicy!
Even with all of PostgreSQL's extensions, we don't distinguish between the type of the *contents* table (i.e. of a row), and the type of the table *itself*.
That is because, we cannot have a column whose type is itself a table --- a whole tables inside a single cell!
But I want for that to be allowed.

> For a bit of a gentle introduction of what data in this shape looks like, see [my hierarchical CSV proposal](./hierarchical-csv.md).

What is this useful for?
Well, for starters, it puts `GROUP BY` on much better footing.
See how [Haskell's standard library](https://hackage.haskell.org/package/base-4.20.0.1/docs/Data-List-NonEmpty.html#v:groupBy) does it (for non-empty lists), for example.
We would want something similar:

```math
\mathrm{group\_by} : \forall A B.\, \mathrm{Eq}(A) \Rightarrow (A \to B) \to \mathrm{Set}(A) \to \mathrm{Map}(B, \mathrm{Set}_{\ne\emptyset}(A)))
```

The idea is that that if we can project out a $B$ from an $A$, then we can convert a $Set B$ into a map of $B$ to every the non-empty-set of every $A$ that shares that same $B$.
The sets are non-empty because we only create groups "on demand", only for each distinct $A$ that we actually projected.

### Constraints

What is "Map"?
Well, recall that SQL databases typically support "unique constraints".
These are typically thought of as being "on tables", they should really be considered as part of types.
We'll need to get more complex later, but for now, we can make do with basic [refinement types](https://en.wikipedia.org/wiki/Refinement_type).

> Those constraints have some [interesting structure](./uniqueness-combinatorial-topology.md), but never mind that for now.

## Expression

```pgsql
SELECT x, q
FROM
  table_0 AS x,
  LATERAL (
    WITH
      y AS (SELECT field_1 FROM table_1 as z WHERE z.field_0 = x.field_0)
    ...
  ) AS q;
```

Could be translated to something like:

```lean
do
  x ← table_0
  q ← do
    let y := do
      z ← table_1
      guard (z == x.field_1)
    ...
  pure (x, q)
 ```

which could be rewritten as

```lean
do
  let checkpoint = do
    let x ← table_0
    let y := do -- note `let` not bind
      z ← table_1
      guard (z == x.field_1)
    pure (x, y)
  let (x, y) ← checkpoint
  q ← ...
  pure (x, q)
```

The idea is that, instead of computing `y` while computing `q`, per `x`,  compute `y` per `x`, as an explicit intermediate relation, and then join that to compute `q` for `(x, q)`.

The `do`-notion obscures this, but this rewrite follows from the $\mathrm{map} (f \circ  g) = \mathrm{map}(f) \circ  \mathrm{map}(g)$ functor law.

The rewrite is fairly simple, but `checkpoint` has the interesting type of $\mathrm{Map}(X, \mathrm{Set}(Y))$ (assuming we have $x : X$ and $y : Y$), which indicates that we convert the rewritten expression here back to SQL.

## Flat normal forms

### Splitting sum types

### Unnesting sets

## Capabilities
