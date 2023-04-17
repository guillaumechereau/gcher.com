---
layout: post
title:  C++ patterns using plain C
categories: C++ C code pattern
redirect_from: /cpp-patterns-using-plain-c.html
---

So that's it, you read [this article][source1], and this [post][source2], and
[this][source3], and [this][source4], and you spent too many time trying to
debug 8 pages long compilation error messages, and you also spent hours trying
to follow your code path involving operator overloading, copy constructors, and
virtual class methods called from constructors or whatnot.  You have had enough
of it.  Yout think it's time for you to stop using C++ altogether and return to
simple, frank, easy to debug, plain, C.

[source1]: http://yosefk.com/c++fqa/defective.html
[source2]: http://c0de517e.blogspot.tw/2014/06/where-is-my-c-replacement.html
[source3]: http://www.fefe.de/c++/c++-talk.pdf
[source4]: http://article.gmane.org/gmane.comp.version-control.git/57918/

But there is a problem.  After all those years spent mastering C++, you have no
idea how to implement things like containers or classes in C.  features.  You
know it's possible, because after all the linux kernel is doing all those
things, but you are not really sure how they achieve it, or how to do it in
your own project.

In this post I am going to show a few examples of simple C++ code, and for each
propose one **possible** implementation using plain C.  I try to focus on the
things I think people with a C++ background would miss the most when using C.
The goal is not to imply that C is better than C++ (in most of those examples
the C++ code is actually cleaner), but merely to show that all the alleged
benefices of C++ are achievable in C with little effort.

## 1. BASIC CLASS

The simplest form of a C++ class: two subclasses of one base class, each one
having a specialized version of a virtual method.  No attributes involved.

Here the trick is simply to store a pointer to the function as an attribute of
each instance of the class (in C it's actually a structure).  The advantage in
the C version is that you could specialize individual instances, the drawback
is that each virtual method requires an extra attribute in the base structure.

C++ version:

    #include <iostream>
    
    class Base
    {
    public:
        virtual char func() = 0;
        virtual ~Base() {}
    };
    
    class A : public Base
    {
    public:
        virtual char func() {return 'A';}
    };
    
    class B : public Base
    {
    public:
        virtual char func() {return 'B';}
    };
    
    int main()
    {
        Base *o1 = new A();
        Base *o2 = new B();
        std::cout << o1->func() << " " << o2->func() << std::endl;
        delete o1;
        delete o2;
    }

C version:

    #include <stdlib.h>
    #include <stdio.h>
    
    typedef struct Base {
        char (*func)();
    } Base;
    
    static char a_func() { return 'A'; }
    static char b_func() { return 'B'; }
    
    static Base *create(char (*func)())
    {
        Base *ret = malloc(sizeof(*ret));
        ret->func = func;
        return ret;
    }
    
    int main()
    {
        Base *o1 = create(a_func);
        Base *o2 = create(b_func);
        printf("%c %c\n", o1->func(), o2->func());
        free(o1);
        free(o2);
    }


## 2. ADVANCED CLASS

A bit more advanced, for the case when we have more virtual methods, and we
don't want to waste memory by storing all the pointers each time.  Instead,
each instance keeps a pointer to a table of pointers to the virtual methods
(the so called V-table).  Instances of the same class share the same V-table to
save space.  Just to show that is is possible, I also added an attribute to the
base class.  Note that in the C version, the methods need to explicitly have a
`this` argument.

C++ code:

    #include <iostream>
    
    class Base
    {
    public:
        Base(int x) {m_x = x;}
        virtual int func1() = 0;
        virtual int func2() = 0;
        virtual ~Base() {}
    protected:
        int m_x;
    };
    
    class A : public Base
    {
    public:
        A(int x): Base(x) {}
        virtual int func1() {return m_x * 10;}
        virtual int func2() {return m_x * 11;}
    };
    
    class B : public Base
    {
    public:
        B(int x): Base(x) {}
        virtual int func1() {return m_x * 20;}
        virtual int func2() {return m_x * 21;}
    };
    
    int main()
    {
        Base *o1 = new A(1);
        Base *o2 = new B(1);
        std::cout << o1->func1() << " " << o2->func1() << std::endl;
        std::cout << o1->func2() << " " << o2->func2() << std::endl;
        delete o1;
        delete o2;
    }

C code:

    #include <stdlib.h>
    #include <stdio.h>
    
    // Base class
    
    typedef struct Base {
        struct Type *type;
        int x;
    } Base;

    // The V-Table type
    typedef struct Type {
        int (*func1)(Base *this);
        int (*func2)(Base *this);
    } Type;
    
    int base_func1(Base *this) {return this->type->func1(this);}
    int base_func2(Base *this) {return this->type->func2(this);}
    
    // class A
    int a_func1(Base *this) { return this->x * 10;}
    int a_func2(Base *this) { return this->x * 11;}
    
    static Type A = {
        .func1 = a_func1,
        .func2 = a_func2,
    };
    
    // class B
    int b_func1(Base *this) { return this->x * 20;}
    int b_func2(Base *this) { return this->x * 21;}
    
    static Type B = {
        .func1 = b_func1,
        .func2 = b_func2,
    };
    
    Base *create(Type *type, int x)
    {
        Base *ret = malloc(sizeof(*ret));
        ret->type = type;
        ret->x = x;
        return ret;
    }
    
    // Main
    
    int main()
    {
        Base *o1 = create(&A, 1);
        Base *o2 = create(&B, 1);
        printf("%d %d\n", base_func1(o1), base_func1(o2));
        printf("%d %d\n", base_func2(o1), base_func2(o2));
        free(o1);
        free(o2);
    }


## 3. MORE ADVANCED CLASS

Now, we also introduce attributes of the subclasses.  This means that we have
to define a different storage structure for each subclass.  We put the parent
class storage structure as a first attribute, that way we can safely cast from
a parent type to a children type.  This is I think the closest to what a C++
compiler would actually do under the hood.

C++ code:

    #include <iostream>
    
    class Base
    {
    public:
        Base(int x) {m_x = x;}
        virtual int func() = 0;
        virtual ~Base() {}
    protected:
        int m_x;
    };
    
    class A : public Base
    {
    public:
        A(int x): Base(x) {}
        virtual int func() {return m_x * 10;}
    };
    
    class B : public Base
    {
    public:
        B(int x): Base(x), m_y(2 * x) {}
        virtual int func() {return m_x * 20 + m_y;}
    protected:
        int m_y;
    };
    
    int main()
    {
        Base *o1 = new A(1);
        Base *o2 = new B(1);
        std::cout << o1->func() << " " << o2->func() << std::endl;
        delete o1;
        delete o2;
    }

C code:

    #include <stdlib.h>
    #include <stdio.h>
    
    // Base class
    
    struct Base;
    
    typedef struct VTable {
        int (*func)(struct Base *this);
    } VTable;
    
    typedef struct Base {
        VTable *vtable;
        int x;
    } Base;
    
    
    int base_func(Base *this) {return this->vtable->func(this);}
    
    Base *base_new(VTable* vtable, int size, int x)
    {
        Base *ret = calloc(1, size);
        ret->vtable = vtable;
        ret->x = x;
        return ret;
    }
    
    
    // class A
    typedef struct A {
        Base base;
    } A;
    
    int a_func(Base *this) { return this->x * 10;}
    
    static VTable A_VTable = {
        .func = a_func,
    };
    
    A *a_new(int x)
    {
        return (A*)base_new(&A_VTable, sizeof(A), x);
    }
    
    // class B
    
    typedef struct B {
        Base base;
        int y;
    } B;
    
    int b_func(Base *base) {
        B *this = (B*)base;
        return base->x * 20 + this->y;
    }
    
    static VTable B_VTable = {
        .func = b_func,
    };
    
    B *b_new(int x)
    {
        B *ret = (B*)base_new(&B_VTable, sizeof(B), x);
        ret->y = x * 2;
        return ret;
    }
    
    // Main
    
    int main()
    {
        Base *o1 = (Base*)a_new(1);
        Base *o2 = (Base*)b_new(1);
        printf("%d %d\n", base_func(o1), base_func(o2));
        free(o1);
        free(o2);
    }


## 4. BASIC TEMPLATE

For the very simple cases, when we just need to have different version of a
function for different types, we can just use a macro.

C++ code:

    template <class T>
    T max(T x, T y)
    {
        return x > y ? x : y;
    }

C code:

    #define max(a, b) ({ \
      __typeof__ (a) _a = (a); \
      __typeof__ (b) _b = (b); \
      _a > _b ? _a : _b; \
      })

