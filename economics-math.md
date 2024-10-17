# Fixing the Mathematics of Growth for Economics and Accounting

## Basic, no calculus

There's something wrong with economics and accounting math.
Not "wrong-answers" wrong, but bad pedagogy, and icky approximations.

It's easy to go after approximations — "No! I insist that you write this other formula that is actually correct" ...and far larger.
And indeed, if there was just a trade off between precision and bloat, I would not bother writing this.
Rather, and what really gets me excited, is that there is no such trade-off — we can be both more correct and just as (or more!) terse.
We just need to use the right abstractions.

Here's a starting point.
Consider a phrase like "2% growth";
you've seen it newspapers, in high school math doing loan interest, and maybe in econ or accounting classes too.
The meaning of the phase is "went from 100% to 102%" (over some period), adding another %2 on top.

But *adding* percents is, at best, a very dubious endeavor.
For example what is 2% growth twice?
Is it 4% growth?
Nope!
That neglects compounding, which is to say each percentage is in terms of the total up to the previous period.
You can only add "synchronous" percentages, and that is not a very common thing to do either.

> A quick not on method:
> the above paragraph is not formal, and also beyond the realm of regular [dimensional analysis](https://en.wikipedia.org/wiki/Dimensional_analysis).
> But we shouldn't dismiss it for lack of formality; the burden of proof should be on justifying correct formulae, not disputing incorrect ones ("guilty until proven innocent").
In type theory terms, we should be conservative and ban
$(+) \colon \mathrm{Percent} \to \mathrm{Percent} \to \mathrm{Percent}$,
and instead replace it with some more complex operation which takes a "proof of synchronicity", whatever that looks like.
> I don't know what sort proposition we'd need to prove, but the good thing is that, based on what fallows, I think we can side-step needing to figure this operation out entirely.

There is an alternative to addition which avoids this problem, however, and handles compounding correctly, for free.
That is multiplication.
Instead of doing (wrong!):

```math
a + 2\% a + 2\% a
```

we can do (right!):

```math
a \cdot (1 + 2\%) \cdot (1 + 2\%)
```

or more simply,

```math
a \cdot 102\% \cdot 102\%
```

And there we have a bit a [shibboleth](https://en.wikipedia.org/wiki/Shibboleth):
call it "102% growth", not "2% growth".
It might be too late to change English, but one can dream...

## Calculus

The world may (or may not be) discrete, but we use continuous math to explore intuitions and ideals for a reason.
In physics there is explicitly [continuum mechanics](https://en.wikipedia.org/wiki/Continuum_mechanics) to make this argument.
I'm surprised given Economics's infamous "physics envy" the phrase "continuum economics" isn't out there;
maybe that's because most/all theoretical neoclassical econ is "continuum economics"?

For this topic, the continuous counterpart we might call *continuously compounding growth*, after the standard term ["continuously compounding interest"](https://en.wikipedia.org/wiki/Compound_interest#Continuous_compounding).
Again, this is nothing obscure, a lot of people will learn it in high school or early college math whether they go on to study economics and accounting, or not.
But, in a typical bad pedagogy mistake, too much emphasis is on how to "solve" the problem/equation/whatever, and not enough is on *what* the problem to be solved *is*.

For loans, the interest rate is constant.
Interesting things just stem from irregular (or arbitrary) payments.
But in general, we also want to consider non-constant, time varying growth.
Regular integration is for "continuous sums";
per the previous section, if the right way to deal with growth is not iterated addition but multiplication,
then what we are looking for is "continuous products".

Enter, the [product integral](https://en.wikipedia.org/wiki/Product_integral).

When I was first taking economics masters classes at [John Jay](https://johnjayeconomics.org/), that Wikipedia article was in a lot worse shape, and didn't give me the answers I was looking for.
But now it is much better, and in particular it now cites <doi:10.1016/j.jmaa.2007.03.081> which is fantastic!
I'll recap some parts of it, but it's short, and you should just go read it yourself.

### Multiplicative derivative

The multiplicative derivative is defined as follows:

```math
f^* := x \mapsto \lim_{h \to 0} \left( \frac{f(x + h)}{f(x)} \right)^\frac{1}{h}
```

The crucial things to note are that:

- Compared to the usual "additive" derivative, each arithmetic operation of "output" values is "promoted":
  addition to multiplication, multiplication to exponentiation.

- There is no addition (or subtraction) of "output" values.

This indicates we are now working on the level of multiplication, and not cheating.
Indeed, the output type of $f$ could be something for which addition is not even defined![^ring-like]

[^ring-like]:
  However, needing exponentiation means we still need some ring-like structure.
  The other flavours of product integrals on the Wikipedia page get into this more, including changing up the definition in ways that are, in my opinion, dubious and against the point of this piece, for scalars, to dodge the exponentiation requirement.

After some manipulations (which you should definitely read work through!) the paper shows this definition equivalent to

```math
f^* := x \mapsto e^{(\ln \circ f)'(x)}
```

Rewritten in [point-free](https://wiki.haskell.org/Pointfree) style, where $D_+$ is the regular (additive) [differential operator](https://en.wikipedia.org/wiki/Differential_operator) and $D_*$ is our new one:

```math
\begin{aligned}

D_* &= (\exp \circ) \cdot D_+ \cdot (\ln \circ) \\
    &= (\exp \circ) \cdot D_+ \cdot (\exp^{-1} \circ) \\
    &= (\exp \circ) \cdot D_+ \cdot (\exp \circ)^{-1}
\end{aligned}
```
(Note: the inner $\circ$ is for function composition for real functions, $\mathbb{R} \to $\mathbb{R}$, whereas the outer $\cdot$ is for function composition for real-to-real functions, $(\mathbb{R} \to $\mathbb{R}) \to (\mathbb{R} \to $\mathbb{R})$.)

We can now see the very nice way our new form of differentiation looks something like a group conjugation: tweak (the function), differentiate, and then untweak.

This is very close to the plogarithmic derivative](https://en.wikipedia.org/wiki/Logarithmic_derivative),
except that one skips the final $\exp$ step, loosing the symmetry.
