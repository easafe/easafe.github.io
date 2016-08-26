---
layout: default
title: Notes on Haskell
tags: [cs]
---

Haskell is a statically typed language- famous for cussing Ocaml's lack of purity- which entire presence on Tiobe index consists of people asking "Why should I learn Haskell?" on stackoverflow. Well, not that Tiobe amounts to anything. Haskell defining feature is ~~monads~~ its type system (make no mistake: [there](http://wiki.portal.chalmers.se/agda/pmwiki.php) [are](http://www.idris-lang.org/) [more](https://coq.inria.fr/) [advanced](http://clean.cs.ru.nl/Clean) [ones](http://shenlanguage.org/), however Haskell got more web scale frameworks than all of those combined).

The nice thing about good type systems is that they force you to see _everything_ in term of types. Such statement might seem tautological, but once you get rid of weak Java-like static typing and can see even the type of types- ahem, [kinds](https://wiki.haskell.org/Kind)- things start to make sense in a sort of organized, Category Theory way, so that you can even ditch superfluous things like documentation because you have [all types nicely displayed above the definition of stuff](https://www.haskell.org/haddock/). But what has such type system of that makes it good and advanced?, may ask you. As always, let's build up leg strength from easy steps:

* Existentially quantified types

In Haskell even innocent looking functions hold key features. Consider the identity function, defined as:

{% highlight haskell %}
id a = a
{% endhighlight %}

Its type, however, reads:
{% highlight haskell %}
id :: forall a . a -> a
{% endhighlight %}

Yay, polymorphic types for free, without silly <> stuff! Oh, wait those are universal types. Can we tone down the ⊥s?

{% highlight haskell %}
data ShowBox = forall s. Show s => SB s

instance Show ShowBox where
 show (SB s) = show s   

heteroList :: [ShowBox]
heteroList = [SB (), SB 5, SB True]
{% endhighlight %}

See? With so advanced types we can crudely construct the heterogeneous lists that every single dynamic language has out-of-the-box. If we turn on the nice extension RankNTypes,

{% highlight haskell %}
{-# LANGUAGE RankNTypes #-}

runST :: forall a. (forall s. ST s a) -> a
{% endhighlight %}

we also can have variables like virtually every fucking existing language! Take that, LISP macros!

* Type families

Type families solves those problems you otherwise wouldn't have with simpler languages. The recommend approach is to use them together with [type level literals](https://hackage.haskell.org/package/base-4.9.0.0/docs/GHC-TypeLits.html). Let's see an example:
