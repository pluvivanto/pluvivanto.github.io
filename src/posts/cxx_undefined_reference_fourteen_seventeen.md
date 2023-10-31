---
title: Weird undefined reference - CXX 14 vs 17
date: git Created
tags: [CXX]
permalink: "posts/{{ title | slug }}/index.html"
---

Lately, I've been working on a project that involves migrating our C++17-based solution to an environment that relies on C++14. Honestly, I'm not entirely sure why a client would want such a downgrade, but the customer is king, right?

So, I went ahead and replaced all the features introduced in C++17, like `<variant>`, `<filesystem>`, inline variables, lambda capture of `*this`, `constexpr` lambdas, `__has_include`, and, regrettably, one of the most important ones, the `[[maybe_unused]]` attribute. Most of these changes went smoothly, but I ran into a peculiar issue: undefined references during linking. I was absolutely certain that everything was correctly declared, defined in the header files, and properly linked.

After several days (and a few sleepless nights) pulling my hair out, I finally pinpointed the root cause. The issue stemmed from `constexpr static` data members in the header files. To be more precise, it's due to the nuances of how C++ treats these members:

> A constexpr specifier used in an object declaration or non-static member function (until C++14) implies const. A constexpr specifier used in a function or static data member (since C++17) declaration implies inline. If any declaration of a function or function template has a constexpr specifier, then every declaration must contain that specifier. 
> 
> \- from [cppreference][cppref1]

> If a const non-inline (since C++17) static data member or a constexpr static data member (since C++11)(until C++17) is [odr-used][cppref3], a definition at namespace scope is still required, but it cannot have an initializer. 
> 
> A constexpr static data member is implicitly inline and does not need to be redeclared at namespace scope. This redeclaration without an initializer (formerly required) is still permitted, but is deprecated. (since C++17)
>
> ```cpp
> struct X
> {
>     static const int n = 1;
>     static constexpr int m = 4;
> };
> const int *p = &X::n, *q = &X::m; // X::n and X::m are odr-used
> const int X::n;                   // … so a definition is necessary
> constexpr int X::m;               // … (except for X::m in C++17)
> ```
>
> \- from [cppreference][cppref2]

This means that I didn't have to re-declare `static const` and `static constexpr` member variables in corresponding `.cpp` files in C++17. But because I'm downgrading the project, I have to do it now.

So, I've solved one problem by re-declaring every `static constexpr` variable. But what happens when the class is templated? According to the [Standard C++ Foundation][isocpp], it's not advisable to split a templated class into declaration and definition.

I scoured for a more elegant, intuitive, and scalable solution but ended up removing the static keyword and creating an instance every time I needed to use the class. I wish I could come up with a better solution, but I don't believe this "problem" can be fixed without refactoring the entire legacy project. It's working, and I'm content with that for now.

[cppref1]: https://en.cppreference.com/w/cpp/language/constexpr
[cppref2]: https://en.cppreference.com/w/cpp/language/static
[cppref3]: https://en.cppreference.com/w/cpp/language/definition#ODR-use
[isocpp]: https://isocpp.org/wiki/faq/templates#templates-defn-vs-decl
[stackoverflow1]: https://stackoverflow.com/questions/52251833/is-static-constexpr-variable-odr-used