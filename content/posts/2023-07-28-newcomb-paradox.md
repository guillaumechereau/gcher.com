---
title: My Solution to the Newcomb Paradox
date: 2023-07-28
---

The Newcomb paradox is stated as a fictional situation where a player enters a
room with two opaque boxes, A and B.  A always contains a small amount of
money (let say $100), and B contains either nothing, either a larger amount of
money (let say $110) [^1].  The player gets to pick one or both boxes and
leave the room with them.

[^1]: Usually the paradox says that B contains a much larger amount of money,
    but here I changed the amounts so that my 'solution' later beat on
    average just picking the box B.  We could also state the problem
    with a single box that you can pick or not, and that contains either
    nothing, either some money.

The catch is that before the player enters the room, an 'all knowing' entity
prepared the boxes and filled the box B with money only if it predicted that
the player will **not** pick the A box.  In this situation, what is the best
strategy?

Most people (I think?) will agree that the player should pick the box B only.
Since picking both boxed will mean that the box B will be empty, the choices
are:

  - A only: win $100
  - B only: win $110
  - A and B: win $100

One counterargument is that since when the player enters the room, both boxes
are already set up, it is always better to pick both A and B.  If we call $X
the amount of money in the B box as we enter the room, the choices are:

  - A only: win $100
  - B only: win $X
  - A and B: win $100 + $X

The argument being that since X is already fixed, its value cannot depend
on the choice we make.

Some people might say that it is simply impossible for an entity to be
'all knowing' so this paradox does not make much sense.


My take is that in order to predict the player choice in advance, the all
knowing entity needs to run the equivalent of a simulation of the world on a
scale big enough to contains the player and its direct environment.  The
simulated player cannot tell that he is part of the simulation, so he needs to
consider two possibilities:

  - Either this is the simulated experiment, running prior to the real one,
    that will decide the content of the box B: in this case, the best
    strategy is to pick the box B only.
  - Either this is the real experiment, and at this point the content of
    the box B is already set: in this case the best strategy is to
    pick both boxes.

Since the entity is all knowing, it should be impossible to determine in which
stage of the experiment we are in order to use a different strategy.

However, no matter how powerful the entity is, it cannot predict the outcome
of a quantum wave function collapse, and we can use this to our advantage.

Here is what I think is the best strategy, that earns on average more than
just taking the B box:

Before entering the room, the player should prepare a device that allows to
make a quantum random choice (For example, a photon source and a half-silvered
mirror that has a 1/2 probability of reflecting it).

Once in the room, the player uses the device to give the equivalent of a 'coin
flip'.  Depending on the result of the flip, he picks either the box B, either
both boxes.

We now have four possibilities, depending on the coin flip result in the
simulated experiment and the real one.  Assuming the player picks both boxes
on a head flip.   The expected gains are:

  - Face / Face   : $110
  - Face / Head   : $210
  - Head / Face   : $100
  - Head / Head   : $100

Which gives an average of $130, beating the strategy of picking the B box
only.
