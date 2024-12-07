# Towards the next database query language

## Unrestricted data

### Ground types

Whatever you want, who cares! :)

### Tuples/records

First class, like Postgresql.
(Names encouraged like SQL, but that is more superficial.)

### Sum types

Don't stop at `NULL`!

`Option` for whole tuples.
This is needed for one-sided outer joins:

```math
\mathrm{left\_join} : \forall A B. \mathrm{Set} A \to \mathrm{Set} B \to (A \to B \to \mathrm{Bool}) \to \mathrm{Set} (A \times \mathrm{Option} B)
```

`These` for full outer joins:

```math
\mathrm{full\_outer\_join} : \forall A B. \mathrm{Set} A \to \mathrm{Set} B \to (A \to B \to \mathrm{Bool}) \to \mathrm{Set} (\mathrm{These} A B)
```

## Flat normal forms

### Splitting sum types

### Unnesting sets

## Capabilities
