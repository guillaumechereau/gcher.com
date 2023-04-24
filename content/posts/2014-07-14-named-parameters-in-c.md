---
categories:
  - C
  - code
  - pattern
alias: /names-parameters-in-c.html
subtitle: material for the IOCCC
title: Named parameters in C
date: 2014-07-14
---


C99 introduced the concept of [designated initializers], that allows to
initialize a structure using the name of the fields, like this:

    struct MyStruct {int x; float y;};
    struct MyStruct a = {
        .x = 10,
        .y = 3.6,
    };

Here is a C macro that extends this syntax to function calls.

I present it as a curiosity, and I really wouldn't advise anybody to actually
use it in a project (except maybe for very special cases, like for example
emulating the interface of a language that accepts named arguments).

The macro is using a few tricks, notably the 'foreach' as described in this
[stack overflow question][foreach].

[**Edit**] As some people pointed out on the [hacker news thread], the macro
also uses a [statement expression], which is a GNU C extension, and so it would
probably not work with MSVC (I am not going to try).


    #define FE_0(a, a0, X)     a0(0, X)
    #define FE_1(a, a0, X, ...) a(1, X)FE_0(a, a0, __VA_ARGS__)
    #define FE_2(a, a0, X, ...) a(2, X)FE_1(a, a0, __VA_ARGS__)
    #define FE_3(a, a0, X, ...) a(3, X)FE_2(a, a0, __VA_ARGS__)
    #define FE_4(a, a0, X, ...) a(4, X)FE_3(a, a0, __VA_ARGS__)
    #define GET_MACRO(_0, _1, _2, _3, _4, NAME,...) NAME
    #define FOR_EACH(a, a0, ...) \
        GET_MACRO(__VA_ARGS__, FE_4, FE_3, FE_2, FE_1, FE_0) \
                  (a, a0, __VA_ARGS__)
    
    #define ARGS_STRUCT_ATTR(n, attr) union {attr, _##n;};
    
    #define ARGS_STRUCT(...) \
        struct { \
            FOR_EACH(ARGS_STRUCT_ATTR, ARGS_STRUCT_ATTR, \
                     __VA_ARGS__) \
        }
    
    #define ARGS_PASS(n, attr) _args._##n,
    #define ARGS_PASS0(n, attr) _args._##n
    #define PASS_STRUCT(...) \
        FOR_EACH(ARGS_PASS, ARGS_PASS0, __VA_ARGS__)
    
    #define CALL_NAMED_ARGS(func, args, ...) ({ \
        ARGS_STRUCT args _args = {__VA_ARGS__}; \
        func(PASS_STRUCT args); \
        })

And here is how we can use it:

    // A normal C function.
    float func(int x, float y)
    {
        printf("%d %f\n", x, y);
        return x + y;
    }
    
    // func2 will call `func` using named arguments.
    #define func2(...) \
        CALL_NAMED_ARGS(func, (int x, float y), __VA_ARGS__)
    
    int main()
    {
        float ret = func2(.y = 1.5, .x = 20);
    }

For the curious, here is what the macro expends to:

    float ret = ({
        struct {
            union {int x, _1;};
            union {float y, _0;};
        } _args = {
            .y = 1.5,
            .x = 20
        };
        func(_args._1,_args._0);
    });


The macro can only support up to 5 arguments (but, could easily be extended for
more).  One interesting thing is that if we omit a parameter it will be
automatically set to 0.  So let say you have a function like:

    void func(int x0, int x1, int x2, int x3, int x4);

And you want to call it by setting most of the arguments to 0.  You could do:

    #define func2(...) CALL_NAMED_ARGS(func, \
        (int x0, int x1, int x2, int x3, int x4), \
        __VA_ARGS__)
    
    func2(.x0 = 1);             // All 0 except x0
    func2(.x1 = 4, .x4 = 2);    // All 0 except x1 and x4
    func2();                    // All 0.

[designated initializers]: https://gcc.gnu.org/onlinedocs/gcc/Designated-Inits.html
[foreach]: http://stackoverflow.com/questions/1872220/is-it-possible-to-iterate-over-arguments-in-variadic-macros
[hacker news thread]: https://news.ycombinator.com/item?id=8029606
[statement expression]: https://gcc.gnu.org/onlinedocs/gcc/Statement-Exprs.html
