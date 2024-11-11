# Fixing the Mathematics of Growth for Economics and Accounting

## Intro

There's something wrong with economics and accounting math.
Not "wrong-answers" wrong, but bad pedagogy, and icky approximations.

It's easy to go after approximations — "No! I insist that you write this other formula that is actually correct" ...and far larger.
And indeed, if there was just a trade off between precision and bloat, I would not bother writing this.
Rather, and what really gets me excited, is that there is no such trade-off — we can be both more correct and just as (or more!) terse.
We just need to use the right abstractions.

## A first example

Here's a starting point.
Consider a phrase like "2% growth";
you've seen it newspapers, in high school math doing loan interest, and maybe in econ or accounting classes too.
The meaning of the phase is "went from 100% to 102%" (over some period), adding another 2% on top.

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

$$
a + 2\% \cdot a + 2\% \cdot a
$$

we can do (right!):

$$
a \cdot (1 + 2\%) \cdot (1 + 2\%)
$$

or more simply,

$$
a \cdot 102\% \cdot 102\%
$$

And there we have a bit a [shibboleth](https://en.wikipedia.org/wiki/Shibboleth):
call it "102% growth", not "2% growth".
It might be too late to change English, but one can dream...

### Differing growth amounts

Suppose we had $g_0$ growth for one period, $g_1$ for another, and so one.
What does that look like?

In the conventional regime where the steady-state is $\bar{g} = 0\%$ not $g = 100\%$, that's

$$
a_n = a_0 \cdot (1 + \bar{g}_0) \cdot (1 + \bar{g}_1) \cdots
$$

But with the "corrected" variables, that's simply

$$
a_n = a_0 \cdot g_0 \cdot g_1 \cdots
$$

Clearly this is terser.
But, this just the same point as the previous section, now made using variables rather than concrete percentages.

### Differing period lengths

Now, say we want to incorporate time.
Our periods will be different lengths: $\Delta t_0$, $\Delta t_1$, etc.
The growth of each period will be given not as literally occurred over that period, but according to what it would be over a unit length of time.
That is to day, we will not specify the growth with

$$
g'_n := \frac{a_{n +1}}{a_n}
$$
but with
$$
g_n := \left( \frac{a_{n +1}}{a_n} \right)^{\frac 1 {\Delta n}}
$$

the final amount (post growth) will thus be

$$
a_n = a_0 \cdot g_0^{\Delta t_0} \cdot g_1^{\Delta t_1} \cdots g_n^{\Delta t_n}
$$

This is correct, and terse.

But I must note there is (again, if you read the previous aside) a problem with dimensional analysis.
$\frac 1 {\Delta t}$ is not dimensionless, but rather has dimension $\frac 1 {\mathrm{T}}$, i.e. units like $\frac 1 {\mathrm{seconds}}$ $\frac 1 {\mathrm{years}}$ or similar.
Both numbers given to exponentiation must be dimensionless, so we shouldn't be doing this.
There is a solution, however, which is to use the identity $g = e^{\ln g}$.
Rewriting with that, we have
$$
a_n = a_0 \cdot e^{(\ln g_0) \Delta t_0} \cdot e^{(\ln g_1) \Delta t_1} \cdots e^{(\ln g_n) \Delta t_n}
$$

$\ln g_t$ we hereby declare to have dimension $\frac 1 T$, and now the final exponent $\ln g_t \Delta t$ properly has dimension $1$ (dimensionless).
The logarithm is just as dimensionally invalid as the exponent before, but for that we just need to inline further.
Recall our definition  for $g_n$:

$$
g_n = \left( \frac{a_{n +1}}{a_n} \right)^{\frac 1 {\Delta n}}
$$
thus
$$
\begin{aligned}
\ln g_n & = \ln \left( \left( \frac{a_{n +1}}{a_t} \right)^{\frac 1 {\Delta t_n}} \right) \\
&= \frac 1 {\Delta t_n} \ln \left( \frac{a_{n +1}}{a_n} \right) \\
&= \frac 1 {\Delta t_n} \left( \ln a_{n +1} - \ln {a_n} \right) \\
\end{aligned}
$$

The final right-hand side "properly" has the proper $\frac 1 {T}$ dimension;
we've additionally found and fixed another dimensional analysis violation in the definition of $g_t$.
Lets call this value $\bar{\bar{g}}$:

$$
\bar{\bar{g}} := \frac 1 {\Delta t} \left( \ln a_{t +1} - \ln {a_t} \right)
$$
so that our formula for $a_t$ is:
$$
a_n = a_0 \cdot e^{\bar{\bar{g}}_0 \Delta t_0} \cdot e^{\bar{\bar{g}}_1  \Delta t_1} \cdots  e^{\bar{\bar{g}}_n  \Delta t_n}
$$
or equivalently
$$
a_n = a_0 \cdot e^{\bar{\bar{g}}_0 \Delta t_0 + \bar{\bar{g}}_1  \Delta t_1 \cdots \bar{\bar{g}}_n  \Delta t_n}
$$

Now everything is dimensionally correct.

## Multiplicative sequence pre-calculus

Let's go over some math before returning to examples / motivation

### Products of sequences

Suppose we have a loan with a variable (compound) interest rate $r$, the outstanding balance is calculated every unit interval, and no payments are made.
Because of the discrete points at which the interest is calculated, $r$ can be a (real-valued) sequence ($\mathbb{N} \to \mathbb{R}$).
The nice way to compute the outstanding balance is thus with products of a sequence:

$$
B_t = A \prod_{u = 0}^t r_u
$$

Similar to lumping together the 1 and 2% above as 102%, note that in the formula above the balance seems more "fundamental" than the total interest:
$B_t - A$ is the total interest written in terms of the balance (and principle), and there isn't an obvious way to rewrite that expression such that we "skip" calculating the balance.

The subscripts in the above formula don't add too much value in this case.
We can define a product operator on sequences as follows:

$$
\prod s := n \mapsto \prod_{i = 0}^n s_i
$$

And then (with arithmetic on sequences defined point-wise), the balance formula above can be rewritten

$$
B = A \prod r
$$

### Ratios of sequences

If we have some sequence $s$, the *(forward) quotient operator* is[^qoppa]

[^qoppa]: https://math.stackexchange.com/q/3691073 made the cheeky suggestion to use the archaic Greek letter "qoppa" for this. I like it!

$$
Ϙ s := n \mapsto \frac{s_{n+1}}{s_n}
$$

We have a nice "fundamental therorem" where

$$
Ϙ \left( \prod s \right) = s
$$

and

$$
\prod Ϙ s = s / s_0
$$

### Variable rate loans

TODO

## Multiplicative Calculus

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

### Multiplicative integral

Enter, the [multiplicative integral](https://en.wikipedia.org/wiki/Product_integral).

When I was first taking economics masters classes at [John Jay](https://johnjayeconomics.org/), that Wikipedia article was in a lot worse shape, and didn't give me the answers I was looking for.
But now it is much better, and in particular it now cites <doi:10.1016/j.jmaa.2007.03.081> which is fantastic!
I'll recap some parts of it, but it's short, and you should just go read it yourself.

Like most calculus texts, that paper covers differentiation before integration, but because the examples we're working from above, let's do the opposite.

$$
{\huge \mathscr{P}}_a^b f(x)^{dx}
$$

TODO

### Multiplicative derivative

The multiplicative derivative is defined as follows:

$$
f^* := x \mapsto \lim_{h \to 0} \left( \frac{f(x + h)}{f(x)} \right)^\frac{1}{h}
$$

The crucial things to note are that:

- Compared to the usual "additive" derivative, each arithmetic operation of "output" values is "promoted":
  addition to multiplication, multiplication to exponentiation.

- There is no addition (or subtraction) of "output" values.

This indicates we are now working on the level of multiplication, and not cheating.
Indeed, the output type of $f$ could be something for which addition is not even defined![^ring-like]

[^ring-like]:  However, needing exponentiation means we still need some ring-like structure.
The other flavours of product integrals on the Wikipedia page get into this more, including changing up the definition to dodge the exponentiation requirement.
I am a bit conflicted on this, because on one hand it is useful to make it work with less structured / more general codomains of the integrand, but on the other hand, those changes, when applied to scalars, go against what I am arguing in this piece!

After some manipulations (which you should definitely read work through!) the paper shows this definition equivalent to

$$
f^* := x \mapsto e^{(\ln \circ f)'(x)}
$$

Rewritten in [point-free](https://wiki.haskell.org/Pointfree) style, where $D_+$ is the regular (additive) [differential operator](https://en.wikipedia.org/wiki/Differential_operator) and $D_*$ is our new one:

$$
\begin{aligned}
D_* &= (\exp \circ -) \cdot D_+ \cdot (\ln \circ -) \\
    &= (\exp \circ -) \cdot D_+ \cdot (\exp^{-1} \circ -) \\
    &= (\exp \circ -) \cdot D_+ \cdot (\exp \circ -)^{-1}
\end{aligned}
$$

(Note: the inner $\circ$ is for function composition for real functions, $\mathbb{R} \to \mathbb{R}$, whereas the outer $\cdot$ is for function composition for real-to-real functions, $(\mathbb{R} \to \mathbb{R}) \to (\mathbb{R} \to \mathbb{R})$.)

We can now see the very nice way our new form of differentiation looks something like a group conjugation: tweak (the function), differentiate, and then untweak.

## Logarithmic derivative, not quite what we want

This is very close to the [logarithmic derivative](https://en.wikipedia.org/wiki/Logarithmic_derivative),
except that one skips the final $\exp$ step, losing the symmetry.

### Multiplicative infinitisemals?

This would be nice for informal multiplicative differential equations, other applied tasks.

## Other topics

### Elasticity

The [Wikipedia article for elasticity](https://en.wikipedia.org/wiki/Elasticity_(economics), like most econ texts I could find from a quick glance, just has an informal definition made from infinitesimals:

The $x$-elasticity of $y$ is:
$$
\epsilon := \frac{\partial y / y}{\partial x / x}
$$

I won't lie, that is pretty.
But it does more suspicious addition — despite looking like all division — in the form of the infinitesimals.
This is because infinitesimals, as "funny zeros" --- funny additive identities --- are an additive concept.
Or, if that is a bit too much woo-woo, more prosaically it is because they stem from subtraction in limits.

This [other wikipedia article](https://en.wikipedia.org/wiki/Elasticity_of_a_function) has a limit definition:
$$
\epsilon(f) := \frac{x}{(f(x)}f'(x)
$$

suspicious addition/subtractions on output values.

A definition free of output-valued addition could look like this:

$$
E_*(f) := x \mapsto \lim_{m \to 1} \left( \frac{f(m x)}{m f(x)} \right)
$$
