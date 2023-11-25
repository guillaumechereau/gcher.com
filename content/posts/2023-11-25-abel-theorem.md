---
categories: [math, groups]
title: Abel's theorem
date: 2023-11-25
math: true
---


The Abel-Ruffini theorem states that there is no solution to the general
polynomial equation of degree five (quintic equation) that can be expressed
with the common operations: (+, -, x, %), plus `√` (radical of any degree).

In this post I'll try to give my quick understanding of the proof, without
explicitly relying on group theory or Riemann surface.  As usual on this blog,
this is all hand wavy.  For a proper introduction to the proof I recommend the
excellent book "[Abel's Theorem in Problems and Solutions]" by V.B. Alekseev.

The proof relies on the non-intuitive fact that certain functions of complex
numbers can change values when we smoothly move the arguments along a closed
loop.  Let's take the function y = √x for example, starting with x = 1,
and y = 1.  If we smoothly rotate x along the unit circle, y will also rotate
at half the speed.  As we return to x = 1, y will be equal to -1.

![Square root](/imgs/abel-0.png)

# Permutations of the polynomial root solutions

Let's assume that we have a function $(r0, r1, r2, r3, r4) = F(a, b, c, d, e)$
that gives us the roots of a polynomial of degree 5 from its coefficient.

It is easy to find the coefficient from the roots, since we know that the
polynomial can be written as the product of the roots minus x:

$$
    P(x) = (r0 - x)(r1 - x)(r2 - x)(r3 - x)(r4 - x)
$$

By expending this and matching with the polynomial in its original form we
can directly recompute the coefficients, which mean we can easily find the
inverse function:

$$
    (a, b, c, d, e) = F'(r0, r1, r2, r3, r4).
$$

If we smoothly move the roots value in order to shuffle them, the coefficients
will return to the same values, so we also have the possibility of
doing closed loops of the coefficients resulting in *any* permutations of the
roots.  This is the property that we are interested in: we will show
that expressions with radicals are not flexible enough to allow for all
permutations of the five roots of the quintic polynomial.

![Polynomial roots permutations](/imgs/abel-1.png)

# Restriction on the permutations with a single radical

If we consider an expression with only one radical, something like:
$r = a + \sqrt[4]{b + c}$.  We have four possible values for r (given that used
a fourth root), but not all permutations are possible.

We can easily see that as we apply a loop, all the values will move
together at the same time, so that we can for
example do a permutation r0 → r1 → r2 → r3 → r0 but not
r0 → r1 → r0.  In general the restriction is that if we take the set
of all the possible permutations, applying two of them in any order should
result in the same permutations.  We can express that by using the
commutator of two permutations, which is the final permutation we get
if we apply two permutation followed by their inverse:
$[A, B] = A B A^{-1} B^{-1}$.

So the restriction of any expression involving a single radical is that the
set of commutators for any possible permutation of the values we can get by
doing closed loops of the arguments should contain only the identity.

Already we can tell that the expression for the quintic polynomial roots,
if it exists, cannot be using a single radical operator, since otherwise
we would not be able to do any permutations of the five roots.

# Representation of any expression as a graph

What about if we use more radicals, in particular if we use an expression with
radicals of radicals?  Maybe this allows us to get some more complex
permutations, and can we still find some restriction on the set of possible
permutations?  In order to reason about composed expression, it is interesting
to see that we can express an expression as a tree of simple
operations.  For example if we have this expression:

$\sqrt[5]{\sqrt[4]{a^2 + \sqrt[3]{b / c}} - \sqrt[2]{d * e}}$

We could map it to this kind of graph:

```goat
             .-------------.                                                
            | E = ⁵√(D + C) |                                               
             '--+-------+--'                                                
               /         \                                              
              /           \                                                 
     .-------+----.     .--+-------.                                              
    | D = ∜(A + B) |   | C = ∛(b/c) |                                           
     '--+------+--'     '----------'                                             
       /        \                                               
      /          \                                              
 .---+--.     .---+------.                                                      
| A = a² |   | B = √(d.e) |                                                     
 '------'     '----------'                                                      
                                                                             
```                                                 

Every node represents an intermediary expression that is using only at most
a single radical operation.


# Recursive elimination of permutations from the full set

If we imagine that the solution to the quintic equation exists, and we
have its graph as a tree of simple operations.  We know from our previous
discussion that by smoothly looping the coefficients of the polynomial, we
can permute any of the root.  This mean that all the 5! = 120
permutations of five values are doable.  We can write them like that:

`(01234)`, `(01243)`, `(01324)`, `(01342)`, ...

Now the main element of the proof is the following observation:

If we consider only the set of all commutators of the 120 permutations,
then in our expression tree, all the leaf nodes will stay at the same value,
because they only have at most a single radical operation.

This means that if we restrict ourselve to the commutators subset, the leaf
node behaves exactly like a 'non radical' expression, and we can then remove
them all from the tree, making their parent nodes the new leaf nodes.

If we repeat this process again, we will eventually have a tree made of
a single node for the final expression, with no radical.  At this point
the set of permutations should only contain the identity.


# Final part of the proof using python

All we have to do now to prove that there is no solution to the quintic
equation is to show that we cannot go from the 120 permutations of five
elements to a single identity permutation by iteratively computing the
subset of all commutators.  To do that we can use a small python script:

```python
import itertools

def inv(p):
    """inverse of a permutation"""
    return sorted(range(len(p)), key=p.__getitem__)

def mul(a, b):
    """Apply permutation a on b"""
    return tuple([b[x] for x in a])

def chain(*ps):
    """Chain apply a list of permutations"""
    *_, last = itertools.accumulate(ps, mul)
    return last

def commutator(a, b):
    """Return commutator of two permutations"""
    return chain(a, b, inv(a), inv(b))

def permutation_group(n):
    """Return the set of all permutation of n elements"""
    return set(itertools.permutations(range(n)))

def all_commutators(g):
    """Return the set of all commutators of a set of permutations"""
    ret = set()
    for a, b in itertools.product(g, g):
        ret.add(commutator(a, b))
    return ret

s = permutation_group(5)

for i in range(5):
    print(f"{i}:{len(s)}")
    s = all_commutators(s)
```

Result:

```
0:120
1:60
2:60
3:60
4:60
```

We see that after the first step the set goes from 120 permutations to
only 60, but after that the set stays at 60 permutations, which means the
commutators subset is the same as the set itself.

This proves that no matter how complicated, there is no graph of an expression
that can be reduced to a single node using our process, which means that there
is no expression that supports any permutation of the roots, which means that
there is no solution in radical for the quintic equation.

[Abel's Theorem in Problems and Solutions]: https://www.maths.ed.ac.uk/~v1ranick/papers/abel.pdf
