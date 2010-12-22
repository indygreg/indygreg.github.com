---
layout: post
title: Why I Love Lua
---

I was introduced to Lua by chance at ApacheCon 2009. There was a lull in interesting sessions for a particular time slot and Brian McCallister was giving a presentation on mod_lua and the future of the Apache HTTP Server. Since I didn't know what Lua was, I thought, "why not?" After hearing him talk about Lua and what it can do, I was sold.

Lua is a very fast and lightweight programming language.  It is dynamically typed and has automatic garbage collection. But, if I had to give one reason why I love the language, it is because of the properties that arise because it is designed to be embeddable.

# Independent Interpreters

From the C API, you create a new Lua interpreter by calling the function lua_newstate(). This function returns a lua_State *, which is an opaque data structure that holds the interpreter's entire state. Every function that interfaces with the interpreter takes a lua_State reference as its first argument. Every lua_State is completely independent. There are no locks across states, which means you can have countless threads operating on countless interpreters and the only overhead is context switching. No critical sections, locks, etc.

# Interpreters Small and Nearly Free

Creating a new Lua interpreter is so simple and nearly free, it should be illegal. On a Linux virtual machine running on my aging desktop, I can instantiate well north of 100,000 interpreters *per second*. And, if my profiling is correct, each interpreter is less than 10k in heap memory. Compare this to say Cpython, which can do about 160 interpreters a second and the interpreter increases RSS by significantly more (2MB for the parent interpreter and about 500k for each sub-interpreter).

When interpreters are that small and fast to create and destroy, it opens up a whole slate of possibilities. Say you are writing a server that needs to maintain persistent state with thousands of simultaneous clients. You want to use a scripting language because it is easier to maintain than C/C++. And, you want to segment each client from the others so state doesn't collide. Solution: fire up a Lua interpreter for each client. They are small, persistent, and can be destroyed and have their memory free()d extremely efficiently. Instead of passing a struct or class instance around (complete with all the threading headaches therein), you can pass a Lua interpreter. When an event of interest occurs, you just queue up a function for execution in Lua and run it.

# Interpreters Are Limited by Default

One of the reasons Lua interpreters are so small is that a freshly-instantiated Lua interpreter has very few libraries loaded. Want to do I/O? You'll need to load a module. Want to have access to the string library? You'll need to load a module. Want to load a module? You'll need to load a module. Wait, wuh?!

Yes, in a fresh interpreter, it is impossible for Lua code to import a module. From the C API, you need to load the C library which enables the specified Lua interpreter to have access to the functions that permit Lua code to load modules.

Some may find this lack of a built-in standard library a setback. I consider it one of Lua's greatest features. For one, there are zero surprises that come with a new interpreter. As someone who wears an amateur security white hat, I ask "what could happen if a malicious user could execute any code here. What's the worst they could do?" With Lua, a whole lot of nothing. File access? Not unless the I/O module is loaded! Networking calls? Not unless I load a networking module! About the worst they can do is execute an infinite loop or allocate a lot of variables to exhaust memory. And, Lua has a way of dealing with that (read on).

When interpreters can't do much, you can consider doing crazy things like allowing execution of arbitrary Lua code from untrusted clients. Try that with almost every other scripting language and I can almost guarantee you'll get hacked. With Lua, you have to explicity load a gun and aim it at yourself. This is much better than trying to disarm a fully automatic rifle pointed right at you.

# Transparently Calling Out Into C

With Lua, it is effortless to register a C function or object into a Lua interpreter. You just need to define a C function that takes a lua_State * argument and call two C APIs to expose your function to Lua code. Have a critical section you want to implement in C for performance reasons? Go right ahead. You can even rip out the version implemented in Lua and replace it with the C function. All at run-time.

# Insanely Fast

The main Lua distribution available on lua.org is pretty darn quick. In many scenarios, it is a couple times faster than other interpreted languages, like Python, PHP, and Ruby (anything is faster than Ruby, it seems). If that's not fast enough for you, you can run LuaJIT, which is a Lua implementation consisting of a lot of assembly and crazy optimizations. It is *insanely* quick. It  can even outperform C in some cases. Outperform C: are you kidding me?! Considering you get garbage collection and more concise code for free, that's a no-brainer.

# It's Not All Perfect

Of course, no programming language is perfect. Lua has many faults, depending on your perspective. It represents all numerics as a single type (double-precision floating point by default). Personally, I wish they would have separate types for integer and decimal (with automatic type conversion, of course).

Another drawback to some is the lack of more "advanced" programming techniques, like classes, member visibility etc. The only data structure defined in Lua is a table. Tables have keys and values. Sometimes you don't care about the keys (this is how you model lists and arrays). Everything, including inheritance (if you want to do that), must be modeled through tables. On one hand, it is really cool that almost every functionality in Lua is accomplished through a single data structure. But, if you are coming from a more "enterprisey" language, like Java, things are sure to feel a bit like the wild west.

