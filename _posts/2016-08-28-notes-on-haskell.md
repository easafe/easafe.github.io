---
layout: default
title: Notes on Haskell
tags: [cs]
---

Haskell is a statically typed language- famous for cussing Ocaml's lack of purity- whose entire presence on Tiobe index consists of people asking "Why should I learn Haskell?" on stackoverflow. Well, not that Tiobe amounts to anything. Haskell defining feature is ~~monads~~ its type system (however, make no mistake: [there](http://wiki.portal.chalmers.se/agda/pmwiki.php) [are](http://www.idris-lang.org/) [more](https://coq.inria.fr/) [advanced](http://clean.cs.ru.nl/Clean) [ones](http://shenlanguage.org/), but Haskell got more web scale frameworks than all of those combined). The nice thing about good type systems is that they force you to see _everything_ in term of types. This statement might seem tautological, but once you get rid of weak Java-like static typing and can spot even the type of types- ahem, [kinds](https://wiki.haskell.org/Kind)- things start to make sense in a sort of organized, Category Theory way, so that you can ditch superfluous things like documentation, as you have [all types nicely displayed above the definition of stuff](https://www.haskell.org/haddock/). But what does such type system have that makes it so good and advanced?, may ask you. As always, let's build leg strength starting up from easy steps:

* Existentially quantified types

In Haskell, even the most innocent looking functions hold key features. Consider the identity function, defined as:

{% highlight haskell %}
id a = a
{% endhighlight %}

Its type, however, reads:
{% highlight haskell %}
id :: forall a . a -> a
{% endhighlight %}

Yay, polymorphic types for free, without silly angles! Oh, wait, those are universal types. Can we tone down the ⊥s?

{% highlight haskell %}
data ShowBox = forall s. Show s => SB s

instance Show ShowBox where
 show (SB s) = show s   

heteroList :: [ShowBox]
heteroList = [SB (), SB 5, SB True]
{% endhighlight %}

See? With state-of-art types we can crudely construct the heterogeneous lists present in every single dynamic language out-of-the-box. So forth, if we turn on the nice extension RankNTypes,

{% highlight haskell %}
{-# LANGUAGE RankNTypes #-}

runST :: forall a. (forall s. ST s a) -> a
{% endhighlight %}

we also can have variables like virtually every existing language! Take that, LISP macros!

* Clarity

Another thing we might say to Haskell's favor is its no nonsense philosophy. Who never walked down the street and thought "Man, a fixed pointer combinator is all I need"!?

{% highlight haskell %}
 fix :: (a -> a) -> a
 fix f = let x = f x in x
{% endhighlight %}

While other languages, such as C++ or Python, use fancy (and overrated) constructs like assignment or while loops, Haskell features only intuitive, down-to-earth practicalities, e.g., [free monads](https://hackage.haskell.org/package/free), [yoneda lemmes](https://hackage.haskell.org/package/category-extras-0.52.1/docs/Control-Functor-Yoneda.html), [type level recursion](https://www.schoolofhaskell.com/user/mutjida/typed-tagless-final-linear-lambda-calculus/6-recursive-types) and, of course, [several clear](https://hackage.haskell.org/package/pipes) ways [to do](https://hackage.haskell.org/package/conduit) streaming [IO](https://hackage.haskell.org/package/io-streams). I wish they had showed me recursion schemes with catamorphisms when I first learned how to program.

* Type families

Type families solves those problems you otherwise wouldn't have with simpler languages. The recommend approach is to use them together with [type level literals](https://hackage.haskell.org/package/base-4.9.0.0/docs/GHC-TypeLits.html). Let's see an example:

{% highlight haskell %}
{-# LANGUAGE TypeFamilies, MultiParamTypeClasses, FlexibleContexts #-}
class (Show pokemon, Show (Move pokemon)) => Pokemon pokemon where
  data Move pokemon :: *
  pickMove :: pokemon -> Move pokemon

data Fire = Charmander | Charmeleon | Charizard deriving Show
instance Pokemon Fire where
  data Move Fire = Ember | FlameThrower | FireBlast deriving Show
  pickMove Charmander = Ember
  pickMove Charmeleon = FlameThrower
  pickMove Charizard = FireBlast

data Water = Squirtle | Wartortle | Blastoise deriving Show
instance Pokemon Water where
  data Move Water = Bubble | WaterGun deriving Show
  pickMove Squirtle = Bubble
  pickMove _ = WaterGun

data Grass = Bulbasaur | Ivysaur | Venusaur deriving Showrecursion schemes
instance Pokemon Grass where
  data Move Grass = VineWhip deriving Show
  pickMove _ = VineWhip

printBattle :: String -> String -> String -> String -> String -> IO ()
printBattle pokemonOne moveOne pokemonTwo moveTwo winner = do
  putStrLn $ pokemonOne ++ " used " ++ moveOne
  putStrLn $ pokemonTwo ++ " used " ++ moveTwo
  putStrLn $ "Winner is: " ++ winner ++ "\n"

class (Show (Winner pokemon foe), Pokemon pokemon, Pokemon foe) => Battle pokemon foe where
  type Winner pokemon foe :: *
  type Winner pokemon foe = pokemon

  battle :: pokemon -> foe -> IO ()
  battle pokemon foe = do
    printBattle (show pokemon) (show move) (show foe) (show foeMove) (show winner)
   where
    foeMove = pickMove foe
    move = pickMove pokemon
    winner = pickWinner pokemon foe

  pickWinner :: pokemon -> foe -> (Winner pokemon foe)

instance Battle Water Fire where
  pickWinner pokemon foe = pokemon

instance Battle Fire Water where
  type Winner Fire Water = Water
  pickWinner = flip pickWinner

instance Battle Grass Water where
  pickWinner pokemon foe = pokemon

instance Battle Water Grass where
  type Winner Water Grass = Grass
  pickWinner = flip pickWinner

instance Battle Fire Grass where
  pickWinner pokemon foe = pokemon

instance Battle Grass Fire where
  type Winner Grass Fire = Fire
  pickWinner = flip pickWinner

main :: IO ()
main = do
  battle Squirtle Charmander
  battle Charmeleon Wartortle
  battle Bulbasaur Blastoise
  battle Wartortle Ivysaur
  battle Charmeleon Ivysaur
  battle Venusaur Charizard
{% endhighlight %}

Fuck simple dynamic dispatch from real OO languages (read: SmallTalk) or no mental overhead dynamic typing- we can play type level Pokemon.

Summing up, Haskell is de facto practical man's language. While its use is best advised where cheap PHP hosting is currently employed, its advanced type system can help you leverage all kinds of [abstract nonsense](https://en.wikipedia.org/wiki/Category_theory).
