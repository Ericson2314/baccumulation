# Geometric Calculus

In search of better formalisms for the reason described in my [economics math](../economics-math.md) write up,
I found my way to the ["product integral"](https://en.wikipedia.org/wiki/Product_integral) wikipedia page.

When I was first taking economics masters classes at [John Jay](https://johnjayeconomics.org/), that Wikipedia article was  in a lot worse shape, and didn't give me the answers I was looking for.
But now it is much better, and in particular it now cites <doi:10.1016/j.jmaa.2007.03.081> which is fantastic!
You should just go read the paper yourself.

There is also <https://sites.google.com/site/nonnewtoniancalculus> which contains a great many other resources, from the people that started this line of research (as far as anyone knows).
The authors [argue](https://sites.google.com/site/nonnewtoniancalculus/-multiplicative-calculus) that the term "multiplicative calculus" should not be used because it is ambiguous, which I found compelling, so I have re-tiled this page accordingly.

I don't like doing the "unoriginal recap as blogpost" thing, but there are a few things I want to highlight,
and also a few more basic precaluculus notions (that the paper did not cover) that I want to place alongside them.

## Sequence pre-calculus

## $\mathbb{R} \to \mathbb{R}$ function calculus

### Geometric derivative

The geometric derivative is defined as follows:

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

### Geometric integral

Defining integrals formally is pain, so let's do something short of that.

```math
{\huge \mathscr{P}}_a^b f(x)^{dx} := \lim_{\Delta x \to 0} \prod f(x)^{\Delta x}
```

The geometric Riemann integral is "defined" as the limit of the product of increasingly many samples of $f(x)$ taken to the $\Delta x$ power.

The spatial/geometric ["geometric" in the strict sense of "geometry", not "multiplication"] intuitions of this are not as clear as the additive Riemann integral because are multiplicands of this are not "little rectangles" the way the addends of that are.
Indeed, as $x$ and $f(x)$ must be dimensionless, any spatial/visual intuition is inherently suspect!

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

This saves us from actually needing to rigorously define the geometric integral from scratch.