## 5. ADVANCED TEMPLATE

The previous example was enough for simple cases, but what about if we there is
a whole module that we want to compile for different types.  For example a
vector library might need to work with `float` or `double`.

Here is how I would do that:  use a macro definition to set the type.  If the
macro is not set, then set it to all the wanted values and let the file include
itself.  Also make sure that the function names are postfixed with the type,
since in C all the function names are unique.

C++ code:

    template <class T>
    T add(T x, T y)
    {
        return x + y;
    }

    template <class T>
    T sub(T x, T y)
    {
        return x - y;
    }

C code:

    #ifndef T
    #  define T int
    #  include __FILE__
    #  undef T
    #  define T float
    #  include __FILE__
    #  undef T
    #  define T double
    #  include __FILE__
    #else
    
    #define FUNC_(name, type) name ## _ ## type
    #define FUNC(name, type) FUNC_(name, type)
    
    static inline T FUNC(add, T)(T x, T y)
    {
        return x + y;
    }
    
    static inline T FUNC(sub, T)(T x, T y)
    {
        return x - y;
    }
    
    #endif

C code usage:

    int     x_int    = sub_int(2, 1);
    float   x_float  = sub_float(2.1f, 1.0f);
    double  x_double = sub_double(2.1, 1.0);


## 6. LIST

One nice way to implement lists in C is to use linked lists, by adding the
pointers directly into the structure we want to put in the list.  The linux
kernel has a generic implementation of this.  Here I show how it works using
the utlist library.

C++ code (using std::list):

    #include <list>
    #include <iostream>
    
    class A
    {
    public:
        A(int x) : x(x) {}
        int x;
    };
    
    int main()
    {
        std::list<A*> list;
        for (int i = 0; i < 10; i++) {
            list.push_back(new A(10));
        }
    
        for (   std::list<A*>::iterator it = list.begin();
                it != list.end();
                it++) {
            std::cout << (*it)->x << std::endl;
        }
    }

C code (using utlist):

    #include <stdlib.h>
    #include <stdio.h>
    #include "utlist.h"
    
    typedef struct A {
        struct A *prev;
        struct A *next;
        int x;
    } A;
    
    int main()
    {
        A *list = NULL, *item;
        int i;
        for (i = 0; i < 10; i++) {
            item = calloc(1, sizeof(A));
            item->x = i;
            DL_APPEND(list, item);
        }
    
        DL_FOREACH(list, item) {
            printf("%d\n", item->x);
        }
    }

