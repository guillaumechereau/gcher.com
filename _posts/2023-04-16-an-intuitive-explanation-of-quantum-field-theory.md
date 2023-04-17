---
layout: post
title: An Intuitive Explanation of Quantum Field Theory
categories: Physics
published: false
---

This post will be my attempt to explain visually what is Quantum Field Theory.

This idea came to me after reading two books: "QED: The Strange Theory of Light
and Matter" by Richard Feynman, and "Quantum Field Theory in a Nutshell", by A.
Zee (that I only started).  Both books use path integrals, but in a different
way, and this is what motivated this post.

Note: this post became longer than I hoped.  Here is a tldr: I am trying to
show that computing the path integral of a single particle moving through
space-time (relativistic) is similar to computing the path integral of a
lattice of coupled particles connected by springs but moving through space
only.  Since the relativistic paths can 'go backward in time', when trying to
define the partial sum of the paths at a fixed time, we need to account for the
segments of paths coming from the future, making the 'pseudo' Lagrangian
behaves similarly to many particles and anti particles.

Big disclaimer: I am not a scientist and this is going to be extremely hand
wavy.  I won't use any unit and just ignore mass and spin.  For the purpose
of this post, a particle has just a position.  I'll ignore normalization and
many other details.

# Path Integral for Quantum Mechanics.

I think most people reading this have some idea of what quantum mechanic is in
comparison to classical mechanics: the absolute position and speed of particles
is replaced by a wave function that is related to the probability of finding
the system in a given state.  The Schrodinger equation then gives us the time
evolution of the wave function.

There is an other mathematically equivalent way to describe quantum mechanics,
using the so call 'path integral', as popularized by Feynman (though I believe
the idea originally came from Dirac).  I am going to present a toy model of
the path integral using a discrete grid to make it easier to think about.
Let's imagine that we have a single particle in one dimension.  Its position
(X) can only take discrete values from 0 to -let say- 10.  The particle starts
at the position A and we want to compute the probability to observe it at
the position B after a interval of time T (that we also assume to be discrete).
Let say we have 8 time intervals:

![Grid](/static/imgs/qft/path1.png)

To compute this value, we follow this algorithm:

1. List all the possible paths that connect the point A to the point B, by
   picking any position for each time.  Here I represented two possible paths,
   the actual number of paths is 10^8:

![Two paths](/static/imgs/qft/path2.png)

2. For each path we compute a complex number (the amplitude) with the following
   rule: we start at the point A with the value 1 (that we can also represent
   as a unit vector of angle zero).  Then we follow the path and at each step
   we consider the kinetic energy T ~ x'^2 and the potential energy V(x),
   and we rotate the value by an angle equals to T - V.

3. Finally we sum all the amplitudes for all the paths.  The norm of the sum
   gives us the probability we are looking for.

# Retrieving the present time

The path integral formulation is not commonly talked about (at least to
non physicist) and I think it's in part because it seems totally impractical
to compute a infinite number of paths, and also because it is not obvious
how this relate to our 'normal' perception of the world as some kind of
state machine, going from a time T to a time T + 1.

In order to simplify the computation of a very large number of paths we can
make the following observation: all the partial segments of path will be
used by many paths.  How can we avoid recomputing them each time?  The
trick is to realize that is we draw a line at a given time, all the paths
will cross it at one position.  If we precomputed the partial sum of all
the paths below this line for all positions, we can use it at a starting
value for our amplitude computation.  That way we can see that by iterating
the time step by step we only need to keep track of N complex numbers, where
N is the number of possible position of the particle.

In this image I try to show how that works: if we already computed the sums
of all the path at each point at time ta, we can easily compute the sum of
all the path at each point at time ta + 1:

![Optimizing the path sums](/static/imgs/qft/path3.png)

This gives us two things:

- A representation of the 'present' time, as a function mapping each possible
  state to a complex number.  We can call this psi(x).

- A iteration function to evolve psi through time: iter(psi(x), dt) => psi(x)

This is not obvious and I won't do the math, but we can show that, using the
computation rule I loosely explained, the iteration function becomes the
Schrodinger equation for a single particle in the limit of an infinitesimal
grid.

# A more complicated system

Let see quickly how the path integral would work for a system of particles.
Specifically we want to consider a one dimension lattice of particles
that can each only move vertically, while being connected to their neighbor
with little springs, like so:

![Lattice of particles](/static/imgs/qft/lattice.png)

Now a single 'state' of the system is not just one X position, but an
array of Y position, one for each particle.  To keep things simple, if we
assume that we have 100 particles and we can only have 5 possible Y values,
the number of states become N = 5^100.

The number of possible path remains T^N.  We can still represent a path in a
diagram, but we need to remember that each X axis position now sets the
position of all the particles.

The book from A. Zee almost immediately starts with the study of such a system,
and then tell us that the lattice excitation can be seen as the evolution
or a single relativistic particle.  No explanation is given to where this
relation comes from.

Here I will give a try at showing this relation.  First lets see a bit
how the amplitudes are computed for this system, still assuming a toy
model of discrete position and time.

The rule to compute the amplitude for a given path remains the same: we start
at the beginning and for each step we consider the start and end states and
compute the V - T value to use as an angle increment.  Let see some examples
for different transitions amplitude we get for some special patterns.  Note
that I consider the average of the potential between the two states:


![Transitions values for the lattice](/static/imgs/qft/lattice-transitions.png)

With great mathematical effort we could find and solve the Schrodinger equation
for this system, but it's not our concern here.  We'll just assume that we have
an infinite computer at our disposition that can sum all the path integrals if
needed.  Once again I need to stress that the goal here is just to build some
intuition.


# Non relativistic particles

Now we can finally start to explain Quantum Field Theory.  We are going to
use the same framework of sum over paths, except that now the particles are
relativistic: instead of moving in space with regard to time, they move
in space-time with regard to their proper time τ(tau).

Here is how it looks like for a few samples of paths:

    TODO add an image.

The number of path is infinite, even if we restrict ourself to a finite grid in
both space and time, since a path can take an unbounded number of steps before
reaching the point B.

The other change is that the kinetic energy is no more proportional to the
squared speed, but also include a factor for the speed *in time*.  For a single
dimension particle, Instead of:

    T ~ (dx/dt)²

We use:

    T ~ (dx/dτ)² - (dt/dτ)²

Can we still fix the time and compute a partial sum of all the paths crossing
the time for all the positions?  Not really, because now a single path
can potentially cross the fixed time line several times, in both directions.

We can still try to make it work.  We have to track separately the partial
sum for all the paths crossing once, then all the paths crossing twice, and
so one, in both direction.  This can be represented by assigning for each
position the number of times the time line has been crossed, with a number
that can be positive or negative.  The wave function that used to assign
a complex number to each state, like this:

    φ(S) -> C

Now assigns a complex number to each possible function that assigns an
integer to a state:

    φ(f(S) -> I) -> C

To visualise it, if we use limit the number of time a path can cross the
line to only 5 possible numbers (-2, -1, 0, +1, +2), and we assume only
100 possible positions in the one dimension space.  The number of inputs
for the wave function is N = 5^100.  Coincidently the same number we got
for the non relativistic lattice of 100 particles, so we already start to
see where the similarity comes from.

Even so it's still not clear that it will work because we don't
know in advance if a path will eventually cross the line backward in time
The solution is to also support tracking paths in reverse.
