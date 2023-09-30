---
categories: [C]
title: Auto function binding in C
date: 2023-09-30
---

Let say you want to add some script support to a C program, like lua or maybe
even something simpler where all you can do is call a function with some
given arguments:

```C
#include <stdio.h>

void my_func(int x, const char *s)
{
    printf("my_func called with x=%d, s=%s\n", x, s);
}

int main()
{
    // Will call my_func with 2 and "hello".
    script_call("my_func", 2, 'hello'");
}
```


Since C does not support reflection, there is no way to implement the
`script_call` function without some extra work.

The common solution involves defining a wrapping function that converts
arguments from the internal script type to the function input types, and then
call it:

```C
int my_func_wrapper(int nb_args, const char **args)
{
    if (nb_args != 2) return -1;
    int x = atoi(args[0]);
    const char *s = args[1];
    my_func(x, s);
    return 0;
}
```

Then we need to register this wrapper to the script engine:

```C
script_register_function(my_func_wrapper, "my_func");
```

And now the script has all it need to call our function.  Scripting languages
like lua all provide a way to register functions like that.

The problem is that when you have a lot of functions to wrap this can become
tedious to write all the wrappers, specially since they usually involve
more complicated error checking.  This is why many project use some sort
of automatic function binding that automatically write the wrappers.

One approach is to use an external tool like [SWIG] that can parse the C code
and automatically generates the wrappers.  This works well, but requires to
add a build step and reliance on auto generated code.

Some other projects (like godot) use C++ variadic template feature to
automatically generate the wrapper functions using meta programming.

Even though C does not support template, we can abuse a little bit the
preprocessor to achieve something close enough in pure C.  The idea is
that we only add one macro below the function to automatically define
the wrapper:


```C
void my_func(int x, const char *s) { ... }
BIND(my_func, int, string);
```

How to write the BIND macro?  The biggest issue is that the C preprocessor does
not allow to iterate on a list of arguments.  Fortunately we can use this
famous trick to achieve something close:

```C
// Basic FOREACH(action, lastaction, args...) macro.
#define F0(A, L)
#define F1(A, L, X0) L(0,X0) 
#define F2(A, L, X0, X1) A(0,X0) L(1,X1)
#define F3(A, L, X0, X1, X2) A(0,X0) A(1,X1) L(2,X2)
// Add as many as needed.

#define GETMACRO(_0, _1, _2, NAME, ...) NAME 
#define FOREACH(A, L, ...) \
    GETMACRO(__VA_ARGS__, F3, F2, F1, F0) \
        (A, L, __VA_ARGS__)
```

Now using FOREACH we can call a macro on each arguments of an other macro.
It also supports using a different call for the last argument, which will be
needed to generate comma separated lists.


The second thing is that we need to specify default conversion between the
argument types we pass to the BIND macro and the C types.  Here we only
do it for 'int' and 'string':

```C
int to_int(const char *s)
{
    return atoi(s);
}

const char *to_string(const char *s)
{
    return s;
}
```

Since a 'string' corresponds to a `char *`, we also need a way to
map type names to C type.  We do by using the convention that `type_<name>`
should be defined as the C type:

```C
#define type_int int
#define type_string const char *
```


Finally the BIND macro can now be written like that:

```C
// BIND macro.
#define GET_VAL(idx,type) type_##type a_##idx = to_##type(args[idx]);
#define PASS_VAL(idx,type) a_##idx,
#define PASS_VAL_LAST(idx,type) a_##idx
#define BIND(func, ...) \
    static void func##_(int nb_args, const char **args) { \
        FOREACH(GET_VAL, GET_VAL, __VA_ARGS__) \
        func(FOREACH(PASS_VAL, PASS_VAL_LAST, __VA_ARGS__)); \
  }
```


And this allows to make semi automatic binding of C functions in a cross
platform manner, without relying on external tool or foreign function call
libraries.


I am only aware of one library using this trick: [LuaAutoC], used by
[darktable] for their Lua binding.  Here is a [post](autoc-tool) from the
creator of LuaAutoC that goes into detail of their actual implementation.


[SWIG]: https://swig.org
[LuaAutoC]: https://github.com/orangeduck/LuaAutoC
[darktable]: https://github.com/darktable-org/darktable
[autoc-tool]: https://theorangeduck.com/page/autoc-tools
