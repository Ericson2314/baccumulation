# Multiplicative infinitesimals

> This goes with [Multiplicative Calculus](./multiplicative-calculus.md), but where as that is mostly summarizing others work, this is original as far as I know, and so I split it out.

[Infinitesimals](https://en.wikipedia.org/wiki/Infinitesimal) are liked, despite their formal rigor (in most settings), are liked in some settings, like informally solving differential equations, and other applied tasks.
(It is interesting to ask why, how they can be made formal, and other questions, but I will refrain from doing so here.)
They are, "additive", in a few key ways:

1. They are near 0, which is the additive identity, and "not a multiplicative number"

2. They correspond to subtraction inside limits.

3. They thus suffice for regular additive calculus, but not the multiplicative calculus linked above.

The first one is self-evident enough, but the other two need some explaining.

For the first, let's start by expanding (quasi-)Leibniz notation for the derivation into the conventional limit definition.
```math
\frac {d f(x)} {d x} = \lim_{a \to x} \frac {f(a) - f(x)} {a - x}
```
Note that every $d E$, where $E$ is a [metavariable](https://en.wikipedia.org/wiki/metavariable), becomes $E[x \mapsto a] - E$,
i.e. we subtract $E$ from "almost $E$", but replacing $x$ with $a$ in the first term.
(Exactly formalizing this is probably harder, so I won't attempt it.)
This is the subtraction I was referring to.

The corresponding definitions in multiplicative calculus sometimes use this subtraction, but also use *quotients* instead of differences.
For that, normal infinitesimals do not help, and this is why they are not sufficient.

But suppose we had something for the $E[x \mapsto a] / E$ quotient; let's call it $qE$?
That would suffice.
Now, we can hardly define this $q$ syntax with the analog what was just a hand-wave for the normal $d$ syntax, but I can do so a different way.

In my writing about [elasticity](../economics-math.md#Elasticity), I wrote about "multiplicative perturbations".
I also concluded with pointing out the $d\ln x$ in the Wikipedia article.
If the $d$ "makes the expression very close to 0"; with the logarithm, that would make the $x$ close to 1.
The $q$ which we are trying to define also does that.
It is always the case (no limit needed for $a \to x$ needed for this to be true) that
```math
\ln a - \ln x = \ln \frac a x
```
On these grounds, let's *declare* (as an axiom) that
```math
d \ln x = \ln q x
```
from that, we can also define $q$ outright:
```math
q x = \exp(\ln q x) = \exp(d \ln x)
```
Note how this is the same sort of relationship we had with the multiplicative derivative and integral,
something in the form $m = \exp \circ a \circ \exp^{-1}$,
where the multiplicative version ($m$) is equal to the conventional additive version ($a$), composed with an $\exp$ after and $\exp^{-1}$ before.

With $q$ now so defined, we can start deriving some things.
First, as warm-up, let's find the $\exp$ counter part to our original axiom:
```math
d \ln x = \ln q x
```
substitute $\exp(x)$ for $x$:
```math
d \ln \exp(x) = \ln q \exp(x)
```
cancel out on the left:
```math
d x = \ln q \exp(x)
```
apply $\exp$ to both sides:
```math
\exp(d x) = q \exp(x)
```

More interestingly, we can also introduce Leibnitz-style notation for the concepts we've covered elsewhere:

- [Elasticity](../economics-math.md#Elasticity):
  ```math
  \epsilon = \frac {d \ln y} {d \ln x} = \frac {\ln q y} {\ln q x} = \log_{q x} {q y}
  ```

- The multiplicative derivative:
  ```math
  \sqrt[d x] {q y}
  ```

The latter nicely works for the [fundamental theorem of calculus](https://en.wikipedia.org/wiki/Fundamental_theorem_of_calculus):

```math
{\huge \mathscr{P}} \left( \sqrt[d x] {q f(x)} \right)^{d x}
```
```math
{\huge \mathscr{P}} q f
```
```math
f
```

Food for thought!
