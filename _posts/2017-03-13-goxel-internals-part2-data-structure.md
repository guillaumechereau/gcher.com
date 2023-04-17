---
layout: post
title:  "Goxel Voxel Editor Internals - Part 2: voxel data structure"
categories: goxel voxel code C C99
redirect_from: /goxel-internals-part2-data-structure.html
---

In this second part of my series about goxel voxel editor internals, I will
talk about the way voxel data is stored in memory.

As usual, all the code is open source and accessible on the [github repo].

The voxel models are composed of many individual voxel, each of them having
a color.

For the structure of the voxel I wanted:

- **Optimized memory usage**: if a model is large but composed of few voxels
  I don't want to use extra memory for the empty spaces.

- **Spatial and temporal locality of the data**: this is important to take
  advantage of CPU cache.  The algorithms running on the voxel should not
  have to jump around the memory too much.  Imagine for example that the
  voxel are stored in a simple 3d array of size `W * H * D`.  The distance
  between two adjacent voxels in memory can be as high as `W * H`.  This is
  not optimal if `W * H * size(voxel)` exceed the CPU cache size.

- **Eventual future GPU storage**: For the moment the GPU is only used for
  actual rendering, so it means I have to spend time sending the voxel data
  to the GPU.  Eventually in the future I would like to be able to store the
  voxels directly in the GPU memory.  Can I map the data into 2d textures?

- **Unlimited model size**.

So let see how it is done in goxel.  The file we want to look at is [goxel.h],
as it contains the C structure definitions.

#### Blocks

The first structure of interest if `block_data_t`:

    #define BLOCK_SIZE 16

    typedef struct block_data block_data_t;
    struct block_data
    {
        int         ref;
        uint64_t    id;
        uvec4b_t    voxels[BLOCK_SIZE * BLOCK_SIZE * BLOCK_SIZE];
    };

This is the primordial voxel storage structure in the code.  The important
part is the matrix of 32 bits RGBA values, of fixed size 16x16x16 (attribute
`voxels`).

Why the size 16x16x16 (= 4096)?  It's because this number has an interesting
property: it can be expressed both as a cube and a square: 16^3 = 64^2.
That way a block data can be used both as a 3d voxel volume of size 16, or a 2d
image of size 64.  This is used for example when saving a file, to compress
the data as png images.

Now, `16^3` was not the only possible choices.  I let you convince yourself
that any square number would have worked.  If we restrict ourself to power of
two numbers, this mean we could have used for the size of the cube any number
of the form: `2^(2*n)`: `1`, `4`, `16`, `64`, `256`, ...

I picked 16 because I thought it would be a good compromise between memory
and CPU usage, we'll see why.

A block data is itself included into a structure of type `block_t`:

    typedef struct block block_t;
    struct block
    {
        UT_hash_handle  hh;
        block_data_t    *data;
        vec3_t          pos;
        int             id;
    };

The block structure adds a few things around the block data:

- A 3d position (argument `pos`).
- A hash handle (`hh`), so that it can be stored into a hash table (if you come
  from C++, don't be scared by the fact that we store container structure into
  the contained type!)
- An id.  We'll ignore this in this post.

To understand why we are using two structures instead of just one, is because
voxel blocks in goxel are using *copy on write* (COW).  Here is how it works:

When we create a new `block_t`, we also allocate memory for its data (as a
malloced `block_data_t`), and we set the reference counter (the `ref` attribute
of `block_data_t`) to one, meaning that only on block is using this data.

If we call the function `block_t *block_copy(const block_t *other)` to make a
copy of a block, we don't actually copy the data, but instead we just have the
two blocks points to the same `block_data_t`, and to remember that this data is
used by two blocks, we also increments its ref counting attribute.

If we modify a block (for example by calling the function `block_fill`),
there are two possible cases:

- If the block data pointed to by the block has a reference counting of one,
  then we know that only this block is using the data, and we can modify it
  directly.
- If instead the data reference counter is greater than one, it means an other
  block is sharing the data, and so we cannot directly modify it.  Instead
  we first copy the data into a new `block_data_t` and decrease the ref
  counting of the original data.

The function doing this is called `block_prepare_write` in [block.c].
(It also does some bookkeeping of id numbers, but we are not interested in
that now).

    static void block_prepare_write(block_t *block)
    {
        if (block->data->ref == 1) {
            return;
        }
        block->data->ref--;
        block_data_t *data;
        data = calloc(1, sizeof(*block->data));
        memcpy(data->voxels, block->data->voxels, N * N * N * 4);
        data->ref = 1;
        block->data = data;
        block->data->id = ++goxel->next_uid;
        goxel->block_count++;
    }

Using copy on write is very powerful, because it means that making duplicates
of blocks is basically free in term of memory usage.

#### Meshes

A block has a fixed size of `16^3` voxels.  But of course we want to support
larger models, so we need to have a data structure that contains several
blocks.  In goxel I use a hash table (hence the `hh` attribute in `block_t`)
indexed by block position.  The hash allows to quickly find a block given
a position, and we don't store empty blocks to save space.  This structure
is quite flexible because we don't have to waste memory on the empty areas of a
model.  But there is an issue: I mentioned that we often need to access the
neighboring voxel of a given voxel. When the voxel is inside a block this
is not a problem we can just check the memory around it, but on the side of
a block, we would have to access the adjacent blocks, and the code would
be full of special branches for this case.  It would also break the spacial
locality of the algorithm since the block data might not be near each other.

    Logical blocks position
    +-----+-----+-----+
    |16^3 |16^3 |16^3 |
    |     |     |     |
    +-----+-----+-----+

    Internal memory structure
    +-----+       +-----+   +-----+
    |16^3 |------>|16^3 |-->|16^3 |
    |     |       |     |   |     |
    +-----+       +-----+   +-----+

To fix this, I positions the blocks in such a way that each adjacent blocks
*overlap* by a distance of two voxels.

    <----->
        <----->
    +---+-+---+
    |   |x|   | Two blocks overlap.
    |   |x|   | X are shared voxels.
    +---+-+---+

Now, all we have to do is ignore the borders when rendering a block.  This
work only if we don't use a neighbor value to update a given block, in that
case we would still need to check the other blocks.

Because of this, a block that contains 16^3 voxels, actually only represents
a 14^3 area of a mesh.  This is an annoyance, but really makes many parts
of the code easier.

The mesh structure looks like this:

    typedef struct mesh mesh_t;
    struct mesh
    {
        block_t *blocks;
        int next_block_id;
        int *ref;
        uint64_t id;
    };

The `block` attribute is actually a hash table of blocks.  Meshes also use copy
on write, albeit in a slightly more complicated way since I didn't want to use
a separate structure for the data.  This is why the ref counter attribute is a
pointer here: it is shared by all the meshes using the same blocks.  It
looks confusing but it really works the same.

#### Conclusion

We have seen how goxel internally stores the voxel data used in the models.
The copy on write mechanism, make duplicating a mesh basically a free
operation.  For example this is why it is quite easy to implement undo/redo in
goxel: at every step we can just make a copy of the mesh and store it in a
list.

To simplify the algorithm, we split the meshes into blocks of fixed size of of
`16^3`. Modifying part of a mesh only requires to update the blocks affected.
This is also a great optimization because we can now *cache* rendering data for
a given block.  I'll talk about this in the next part about OpenGL rendering.

[github repo]: https://github.com/guillaumechereau/goxel
[goxel.h]: https://github.com/guillaumechereau/goxel/blob/master/src/goxel.h
[block.c]: https://github.com/guillaumechereau/goxel/blob/master/src/block.c
