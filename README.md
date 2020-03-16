> You probably got on this page from [jtc](https://github.com/ldn-softdev/jtc#programming-model), where I made quite a bold statement about `C` and `C++`. This article hopefully clarifies what I meant.

<p align="right">
- How to tell appart a good developer from a bad one?<br>
- Easy! A good one takes time to cover his code with UT,<br>
a bad one never does</br>
</p>


# What really makes `C++` superior to `C` and most of other bare metal languages 

> This article represents the author's opinion only and in no degree meant to claim the absolute verity. Feel totally free to agree or disagree with the opinion of the author.  
> The article does not compare C++ to C (obviously the former is a superset over to the latter), but rather it provides a view on the programming methodology both languages can offer.

There's been many debates and holy wars between `C` and `C++` camps. I do not mean to start another one, but I want to _highlight just one difference_, which, _IMO_ really sets `C++` apart from `C`. By the notion, I mean to show the advantage of `C++` in a very short article and explain it with a very basic and a simple example.

The difference I meant is a _programming methodology_ (ways a language offers to write programs). When `C` was created (early 70's) it was someone may call a revolutionary and breaking through (which is true), but hey, take a look at the calendar, what year is it now? Since then the programming methodology has progressed by so much.

While `C` has received some updates (`C99`, `C11`, `C18`) all those were mostly facelifts. Those updates did not change much the programming methodology `C` offers, which up to date remains what it was when the language was created - _largely, a syntactic sugar for an assembly language_. I know, that order of words may cause a great unrest in the `C` camp and I'm aware that `C` is not entirely a syntactic sugar - it has data types, stack/calls overheads dealt by the compiler, etc, but all what it has does not take the language too much far away from that definition even after all those updates.

On the other hand, `C++` programming methodology evolves with almost every `C++` update (take `C++11` alone: among other things, introduction of a _move semantic_, _perfect forwarding_, _functional lambdas_ - reshaped `C++` methodology - e.g., it introduced a whole new way of designing/writing software which could help avoiding resources leaks (memory leaks come here too) entirely
> Clarification: given that such way of _idiomatic_ writing in `C++` would be largely based on the underlying libraries - `STL` and `Boost`, that statement is as good as leak-free `STL` and `Boost` are, which is a quite high certainty.

That is the main idea behind my saying that today `C` is a flawed language. It does not mean that `C++` has no flaws, not at all, but it means that `C` is flawed much more (actually a big time) than `C++` when it comes down to a programming ideology and methodology.

Just to example what I mean, let's review a tiny code sample (to keep the article short): consider a following code snippet, where some function needs to operate depending on some external state, but for domestic purposes it has to modify it and reinstate upon exit. In `C` it would look like this:

```C
...
enum Color global_color = Red;
...

void green_func(void) {
 /* preserve externals state of the variable */
 enum Color preserved_color = global_color;

 /* set color to the required code */
 global_color = Green;

 /* ... do some stuff which would be dependent on color ... */
 
 /* reinstate color code */ 
 global_color = preserved_color;
 return;
}
```

For the moment, let's leave aside an argument that such code is not concurrent-able, say, it's meant to be used only in a single-threaded environment. That code above is good and bug-free (w.r.t to the shown code snippet). But only until you come back some time later (or your not so attentive colleague) and modify that code, e.g., enhance a design of the program by inserting somewhere in the mids a condition to leave the function:
```C
void green_func(void) {
 /* preserve externals state of the variable */
 enum Color preserved_color = global_color;

 /* set color to the required code */
 global_color = Green;

 /* ... do some stuff which would be dependent on color ... */
 if(some_condition) return;

 /* ... do some more other stuff ... */

 /* reinstate color code */ 
 global_color = preserved_color;
 return;
}
```
So, we arrive to a bug here: `global_color`'s state is not reinstated if the condition is hit. And this is the main flaw (IMO): `C` offers no mechanism, no methodology to protect its code against one intrinsic property of a human brain: capacity to miss things (forget stuff, or speaking generally make errors - create bugs).

Even in the future, after fixing that bug, if more of such conditions to be added to the code, _there's absolutely no guarantee_ that a person who would be doing that code change won't introduce exactly the same bug into the code.

Now, the same code sample in `C++` could be written in (almost) exactly the same way, but that would be a `C`-way of writing a `C++` program. That's why when comparing `C` and `C++`, it must only be compared a `C`-way vs idiomatic `C++` way (the `C++`'s possibly biggest flaw is that it let creating `C`-style programs; the `C++`'s complexity then would come second, but `C++` evolves even in that aspect, e.g.: `C++17`'s introduction of _`fold expressions`_ and update of `C++11`'s _`constexpr`_ specifier simplifies 
[`TMP`](https://en.wikipedia.org/wiki/Template_metaprogramming)
a bit, there are many other enhancements too).

So, what `C++` can offer here when writing the same code snippet (w/o altering the overall design)?
```C++
...
Color global_color;
...

void green_func(void) {
 PRESERVE(global_color)	// macro PRESERVE will expand into an object creation, which preserves the value of
		        // the `global_color` in the constructor and reinstate its value in the destructor;
 // set color to the required code
 global_color = GREEN;

 // ... do some stuff which would be dependent on color ...
 if(some_condition) return;

 // ... do some more other stuff ...

 return;
}
```
##
> If you're interested seeing how `PRESERVE` macro and the underlying class could be implemented, look in my
[source](https://github.com/ldn-softdev/jtc/blob/master/lib/extensions.hpp#L199)
for **`GUARD`** macro definition - it's a _polymorphic_ macro allowing safeguarding elements accessbile by address, or by respective
_getter_ and _setter_ methods
##
As you can see - that code above is immune to the pitfall of the `C` example, because does not matter if you need to insert one or multiple conditions to leave the function (or even leaving it unintentionally, via _`exception`_/_`throw`_) - the restoration of the preserved variable/object will be taken care now automatically by the compiler and not by a human (the compiler will never miss to destroy an automatic object leaving the scope). 

That is just a single example of possible sources of bugs, but the number of such possibilities is countless, and in many cases `C++` offers a way to deal with it - i.e. eliminate the possibility of the potential problems (the shown way of relying on auto-destruction of automatic objects leaving the scope is not the only trick up the `C++`'s sleeve, there are plenty others).

The modern `C++` is possibly the most comprehensive language - it offers pretty much all programming paradigms: _structured_, _imperative_, _OOP_, _functional_, _declarative_, _pure generic
([`TMP`](https://en.wikipedia.org/wiki/Template_metaprogramming))_.
Based on those it's possible to achieve easily other models as well (e.g., _event-driven_). There's only a handful of other languages nearly as comprehensive as `C++`. But it's the ability to let creating an idiomatic code which is immune and resistant to human ability to make bugs (to some extend) is the one aspect that sets it apart from the most of others and from `C` particularly.

















