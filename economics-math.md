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

```math
a + 2\% \cdot a + 2\% \cdot a
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

### Differing growth amounts

Suppose we had $g_0$ growth for one period, $g_1$ for another, and so one.
What does that look like?

In the conventional regime where the steady-state is $\bar{g} = 0\%$ not $g = 100\%$, that's

```math
a_n = a_0 \cdot (1 + \bar{g}_0) \cdot (1 + \bar{g}_1) \cdot \ldots \cdot (1 + \bar{g}_{n-1})
```

But with the "corrected" variables, that's simply

```math
a_n = a_0 \cdot g_0 \cdot g_1 \cdot \ldots \cdot g_{n-1}
```

Clearly this is terser.
But, this just the same point as the previous section, now made using variables rather than concrete percentages.

### Differing period lengths

Now, say we want to incorporate time.
Our periods will be different lengths: $\Delta t_0$, $\Delta t_1$, etc.
The growth of each period will be given not as literally occurred over that period, but according to what it would be over a unit length of time.
That is to day, we will not specify the growth with

```math
g'_n := \frac{a_{n +1}}{a_n}
```
but with
```math
g_n := \left( \frac{a_{n +1}}{a_n} \right)^{\frac 1 {\Delta n}}
```

the final amount (post growth) will thus be

```math
a_n = a_0 \cdot g_0^{\Delta t_0} \cdot g_1^{\Delta t_1} \cdot \ldots \cdot g_{n-1}^{\Delta t_{n-1}}
```

This is correct, and terse.

But I must note there is (again, if you read the previous aside) a problem with dimensional analysis.
$\frac 1 {\Delta t}$ is not dimensionless, but rather has dimension $\frac 1 {\mathrm{T}}$, i.e. units like $\frac 1 {\mathrm{seconds}}$ $\frac 1 {\mathrm{years}}$ or similar.
Both numbers given to exponentiation must be dimensionless, so we shouldn't be doing this.
There is a solution, however, which is to use the identity $g = \exp (\ln g)$.
Rewriting with that, we have
```math
a_n = a_0 \cdot \exp (\Delta t_0 \cdot \ln g_0) \cdot \exp ( \Delta t_1 \cdot \ln g_1) \cdot \ldots \cdot \exp (\Delta t_{n-1} \cdot \ln g_{n-1})
```

$\ln g_t$ we hereby declare to have dimension $\frac 1 T$, and now the final exponent $\ln g_t \Delta t$ properly has dimension $1$ (dimensionless).
The logarithm is just as dimensionally invalid as the exponent before, but for that we just need to inline further.
Recall our definition  for $g_n$:

```math
g_n = \left( \frac{a_{n +1}}{a_n} \right)^{\frac 1 {\Delta n}}
```
thus
```math
\begin{aligned}
\ln g_n & = \ln \left( \left( \frac{a_{n +1}}{a_t} \right)^{\frac 1 {\Delta t_n}} \right) \\
&= \frac 1 {\Delta t_n} \ln \left( \frac{a_{n +1}}{a_n} \right) \\
&= \frac 1 {\Delta t_n} \left( \ln a_{n +1} - \ln {a_n} \right) \\
\end{aligned}
```

The final right-hand side "properly" has the proper $\frac 1 {T}$ dimension;
we've additionally found and fixed another dimensional analysis violation in the definition of $g_t$.
Lets call this value $\bar{\bar{g}}$:

```math
\bar{\bar{g}} := \frac 1 {\Delta t} \left( \ln a_{t +1} - \ln {a_t} \right)
```
so that our formula for $a_t$ is:
```math
a_n = a_0 \cdot \exp(\Delta t_0 \cdot \bar{\bar{g}}_0) \cdot \exp(\Delta t_1 \cdot \bar{\bar{g}}_1)  \cdot \ldots \cdot  \exp(\Delta t_{n-1} \cdot \bar{\bar{g}}_{n-1})
```
or equivalently
```math
a_n = a_0 \cdot \exp(\Delta t_0 \cdot \bar{\bar{g}}_0 + \Delta t_1 \cdot \bar{\bar{g}}_1  + \ldots + \Delta t_{n-1} \cdot \bar{\bar{g}}_{n-1})
```

Now everything is dimensionally correct, and still terse.
Conversely, rewriting any of these equations with $\bar{g}$ would have cluttered things up again, only introducing more $\_ + 1$ inside these formulae.
That would have been a real mess.

The terse approximation for $a_n$, only valid for low $g$ and low $n$, is:

```math
a_n \approx a_0 \cdot (1 + \Delta t_0 \cdot \bar{g}_0 + \Delta t_1 \cdot \bar{g}_1  + \ldots + \Delta t_{n-1} \cdot \bar{g}_{n-1})
```

but that is hardly more terse!

### Sorting out $g$, $\bar{g}$, and $\bar{\bar{g}}$

$\bar{\bar{g}}$, the final sort of growth rate variable we settled on, does admittedly have some similarites to $\bar{g}$, the one we rejected.
$\ln g$ and $g - 1$ are similar when $g$ is close to 1, as is the case for most macro growth rates, and identical at 1: $\ln 1 = 1 - 1 = 0$.
Perhaps correctness and usefulness of $\bar{\bar{g}}$ boosted $\bar{g}$'s reputation by means of the latter being a convenient approximation for the former.

In any event, we will expressions that look like $g$ and $\bar{\bar{g}}$  in the further developments below.

## Multiplicative sequence pre-calculus

So far, we've used standard concepts and notations, even if the emphasis on absolute syntactic rigor with no approximations is a bit unidiomatic.
Now, however, will introduce not harder but more obscure concepts

### Products of sequences

Suppose we have a loan with a variable (compound) interest rate $r$, the outstanding balance is calculated every unit interval, and no payments are made.
Because of the discrete points at which the interest is calculated, $r$ can be a (real-valued) sequence ($\mathbb{N} \to \mathbb{R}$).

The formula for the balance is informally.

```math
B_t = A \cdot r_0 \cdot r_1 \cdot r_2 \cdot \ldots \cdot r_{t -1 }
```

Similar to lumping together the 1 and 2% above as 102%, note that in the formula above the balance seems more "fundamental" than the total interest:
$B_t - A$ is the total interest written in terms of the balance (and principle), and there isn't an obvious way to rewrite that expression such that we "skip" calculating the balance.

We can do better, formalizing it, with the notion of of a product of a sequence:

```math
B_t = A \prod_{u = 0}^{t - 1} r_u
```

The subscripts in the above formula don't add too much value in this case.
We can define cumulative product and sum operators on sequences, that is, $\sum, \prod : (\mathbb{N} \to \mathbb{R}) \to (\mathbb{N} \to \mathbb{R})$, as follows:

```math
\sum s := 0 :: \left( n \mapsto \sum_{i = 0}^n s_i \right)
```
```math
\prod s := 1 :: \left( n \mapsto \prod_{i = 0}^n s_i \right)
```
where $v :: s$ "delays" $s$ by one, using $v$ as the new initial value.

And then (with arithmetic on sequences defined point-wise), the balance formula above can be rewritten:

```math
B = A \cdot \left(\prod r \right)
```

Just as we did above, this can be rewritten with more familiar summation with $\exp$:

```math
B = A \cdot \exp\left(\sum \ln r \right)
```

### Ratios of sequences

If we have some sequence $s$, the *(forward) quotient operator* is[^qoppa]

[^qoppa]: https://math.stackexchange.com/q/3691073 made the cheeky suggestion to use the archaic Greek letter "qoppa" for this. I like it!

```math
{\Large Ϙ} s := n \mapsto \frac{s_{n+1}}{s_n}
```

We have a nice "fundamental therorem" where

```math
{\Large Ϙ} \left( \prod s \right) = s
```

and

```math
\prod \left( {\Large Ϙ} s \right) = s / s_0
```

### Differing period lengths, again

We can now use the above path to rewrite our formulae for growth with varying rates for varying period lengths much more succinctly.
The informal $\ldot$ can be replaced with the use our cumulative sequential operators.

```math
\begin{align}
a
& := a_0 \cdot \prod \exp(\Delta t \cdot \bar{\bar{g}}) \\
& ~ = a_0 \cdot \exp\left(\sum \Delta t \cdot \bar{\bar{g}}\right)
\end{align}
```

The commuting of the cumulative product/sum (it changes) and exponentiation is on display with little else to distract from it.

### Loan with payments

Now let's add one more complication to our modeling goal. We'll have a variable interest rate, and variable time lengths, like before, but also variable loan payments (imagine the debtor gets behind and tries to check up).

The recurrence relation is this:
```math
B_{n +1} = (B_n - P_n) \cdot \exp(\bar{\bar{r}}_n \cdot \Delta t_n)
```
which is say the next balance before interest is the old balance less the payment.
Then the final next balance also includes the interest is calculated from that intermediate total.

We can do "discrete differential equations", both additive and multiplicative, for this:
```math
\Delta B = (B - P) \cdot \exp(\bar{\bar{r}} \cdot \Delta t) - B
```
```math
{\Large Ϙ} B = \left(1 - \frac P B \right) \cdot \exp(\bar{\bar{r}} \cdot \Delta t)
```

Admittedly, neither of these look very pretty.

We can write the first additive one:
```math
\Delta B = (B - P) \cdot (\exp(\bar{\bar{r}} \cdot \Delta t) - 1) - P
```
where $\exp(\bar{\bar{r}} \cdot \Delta t) - 1$ is just "pure interest" rate forthat period, something like $2\%$ not $102\%$ as we've been arguing against.
However, this does not actually simplify the formula, for two reasons.
Firstly, the interest rate itself tersely reaonably use $\bar{r}$, because the $\Delta t$ has to be an exponent (as we've discussed before.
Secondly, we're stuck repeating the $-P$ because we need to calculate the interest *after* including the payment.

There is not a close-form equation for this in the style of what we've done so far — only the recurence relation which gets around the issue with subscripts. The fundamental problem is that the loan payments are inherently additive, while the interest calculation is inherently multiplicative, so we cannot express the sequence as single cumulative sum or product for a closed-form solution.

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

Defining integrals formally is pain, so let's do something short of that.

```math
{\huge \mathscr{P}}_a^b f(x)^{dx} := \lim_{\Delta x \to 0} \prod f(x)^{\Delta x}
```

The multiplicative Riemann integral is "defined" as the limit of the product of increasingly many samples of $f(x)$ taken to the $\Delta x$ power.

The geometric intuitions off this are not as clear as the additive Riemann integral because are multiplicands of this are not "little rectangles" the way the addends of that are.
Indeed, as $x$ and $f(x)$ must be dimensionless, any geometric/visual intuition is inherently suspect!

This definition can be reworked into a regular integral:

```math
\begin{align}
{\huge \mathscr{P}}_a^b f(x)^{dx}
& = \lim_{\Delta x \to 0} \prod f(x_i)^{\Delta x} \\
& = \lim_{\Delta x \to 0} \prod \exp\left((\ln f(x_i)) \cdot {\Delta x}\right) \\
& = \lim_{\Delta x \to 0} \exp\left(\sum (\ln f(x_i)) \cdot {\Delta x}\right) \\
& = \exp\left(\lim_{\Delta x \to 0} \sum (\ln f(x_i)) \cdot {\Delta x}\right) \\
& = \exp\left(\int_a^b \ln f(x) \cdot {\Delta x}\right)
\end{align}
```

This saves us from actually needing to rigorously define the product integral from scratch.


### Multiplicative derivative

The multiplicative derivative is defined as follows:

```math
f^* := a \mapsto \lim_{x \to a} \left( \frac{f(x)}{f(a)} \right)^\frac{1}{x -a}
```

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

```math
f^* := x \mapsto e^{(\ln \circ f)'(x)}
```

Rewritten in [point-free](https://wiki.haskell.org/Pointfree) style, where $D_+$ is the regular (additive) [differential operator](https://en.wikipedia.org/wiki/Differential_operator) and $D_*$ is our new one:

```math
\begin{aligned}
D_* &= (\exp \circ \_) \circ D_+ \circ (\ln \circ \_) \\
    &= (\exp \circ \_) \circ D_+ \circ (\exp^{-1} \circ \_) \\
    &= (\exp \circ \_) \circ D_+ \circ (\exp \circ \_)^{-1}
\end{aligned}
```

> Note: the inner $\circ$ is for function composition for real functions, $\mathbb{R} \to \mathbb{R}$, whereas the outer $\circ$ is for function composition for real-to-real functions, $(\mathbb{R} \to \mathbb{R}) \to (\mathbb{R} \to \mathbb{R})$.

We can now see the very nice way our new form of differentiation looks something like a group conjugation: tweak (the function), differentiate, and then untweak.

### Logarithmic derivative, not quite what we want

This is very close to the [logarithmic derivative](https://en.wikipedia.org/wiki/Logarithmic_derivative),
except that one skips the final $\exp$ step, losing the symmetry.

## Other topics

### Elasticity

The [Wikipedia article for elasticity](https://en.wikipedia.org/wiki/Elasticity_(economics)), like most econ texts I could find from a quick glance, just has an informal definition made from infinitesimals:

The $x$-elasticity of $y$ is:
```math
\epsilon := \frac{\partial y / y}{\partial x / x}
```

I won't lie, that is pretty.
But it does more suspicious addition — despite looking like all division — in the form of the infinitesimals.
This is because infinitesimals, as "funny zeros" — funny additive identities — are an additive concept.
Or, if that is a bit too much woo-woo, more prosaically it is because they stem from subtraction in limits.

This [other Wikipedia article](https://en.wikipedia.org/wiki/Elasticity_of_a_function) has a formal limit definition:
```math
\epsilon(f) := \frac{x}{f(x)} \cdot f'(x)
```

but underneath the definition of the derivatie are the suspcious addition/subtractions on values with output-dimension we'd like to avoid.

However, that article also transforms the original defintion into

```math
\epsilon(f) := \lim_{x \to a} \frac{\frac {f(x)}{f(a)} - 1} {\frac x a -1}
```

With this defintion, we divide first, and then only subtract dimensionless values.
This sucesfully avoids any critism for suspicous subtrations.
Also, the $-1$ is very natural in this case: it would seem to be the perfect repudiation to my original claim that using values like $2\%$ rather than $102\%$ is unnatural and misguided!

However, there is anothe problem with this, and a solution in the more the vein I am thinking.
Recall that the curves of constant elasticity are [power-law functions](https://en.wikipedia.org/wiki/Power_law#Power-law_functions) in the form:

```math
x \mapsto a \cdot x^k
```

(In particular, $\epsilon(x \mapsto a \cdot x^k) = k$.)

Recall the limit definition of a derivative
```math
f' = a \mapsto \lim_{x \to a} \frac {f(x) - f(a)} {x - a}
```
The standard gemometric interpretation of this is we have a family of secants, with the two points of the secant growing ever closer together, and their limit is the one-point tangent.[^tangent-limit] expresion underneath the limit is the slopes of the family of secants, and with the limit it is the slop of the tangent.

[^tangent-limit]: For anyone not familiar, the [Wikipedia page on tangants](https://en.wikipedia.org/wiki/Tangent) speaks of this limit somewhat.

Less well-known is the idea that we can do a similar geometric construction for elasticities.
Two points determine a power-law function just as they determine a line; we can thus speak of a "power-law secent", and in the limit as the two points approach, we have a "power law tangent".

The curves of constant slope are just lines, graphs of functions in the form $x \mapsto  a \cdot x + b$. In this case, we note two lemmas, one geometric and the other arithmatic:
- **geometric**: the original line *is* every line in the family of secants *is* the tangent line
- **arithmetic**: the limit is trivial and we can just as well use the underlying expression for any value of $x$ and $a$ to calculate the constant slope.

For good practice, lets proove the second lemmma:

```math
\frac {(c \cdot x + b) - (c\cdot a +b)} {x - a} = \frac {c\cdot (x - a)} {x - a} = c
```

Likewise, we would expect the same thing about curves of constant elasticity:
- **geometric**: the original power law curve[^power-law-curve], the every curve in the family of "power law secants", and the "power law tangent" are all the same curve.
- **arithmetic**: the limit is trivial and we can just as well use the underlying expression for any value of $x$ and $a$ to calculate the constant elasticity.

[^power-law-curve]: the properties we care about of these curves are not translation-invariant; on the contrary the location of the origin in crucial for comparing ratios of inputs to ratios of outputs. It is therefore fair to point out this is not "geometric" in the usual euclidean sence.

The geoemetric lemma is true for power law curves, but with the formulae given above, the arithmetic lemma is *false*!
Let's try substituting an aribtrary power-law function and simplifying:

```math
\frac {\frac {b \cdot x^c} {b\cdot a^c} - 1} {\frac x a - 1} = \frac {\frac {x^c} {a^c} - 1} {\frac x a - 1}
```

We can't readily simplify it further, and if we plug in different values for $x$ and $a$, we in fact get different results! We do not get $c$ in all case, even though the (constant) elasticity everywhere on the curve is in fact $c$.

Is all hope lost? Is elasticity just a more broken concept than slope? Not so! The key is we just need a different formula

Try this:

```math
\epsilon(f) \stackrel{?}{=} a \mapsto \lim_{x \to a} \log_{x/a}{\frac {f(x)} {f(a)}}
```

The intution here is we are comparing a small *multiplicative* perturbation in the input to the corresponding perturbation in the output, and instead of taking quotient of these quantities (inverse binary multiplication), we are taking the logarithm (inverse binary exponentiation). We are asking, what power of the input multiplicative perturbation yeilds the output multiplicative perturbation?

It is very interesting to compare this definition, the multiplicative derivative from before, and the regular additive derivative. The multiplicative derivative "upgraded" *output*-dimesion operation (subtraction to division), but left *input*-dimesion one the same, additive as before (that is $a - x$ is still the same). The multiplicative derivative's mixed dimension $\frac {\mathrm{Output}} {\mathrm{Input}}$ operation (division) also got upgraded to a root. (Note that after this upgrading, the mixed-dimension operation becomes a dimensionless one.)

In this definition, *both* the input and output dimension operations are upgraded, $f(x) - f(a)$ becomes $\frac {f(x)} {f(a)}$, and $x - a$ becomes $x / a$. The mixed dimension division gets upgraded to a different sort of next level operation, the logarithm. It might be tempting to think of this function as "more upgraded" since the input operations are updated too, and logarithms are more "exotic" than roots, but keep in mind that there is also a tension between inputs and outputs, and in some sense upgrading both "cancels out" the upgrading some more.

Finally, lets note that we can rewite the non-standard-base logarithm the usual way.
```math
\epsilon(f) \stackrel{?}{=} a \mapsto \lim_{x \to a} \frac {\ln \frac {f(x)} {f(a)}} {\ln \frac x a} = a \mapsto \lim_{x \to a} \frac {\ln f(x) - \ln f(a)} {\ln x  - \ln a}
```
I didn't do this before because it obscures the analogies I wanted to make, and also because the third expression above is not dimensionally compliant. But I include them now since these are more "conventional" formulae, at what I deem less cost.

Is this formula valid for elasticity? Well, it does work for power-law functions:

```math
a \mapsto \lim_{x \to a} \frac {\ln \frac {b \cdot x^c} {b \cdot a^c}} {\ln \frac x a}
```
```math
a \mapsto \lim_{x \to a} \frac {c \cdot \ln \frac {x} {a}} {\ln \frac x a}
```
```math
a \mapsto \lim_{x \to a} c
```

And even better, look how we were able to derive a constant without first solving the limit!
That bails out our second property after all, if this formula is in fact a correct definition:
the new limit is trivial for power law functions, and thus we do not need to use a limit to compute functions where the elasticity is everywhere constant.

Finally, we sketch a proof that it is.
The proof is limit more than our previous observation comparing $\bar{g}$ and $\bar{\bar{g}}$:
$\ln$ and $x \mapsto x - 1$ are similar functions close to 1, and by [L'Hôpital's_rule](https://en.wikipedia.org/wiki/L%27H%C3%B4pital%27s_rule), the original and new formula (with the limits) are in fact equal.

For the record, the new formula is not entirely made up by me. The wikipedia pages after all have the informal
```math
\epsilon = \frac {d \ln y} {d \ln x}
```
It is not exactly clear what this means just looking at it alone, but as far as I can tell what this correspond to is the final rewrite we did above:
```math
\epsilon(f) = a \mapsto \lim_{x \to a} \frac {\ln f(x) - \ln f(a)} {\ln x  - \ln a}
```

### Multiplicative infinitesimals?

This would be nice for informal multiplicative differential equations, other applied tasks.

I am not sure but above I talked about "multiplicative perturbations".
We also concluded with pointing out the $d\ln x$ in the Wikipedia article.
If the $d$ "makes the expression very close to 0"; with the logarithm, that would make the $x$ close to 1.
let's say that $q$ "makes the expression very close to 1", makes a multiplicative infinitesimal.
Then we have the following things:

An identity:
```math
d \ln x = \ln q x
```

Elasticity a new way:
```math
\epsilon = \frac {\ln q y} {\ln q x} = \log_{q x} {q y}
```

The multiplicative derivative is:
```math
\sqrt[d x] {q y}
```

Food for thought!
