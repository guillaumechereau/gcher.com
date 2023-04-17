---
layout: post
title:  Unit Tests in C with the constructor attribute
categories: C code test
redirect_from: c-unit-tests-constructor-attribute.html
---

In one of my project, I used GNU [constructor attribute][1] to add some simple
unit tests to any C file.  I liked this approach, but I don't see many open
source projects using anything similar, so I though I'll describe it here.

[1]:https://gcc.gnu.org/onlinedocs/gcc/Common-Function-Attributes.html#Common-Function-Attributes

Here is an example of a C file containing a function:

    ...
    // Test if a string ends with an other string.
    // If any of the string if NULL, return false.
    bool str_endswith(const char *str, const char *end)
    {
        if (!str || !end) return false;
        if (strlen(str) < strlen(end)) return false;
        const char *start = str + strlen(str) - strlen(end);
        return strcmp(start, end) == 0;
    }

# BASIC UNIT TESTS

If I add the unit test code in the file:

    void my_unit_test(void)
    {
        assert(str_endswith("hello", "lo"));
        assert(str_endswith("hello", ""));
        assert(!str_endswith("hello", "ol"));
        assert(!str_endswith("hello", NULL));
        assert(!str_endswith(NULL, "hello"));
    }

Then somewhere in the main file, I can run the test like that:

    if (run_tests) {
        my_unit_test();
    }


This could be improved.  The problems with this approach are:

- The function that executes the tests needs to know about the unit
  test signature.
- If we add tests, we have to also add them to the main function.
- We want some tests to be executed at each startup.

# USING CONSTRUCTOR ATTRIBUTE

We are going to use gcc constructor attributes to improve the unit test.
In the original file, we add:

    #include "tests.h"

    ...

    #if COMPILE_TESTS

    // (Notice that the function is now 'static')
    static void test_str_endswith(void)
    {
        // The orignal unit test
        ...
    }

    TEST_REGISTER(NULL, test_str_endswith, TEST_AUTO);

    #endif


The file tests.h contains the `TEST_REGISTER` macro:

    enum {
        TEST_AUTO = 1 << 0,
    };

    #if COMPILE_TESTS
        void test_register(const char *name, const char *file,
                           void (*setup)(void),
                           void (*func)(void),
                           int flags);
        void tests_run(const char *filter);

        #define TEST_REGISTER(setup_, func_, flags_) \
            static void reg_test_##func_() __attribute__((constructor)); \
            static void reg_test_##func_() { \
                test_register(#func_, __FILE__, setup_, func_, flags_); }
    #else
        #define TEST_REGISTER(...)
        static inline void tests_run(const char *filter) {}
    #endif


And we put the code for `test_register` in a new file (tests.c):

    // Tests.c

    #if COMPILE_TESTS

    typedef struct test {
        struct test *next;
        const char *name;
        const char *file;
        void (*setup)(void);
        void (*func)(void);
        int flags;
    } test_t;

    static test_t *g_tests = NULL;

    void tests_register(const char *name, const char *file,
                        void (*setup)(void),
                        void (*func)(void),
                        int flags)
    {
        test_t *test;
        test = calloc(1, sizeof(*test));
        test->name = name;
        test->file = file;
        test->setup = setup;
        test->func = func;
        test->flags = flags;
        // Add the test to the global list:
        test->next = g_tests;
        g_tests = test;
    }

    static bool filter_test(const char *filter, const test_t *test)
    {
        if (!filter) return true;
        if (strcmp(filter, "auto") == 0) return test->flags & TEST_AUTO;
        return strstr(test->file, filter);
    }

    void tests_run(const char *filter)
    {
        test_t *test;
        printf("Run tests: %s\n", filter);
        for (test = g_tests; test; test = test->next) {
            if (!filter_test(filter, test)) continue;
            if (test->setup) test->setup();
            test->func();
            printf("Run %-20s OK (%s)\n", test->name, test->file);
        }
    }

    #endif



# HOW IT WORKS

Using the `TEST` macro, we can register any unit test function and associated
attributes into a global list of unit tests.  The registering is done
automatically at startup time (before main is executed).

Then to run the tests we just need to call the `tests_run` function, with
an optional filter string.  The function will check for any tests whose name
contains the filter, and run it.

    tests_run("str_endswith");

Some tests can have the `TEST_AUTO` flag set, in that case we can execute
all of them using the filter "auto".  This allows to always run those tests
at startup with a single line of code:

    if (COMPILE_TESTS) tests_run("auto");

I try to use `TEST_AUTO` for all the tests that don't take time, so that I
don't have to think about running my tests, they are always executed.
