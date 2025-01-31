# Uniqueness and combinatorial topology

## Uniqueness

"Unique constraints" are used to indicate that a given projection from the relation must identify elements of the relation uniquely.
That is, given relation $A' \in \mathscr{P}A$, and projection $p$, $p$ *projects uniquely from* $A'$ if:

```math
\forall a, b \in A' .\, a = b \leftarrow p(a) = p(b)
```

A projection $a : A \to C$ *factors* into $c \cdot b$ if their exists two other projections $b : A \to B$ and $c : B \to C$ such that $a = b \cdot c$.

Theorem:

> Given $A' \in \mathscr{P}A$,
> If $a$ projects uniquely from $A'$, and $a$ factors into $b \cdot c$, then $c$ must also project uniquely from $A'$.

## SQL

In relational databases, unique constraints are commonly specified not as arbitrary functions, but as a subset of the columns of the table.

## Combinatorial topology refresher

(See [Wikipedia](https://en.wikipedia.org/wiki/Abstract_simplicial_complex) for more.)

Take any finite set $V$, called the vertex type.
A *simplex* on $V$, hereafter $S(V)$, is a finite non-empty set of $V$.
This is its type:

```math
S(V) = \sum_{s \in \mathscr{P}V} 0 \lt |s|
```

A *simplicial complex* on $V$ is a set of simplices closed under subset.
This is its type:

```math
C(V) = \sum_{c \in \mathscr{P}S(V)} \forall s, t \in c.\, t \subset s \wedge s \in c \Rightarrow t \in c
```

A *facet* is maximal simplex in a complex;
precisely, it is one that is not a subset of in any other simplex in the complex.

## Cocomplices

Let me make up a new concept called a *simplicial **co**complex*.
This would be a set of simplices closed under *super*set.
This is its type:

```math
C(V) = \sum_{c \in \mathscr{P}S(V)} \forall s, t \in c.\, t \supset s \wedge s \in c \Rightarrow t \in c
```

> Note the flipped $\subset$ to  $\supset$

A *facet* of a cocomplex will likely be defined is as a *minimal* simplex in the cocomplex
precisely, it is one that no other simplex in the complex is a subset of.

Dual to every complex is a cocomplex.

## The connection

Let us return to the "conventional" SQL projections specified by column set.
When we project from a table using such a projection, we have a new table (new relation, not talking about materializing) that just contains those columns.
If a projection factors through two "conventional" projections, that means we first restrict ourselves to a subset of columns, and then restrict ourselves to a subset of that subset.

Relations are sets, so the identity projection is always unique (on the relation in question).
If some other non-trivial conventional projection is unique, then any conventional projection that projects those columns and any more is also unique, since we can thence project the original unique columns from that coarser projection.
(This is by the theorem above.)

Let as now think more syntactically, and define a "unique column set" as a set of column (names) that identifies a unique projection.
Per the above, any column set that contains a unique set is itself unique.
This is not a novel insight, but a standard one is in the "relational model"; see [superkey](https://en.wikipedia.org/wiki/Superkey)
But it is also precisely the "closed under supersets" property, which means the unique columns sets form a simplicial cocomplex (with column names as vertices)!

[Candidate keys](https://en.wikipedia.org/wiki/Candidate_key) are the minimal column sets, where removing any column results in a projection that is no longer unique.
This is precisely the same minimality condition as that which characterizes a facet of a cocomplex --- remove any vertex and the set is no longer valid (in that cocomplex) simplex.

The column sets which are not required to contain unique values per row likewise form a regular simplicial complex.[^non-unique]
That is, every column set which is *not* superkey is simplex in such a complex, and every maximal non-superkey is a facet.

Note that while the complex and the co-complex are complements (relative the set of all possible simplexes, of which all complexes and co-complexes are subsets), that does *not* mean that the complement of simplex in one complex/co-complex is a simplex in its dual.
The complete complex (which is also a co-complex) contains every simplex, and thus the complement of every simplex also, and its dual is the empty complex/co-complex which contains no simplices.

[^non-unique]: I didn't simply call those "non-unique" column sets, or column sets which contain non-unique values, because they aren't actually *required* to contain duplicates, but merely *allowed* to.
  You can have two "John Smiths" in your database, but you are not required to.
