---
categories:
  - code
  - C
  - game
  - doom
  - quake
  - C++
aliases: /entity-reference.html
title: Comparison of Doom 1, Quake, and Doom 3 entity references system
date: 2015-04-29
---


The architecture of a video game code is usually based around a list of
entities that represent all the game elements: players, enemies, etc.  The
engine iterates over this list and call the appropriate methods to update or
render the entities.  Since game elements get constantly created and destroyed,
the list is not static but need to be updated at each iteration.

The problem is that some entities keep references to others.  For example, an
enemy might have a reference to the player it is currently attacking.  If an
entity gets removed from the list, all the references to it need to be
invalidated, failing that it will introduce bugs in the code.  So in a sense
those references are weak (in the sense that they won't prevent the entity from
being destroyed, but must be able to find out that it has happened).

Since I am talking about video games written in C or C++ here, the language has
no support for weak references, and so we have to find some other ways.

Let's imagine we have three entities in the game (A, B and C), and C has a
reference to A (maybe C is an enemy and A its target).

Imagine we use a basic implementation, where entities are stored in a linked
list, and references are done with normal pointers.

When an entity update function is called, the object can decide to remove
itself from the list.  We risk to run into this situation:

    Step 1: entity A update function is called,
            A decides that it should be
            removed from the game.

      +---------------+
      v               |
    +---+   +---+   +---+
    | A |-->| B |-->| C |
    +---+   +---+   +---+
      ^


    Step 2: The engine removes A from the list and
            release the memory, the update continue
            with object B, and then object C.
            At this point C might try to access A,
            resulting at best in a segmentation fault,
            at worst into strange random behaviors
            (those bugs are very hard to track!)

      +---------------+
      v               |
    invalid +---+   +---+
    pointer | B |-->| C |
            +---+   +---+
                      ^


# Doom 1

In the original [DOOM] game, the solution used is very simple:  when an entity
dies, it is not immediately removed from the list, but instead it goes into the
'dead' state (this is implemented by having the entity state function points to
-1).  Only at the next iteration will the entity actually be removed from the
list.

This gives all the other entities the chance to update their references to NULL
if they point to an entity about to get deleted.

    Step 1:  entity A decides that is should die,
             so it flags itself as dead.

      +---------------+
      v               |
    +---+   +---+   +---+
    | A*|-->| B |-->| C |
    +---+   +---+   +---+
      ^


    Step 2:  when C update function is called,
             C find out that A is dead, and
             so remove its reference to it
             (make it point to the NULL address).

    +---+   +---+   +---+
    | A*|-->| B |-->| C |
    +---+   +---+   +---+
                      ^

    Step 3:  A is updated again, since it was
             flagged as dead, it can be safely
             removed from the list.

            +---+   +---+
            | B |-->| C |
            +---+   +---+
       ^


This implementation is simple, but you have to be careful: every entity holding
references to a dead entity need to checked at each iteration that the
referenced object is still valid.  Moreover, an entity dead state can only be
set from the entity update function.

Imagine the following state:

              +-------+
              v       |
    +---+   +---+   +---+
    | A |-->| B |-->| C |
    +---+   +---+   +---+
      ^

If the update method of A sets the B state to dead, then B will immediately
removes itself from the list without leaving a chance to C to update its
reference.

As a side note: this is more or less the technique I used in my video game
[voxel invaders].  Though it works pretty well, a few times I spent hours
trying to figure out strange bugs that were due to the fact that an entity
forgot to check and update its references.

# Quake

In [Quake 1], the code is a bit different.  The entity list is preallocated
with a preset value of max entities.  Note: apparently someone at id software
(maybe John Carmack?) thought the max number of 768 might be too low.  In the
code we can see:

    #define MAX_EDICTS 768  // FIXME: ouch! ouch! ouch!

When an entity is deleted, it is simply flagged as such, so that the engine
will skip it.  If we need to create a new entity, we loop over the array and
find a free spot.  Now, the trick is that we also keep track of the time when a
slot was freed, and only allow to reuse a slot two seconds after it has been
freed.  What this mean is that other entities in the game gets two seconds to
update their references to dead entities.

    Step 1: at t = 3s, the entity A update method
            decides that it should be removed
            from the game.  It clears itself
            in the array, and set its timer to 3.

      +-------+
      V       |
    +---+---+---+--
    |t=3| B | C |
    +---+---+---+--
      ^


    Step 2: eventually (but no more than 2 seconds
            later), the entity C find out that A
            slot has been cleared, and so nullify
            its reference to it.

    +---+---+---+--
    |t=3| B | C |
    +---+---+---+--
              ^

    Step 3: at t > 5s, the empty slot is reused
            for a new entity.

    +---+---+---+--
    | D | B | C |
    +---+---+---+--


This solution is also quite simple to implement and give more flexibility to
the game: we do not have to update the references immediately, also it doesn't
matter if an entity get deleted from an other entity update method.  There is
till the danger that an entity might not update its references during the
window time.  An other problem is that if we create and destroy a lot of
entities quickly, then most of the slots get unused (in fact in quake, there is
an explicit relaxation of the window period during level initialization to
prevent this problem).


# Doom 3

In [DOOM III], a more robust solution is used.  The idea is similar to quake,
except that every reference to an entity also stores an id that uniquely
identify the entity (an incremented counter).  If the slot pointed to has been
reused for an other entity, then we can figure it out by comparing the entity
id and the reference id.

In fact, in Doom 3 code, things are a bit more complicated, since the unique id
is not stored in the entity itself, but in a separated array (I suppose so that
we do not waste ids for entities that cannot be referenced anyway).


    State 1: all entities have a unique id
             assigned to them.  C keeps a pointer
             to A (in fact it is just an index in
             the entities array), but also the id
             of A (1).

      +-------+ 1
      V       |
    +---+---+---+--
    |A 1|B 4|C 2|
    +---+---+---+--


    State 2: A got removed from the list.
             C can figure out that its reference
             is invalid since the slot is empty.

      +-------+ 1
      V       |
    +---+---+---+--
    |   |B 4|C 2|
    +---+---+---+--


    State 3: the free slot has been reused by a
             new entity with id 5.  C does not
             need to update its reference, since
             it can still figure out that it is
             invalid by comparing the entity id
             (5) with the one he expects (1).

      +-------+ 1 (1 != 5)
      V       |
    +---+---+---+--
    |D 5|B 4|C 2|
    +---+---+---+--


This solution is more robust than the two previous ones.  We can keep
references to entities around for as long as we want.  As long as the entities
ids are really unique, we can always tell that a reference is invalid.  From
the code we can tell that Doom 3 support spawning 1,048,576 entities before an
error occurs.

The drawback is that it makes the code a bit more complicated, since we need to
keep two pieces of information to represent a reference to an entity.

[voxel invaders]: http://noctua-software.com/voxel-invaders
[DOOM]: https://github.com/id-Software/DOOM
[Quake 1]: https://github.com/id-Software/Quake
[DOOM III]: https://github.com/id-Software/DOOM-3
