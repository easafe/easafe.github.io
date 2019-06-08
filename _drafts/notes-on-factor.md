Imagine yourself a Haskell programmer. You sip on a glass of 20 years old whisky, fireplace creeks while a SPJ talk plays as background music, and your hatred of named parameters burns stronger than never. You have engaged in (pointless)[point free link] programming; your code is literally . followed by $ with some Arrows for good measure; you don't even need to name your functions. But it is not enough. What else can be done to further elevate glorious function composition? Can you finally get rid of the one thing which is the root of all evil: variables?

Okay, the concatenative paradigm is older than C, and there is no evidence that any of its creators ever used Haskell, but the obsession with function composition is very real. What's a concatenative programming language, asks you? Well, imagine you had a very good idea, and then you applied it to absolutely everything until it became impractical- that's how it feels to use any such languages.

To represent the concatenative style, we will use Factor. Any Factor function can be seen as taking a stack and returning a modified stack, i.e., instead of named parameters, you just pop or push values to the stack. Consider the following expression:

1 1 12 3 4 + - 3array [ 9 * ] map

Evaluation is left to right. Numbers just push themselves to the stack, so we get(top of the stack is at the bottom):

1
1 
12
3
4

then the function + pops up two values from the stack and returns a single one

1
1
12
7

likewise, -

1
1
5

the function 3array creates an array made up of the three top most values on the stack

{ 1 1 5 }

brackets, known as quotations, "escape" expressions, so instead of being evaluated they just push themselves to the stack

{ 1 1 5 }
[ 9 * ]

finally, the map function uses the quotation as a sort of anonymous function which for every element in the array pushes 9 to the stack and multiples them, yielding

{ 9 9 45 }

This has two interesting immediate consequences. First, it makes things balls to the wall weird. Second, every single piece of code can refactored and made its own thing, since no named parameters are referenced, and function composition is just juxtaposition (the name Factor seems to be a pun on raptor though). Here the imaginary haskeller of the first paragraph gets very happy: does it mean that equational reasoning is pervasive in concatenative languages? Not so fast. There is nothing actually stopping any given function from accessing any element in the stack (the state of the entire program) except that it is very inconvenient to reach past the fourth or so element. This brings another crucial point about Factor(or concatenative languages in general): you will either spend a lot of time making sure the elements in the stack are in the right order or you will code will be an unreadable clusterfuck of shuffling functions such as dup, dip or drop.

But now imagine that the haskeller was a smalltaker. Yes, not even concatenative languages are safe from the "everything is an object" motto. Everything in Factor is an object. But object orientedness comes not from classes but rather tuples, generic words and predicates.

TUPLE: nice-folks name url ; ! probably "azafeh.com" "Eduardo Asafe" will be on the stack

GENERIC: + ( obj obj -- obj ) ;

M: fixnum + 
     <actual math> ;

M: nice-folks + 
      <something which makes sense for nice-folks> ; 

Parsing a concatenative language like Factor is a joy(pun intended). Everything is separated by white space. Can you see where this is going? Factor offers metaprogramming via the SYNTAX: word(a direct jab at LISPs) with a [surprisingly simple api](http://docs.factorcode.org:8080/content/article-parsing-words.html)


