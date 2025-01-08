# Multiplicative infinitesimals

## Introduction

> This goes with [Multiplicative Calculus](./multiplicative-calculus.md), but where as that is mostly summarizing others work, this is original as far as I know, and so I split it out.

[Infinitesimals](https://en.wikipedia.org/wiki/Infinitesimal) are liked, despite their formal rigor (in most settings), are liked in some settings, like informally solving differential equations, and other applied tasks.
(It is interesting to ask why, how they can be made formal, and other questions, but I will refrain from doing so here.)
They are, "additive", in a few key ways:

1. They are near 0, which is the additive identity, and "not a multiplicative number"

2. They correspond to subtraction inside limits.

3. They thus suffice for regular additive calculus, but not the multiplicative calculus linked above.

For the first, the intuition for a regular infinitesimal is that is a very small number, smaller than any non-zero real number.
If we multiply a real number by an infinitesimal, we always get another infinitesimal, never a real number, just like when we multiply any real number by 0, we always get zero.
0 and infinitesimals are "not multiplicative numbers" like the non-zero real numbers, because they represent this "point of no return".

For the second, let's start by expanding (quasi-)Leibniz notation for the derivation into the conventional limit definition.
```math
\frac {d f(x)} {d x} = \lim_{a \to x} \frac {f(a) - f(x)} {a - x}
```
Note that every $d E$, where $E$ is a [metavariable](https://en.wikipedia.org/wiki/metavariable), becomes $E[x \mapsto a] - E$,
i.e. we subtract $E$ from "almost $E$", but replacing $x$ with $a$ in the first term.
(Exactly formalizing this is probably harder, so I won't attempt it.)
This is the subtraction I was referring to.

The corresponding definitions in multiplicative calculus sometimes use this subtraction, but also use *quotients* instead of differences of expressions involving the limit variable and where its going.
In other words, They have terms like $E[x \mapsto a] / E$, not just terms like $E[x \mapsto a] - E$.
For the former terms, normal infinitesimals won't work: whereas the differences go to zero in the limit, these go to one.
Normal infinitesimals are thus not sufficient for a Leibnitz-style multiplicative calculus (without limits); we'll need something else.

But suppose we had something for the $E[x \mapsto a] / E$ quotient?
This would not be a very small number, but it would be very close to $1$, the way a normal infinitesimal is very close to $0$.
$1$ is the multiplicative identity just as $0$ is the additive identity; this is a good sign we are on the way to an analogous notion.
Without knowing quite what we have yet, lets bestow our notion with some syntax: let's call the infinitesimal-like thing corresponding to $E[x \mapsto a] / E$ $qE$.
Now, we can hardly define this $q$ syntax with the analog what was just a hand-wave for the normal $d$ syntax, but I can do so a different way.

In my writing about [elasticity](../economics-math.md#Elasticity), I wrote about "multiplicative perturbations";
If a regular infinitesimal is an "additive perturbation" --- think of the difference as first being slightly displaced, and then returning to almost but not quite the same position, this would be a "multiplicative" one, would it not?
I also concluded with pointing out the $d\ln x$ in one of the Wikipedia articles for elasticity.
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

## Usage

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
flip:
```math
q \exp(x) = \exp(d x)
```
Now we have equations for both how we can pull-out a $\ln$ over a $d$, converting it a $q$, and how we can pull out an $\exp$ over a $q$, converting it to a $d$.

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
{\huge \mathscr{P}} q f(x)
```
```math
f
```

Food for thought!

## Models

I checked out [Nonstandard analysis](https://en.wikipedia.org/wiki/Nonstandard_analysis) and [Hyperreal number](https://en.wikipedia.org/wiki/Hyperreal_number) on Wikipedia, which I had been meaning to do for a while, to see whether these multiplicative infinitesimals are already "handled" by it.
I think they are.
Nonstandard analysis has a notion of [*halo*](https://en.wikipedia.org/wiki/Monad_(nonstandard_analysis)).
The halo around $0$ is just the infinitesimals.
The halo around $1$ should be the multiplicative infinitesimals.

I think follows from the definition of $q$ we gave above:
```math
q x = \exp(d \ln x)
```
then use the nonstandard analysis definition of $d$:
```math
q x = \exp\left(\frac 1 x d x\right)
```
and then pull out the $d x$:
```math
q x = \exp\left(\frac 1 x\right)^{d x}
```
That is not easy to read, but I am pretty sure that since $\mathop{\text{st}}(d x) = 0$, then for all $x \ne 0$:
```math
\begin{align}
\mathop{\text{st}}(q x)
&= \mathop{\text{st}}\left(\exp\left(\frac 1 x\right)^{d x}\right) \\
&= \exp\left(\frac 1 x\right)^0 \\
&= 1
\end{align}
```
