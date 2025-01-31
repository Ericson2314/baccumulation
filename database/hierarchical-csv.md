# Hierarchical CSV

## Regular CSV

Suppose I have a simple database of IDs and names.
I can do this in JSON5 with:
```json5
[
  { "id": 0, "name": "Alice" },
  { "id": 1, "name": "Bob" },
]
```
simple enough.

But note how "id" and "name" must be repeated every row.
This is both cumbersome, yes, and also opens the door to various failure modes:
what happens if the keys are incomplete, misnamed, etc.?

Another alternative is this:
```json5
{
  "header": ["id", "name", ],
  "data": [
    [0, "Alice"],
    [1, "Bob"],
  ],
}
```
This works quite well:
with the *header* positions are assigned keys, and then in *data* we just specify information positionally, with an array of arrays.
The overall object is still "self-describing", but the self-description has been separated from the data proper.

This second use of JSON is the idea behind the (far predating JSON) venerable CSV format.
We can redo it in CSV very simply as follows:

```csv
id,name
0,Alice
1,Bob
```

## Hierarchical data

Say, however, we want to track people and their pets?

> The relational-db-knowing folks in the room are clamoring "use two tables!", but let's not go there.
> We haven't said what a "table" is, we shouldn't start now.

The data in "conventional" JSON5 usage looks like this:
```json5
[
  { "id": 0,
    "name": "Alice",
    "pets: [
      {
        "id": 0,
        "name": "Spot",
        "species": "dog",
      },
    ],
  },
  {
    "id": 1,
    "name": "Bob",
    "pets: [
      {
        "id": 0,
        "name": "Teacup",
        "species": "pig",
      },
      {
        "id": 1,
        "name": "Sassy",
        "species": "cat",
      },
    ],
  },
]
```

How can we skip the redundant field names in the same way?
Try this:

```json5
{
  "header": ["id", "name", ["pets", [ "id", "name, "species", ], ], ],
  "data": [
    [
      0,
      "Alice",
      [
        [0, "Spot", "dog"],
      ]
    ],
    [
      1,
      "Bob",
      [
        [0, "Teacup", "pig"],
        [1, "Sassy", "cat"],
      ]
    ],
  ],
}
```

This works on similar principles.
The header now has this type (in Typescript):
```typescript
type Header = (
  | string // name of regular or "atomic" field
  | [ // nested data
    string, // name of "nested data" field
    Header, // header of this "nested data" field
  ] // nested
)[];
```

> Note that every nested data field gets *two* `[..]`, one for the name, one for the sub-header.

As before, all of our data below is specified purely positionally.

## Extending CSV

CSV doesn't support it, but if we steal `[` and `]` syntax (those can be quoted to appear in strings), it can:

```hscv
id,name,[pets,[id,name,species]]
0,Alice,[
0,Spot,dog
]
1,Bob,[
0,Teacup,pig
1,Sassy,cat
]
```

Every row is on its own line, and `[` `]` indicates:

1. the header hierarchy, all one one line

2. the beginning and ending of each "run" of rows, where each run corresponding to a nested header

## Final thoughts

### HTML

The venerable HTML
[`rowspan`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/td#rowspan)
and
[`colspan`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/th#colspan)
attributes exist for the purpose of rending this sort of information.

For example, here is an HTML table for our running pets example:

<table border="1">
  <tr>
    <th rowspan="2">id</th>
    <th rowspan="2">name</th>
    <th colspan="3">pets</th>
  </tr>
  <tr>
    <th>id</th>
    <th>name</th>
    <th>species</th>
  </tr>
  <tr>
    <td rowspan="1">0</td>
    <td rowspan="1">Alice</td>
    <td>0</td>
    <td>Spot</td>
    <td>dog</td>
  </tr>
  <tr>
    <td rowspan="2">1</td>
    <td rowspan="2">Bob</td>
    <td>0</td>
    <td>Teacup</td>
    <td>pig</td>
  </tr>
  <tr>
    <td>1</td>
    <td>Sassy</td>
    <td>cat</td>
  </tr>
</table>

However, the design of these attributes is oriented around presentation more than the content itself.
The underlying hierarchical structure, like that if the hierarchical CSV example above, is not reproduced in the document node hierarchy, and instead rows have varying number of cells, with the numbers to `colspan` and `rowspan` patching things up.
The HTML system is more expressive, in that there HTML "table shapes" that do not correspond to any hierarchical CSV, but the semantic meaning of those table shapes would have some other semantic meaning, left to the reader to infer.

### Multiple tables

I wrote above

> The relational-db-knowing folks in the room are clamoring "use two tables!", but let's not go there.
> We haven't said what a "table" is, we shouldn't start now.

Well, to cut to the chase, the interesting thing is that this is [isomorphic to two tables with the right foreign-key structure](./flattening-a-schema.md#nested-containers).
Yes, that can be seen as reason to ban this in lieu of that, but I rather keep both ways of doing thing.
This has implications for [query language design](./query-language.md).
