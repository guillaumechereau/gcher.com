---
categories:
  - C++
  - code
  - pattern
  - video
  - game
alias: /object-factory-c++.html
subtitle: A nice trick for video game code design
title: Automatic object factory in C++
date: 2014-06-15
---


Unlike other object oriented languages like java, C++ does not have much
support for introspection.  So for example it is not possible to request the
list of subclasses of a given class.

In this post I am going to talk about a little trick that I found very useful
when dealing with a lot of subclasses of a single base class.  This is
typically what we see in video game, where we have a base `Object` class that
represents any kind of object in the game, and then one subclass for each type
of object, like the player, enemies, etc.


In its simplest form, we can imagine a program that defines two kind of
entities in the game, one being the player, and the other one an enemy.  Both
inherit from the same base class:


    class Object
    {
    public:
        virtual void iter() {}
    };


    class Player : public Object
    {
    public:
        virtual void iter() {
            cout << "player iter" << endl;
        }
    };

    class Enemy : public Object
    {
    public:
        virtual void iter() {
            cout << "enemy iter" << endl;
        }
    };

Typically we would then have a list of objects, and we can fill it with
instances of either `Player` or `Enemy`.  As the code grows, we can add other
subclasses of `Object`.

Now the problem is that each time we add a new object in the game we need to
call the proper constructor for the type we want, so if we want to add one
player and two enemies the code might look like this:

    objects.append(new Player());
    objects.append(new Enemy());
    objects.append(new Enemy());

However, sometime we don't know in advance what type of object we want to add
into the game.  For example when we read a level file, we might decide the type
by parsing some sort of id.  So what we need is some sort of factory function
that could create any type of object from an id.  It might look like this:

    // Static method of Object to create an object from a subclass name.
    Object *Object::create(const string &name)
    {
        if (name == "Player") return new Player();
        if (name == "Enemy") return new Enemy();
        ...
        return NULL;
    }

Writing this function by hand is a bit tedious when the number of subclass
become important.  Fortunately we can do better than that:

First let's define an abstract factory class, whose only virtual method returns
a new instance of `Object`:

    class ObjectFactory
    {
    public:
        virtual Object *create() = 0;
    };

Then, we also add a static map of string to factory object to the `Object`
class, and a static method to register a new factory into the map.

    class Object
    {
    ...
    public:
        static void registerType(
            const string& name, ObjectFactory *factory)
        {
            fatories[name] = factory;
        }
    private:
        static map<string, ObjectFactory*> factories;
    };

So if we assume that the factories map is filled with items that map all the
subclass names to a factory creating a new instance of the subclass, our global
factory function can become:

    Object *Object::create(const string &name)
    {
        return factories[name]->create();
    }

Now the only remaining problem is to fill the factories map automatically from
the subclasses of `Object`.  Turn out there is an elegant way to achieve that.

The idea is to add bellow each subclass a global static instance of a Factory
subclass, whose constructor calls `Object::registerType`.  Since global object
constructors are called at startup before main, this will automatically fill
our factory list.  The code for the `Player` class would look like this:

    // Player.cpp

    class PlayerFactory : public ObjectFactory
    {
    public:
        PlayerFactory()
        {
            Object::registerType("Player", this);
        }
        virtual Object *create() {return new Player();}
    };

    static PlayerFactory global_PlayerFactory;

Since the code is always the same, we can turn it into a convenient macro:

    #define REGISTER_TYPE(klass) \
        class klass##Factory : public ObjectFactory { \
        public: \
            klass##Factory() \
            { \
                Object::registerType(#klass, this); \
            } \
            virtual Object *create() { \
                return new klass(); \
            } \
        }; \
        static klass##Factory global_##klass##Factory;

And now all we have to do is add a line to register the types in each subclass
cpp file:

    // Player.cpp
    REGISTER_TYPE(Player)

    // Enemy.cpp
    REGISTER_TYPE(Enemy)


And there is even an extra advantage: since we never have to access the
subclasses directly -even their constructors- we can get ride of all the header
files, and directly move the subclass definition into the cpp files to make the
code smaller.
#}
