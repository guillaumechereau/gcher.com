---
layout: post
title:  "Swift vs C++: lambda functions"
subtitle: Swan vs Platypus
categories: code swift C++ lambda function
redirect_from: /swift-vs-cpp-lambdas.html
---

Recently I got interested in the new Apple language [Swift], that will
probably replace objective-c as the language of choice for native iOS
and OSX applications.

There are many things I like in Swift, and also other things I don't like.
But one thing that I really enjoy is the support for [lambdas], specially
compared to the way it works in C++.

Why do I think the lambdas in swift are better?  Let's have a look:

# A common use case for lambdas

In my opinion, one of the most important usage of lambdas is as a syntax
sugar for non-deferred callback functions.  By non-deferred I mean that
the callback will only be used during the call of the function it is
passed to (deferred callbacks are also interesting, but I think I actually use
them less often).

Let's imagine that I have a module that handles a collection of objects.
For the sake of the argument, let say it's an interface to a database of
cities.

The API provides a function that allows to search the data for all the
cities matching a given name.  We cannot return a single city, because
several cities might match the request.  We could return a list, but there
is a cost to creating a new list for each request.  So instead we can use
a callback function, and the API (in C) might look like that:

    // Iter all the cities matching a given name.
    void city_search(const char *name,
                     void (*f)(const struct city *city));

It can be used like that:

    static void on_city(const struct city *city) {
        printf("got a city: %s, %s\n",
               city->name, city->country);
    }

    void list_cities(const char *name) {
        city_search(name, on_city);
    }

I have two issues with this.  The first one is that the code is split into two
functions, even though they are logically both part of the same logical block.
When reading the code, it might not be obvious that the `on_city` function is
simply a helper for the `list_city` function.

The second problem is that we might want to use local variables of the
`list_cities` function inside the callback.  In order to do this the
usual way in C is to add an extra `void*` argument in the callback to pass
any value from the caller to the callback:

    void city_search(const char *name,
                     void (*f)(const struct city *city),
                     void *arg);

    static void on_city(const struct city *city, void *arg) {
        const int x = *((int*)arg);
        printf("got a city: %s, %s %d\n",
               city->name, city->country, x);
    }

    void list_cities(const char *name) {
        int x = 10;
        city_search(name, on_city, &x);
    }

It works, but it's not very pretty.

# C++ 11 lambdas

Then comes [C++ 11], and it's lambda support.  If we start with the same
original API:

    void city_search(const char *name, void (*f)(const struct city *city));

We can reimplement the `list_cities` function using a lambda instead of a
callback:

    void list_cities(const char *name) {
        city_search(name, [](const struct city *city) {
            printf("got a city: %s, %s\n",
                   city->name, city->country);
        });
    }

This is already much better.  But there is a problem.  If we want to
use some local variables in the lambda, the compiler will complain that
our lambda has no capture-default, and so doesn't accept local variables.

After reading the documentation we figure out that we need to use the `[=]`
syntax instead of the `[]` for the lambda.  If we want to capture the local
variable by reference instead, we can use `[&]`:

    void list_cities(const char *name) {
        int x = 10;
        // Error: city_search doesn't match the lambda.
        city_search(name, [=](const struct city *city) {
            printf("got a city: %s, %s %d\n",
                    city->name, city->country, x);
        });
    }


But this doesn't work either, because while converting a non capturing
lambda to a function pointer works fine, we cannot convert a capturing
lambda to a function pointer!

As far as I know, the simplest way to make it work properly is to use the
`<functional>` std library and change the syntax of the API to this hideous one:

    #include <functional>
    void city_search(const char *name,
                     std::function<void(const struct city *city)> f);

Or, if we don't want or cannot use the std lib, we can also do the abomination
of turning the function into a template:

    template<class F>
    void city_search(const char *name, F f) {
        ...
    }

So in the end we have to decide between three different choices of function
for our API, none of them being simple and convenient.

# Lambdas in swift

In swift, the same code would be as follow:

    func city_search(_ name: String, callback: (City) -> Void) {
        ...
    }

    func list_cities(name: String) {
        let x = 10
        city_search(name) {
            print("got a city \($0.name) \($0.country) \(x)")
        }
    }

It is much better:

- The lambda can access local variables by default, there is no need
  for a special syntax for it.

- Since the last argument of `city_search` is a function, we can put the
  lambda *after* the closing paren!  This makes the code more readable, and
  the function looks similar to the native control structures (if, while,
  ...)

- For such a small lambda, we can skip the argument declaration and use `$0` as
  the default name for the city variable.  This is specially nice for example
  when doing array mapping function one liners:

        var x = [1, 2, 3, 4]
        var y = x.map {$0 * 2} // [2, 4, 6, 8]

[Swift]: https://swift.org/
[lambdas]: https://en.wikipedia.org/wiki/Anonymous_function
[C++ 11]: https://en.wikipedia.org/wiki/C%2B%2B11
