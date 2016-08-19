---
layout: default
title: Notes on Racket
tags: [cs]
---

The Racket language belongs to the LISP family. The main difference between LISP and other languages is that you can name like-this, as opposed to likeThis (or ,even worse, like_this), and all operators all prefix. That and homoiconicity. Homoiconicity is a fancy name for "code as data", i.e., LISP code can be manipulated as any other data structure. Such manipulation is achieved by the use of macros- functions that receive code and output code, which in some cases might [as well be black magic](http://www.greghendershott.com/fear-of-macros/).

But let's not get ahead of ourselves. Shall we begin with a simple function?

{% highlight scheme %}
(define simple-function
  (let ([mapadd (lambda (f l last)
                  (let loop ([l l])
                    (if (null? l)
                      (list last)
                      (cons (f (car l)) (loop (cdr l))))))])
    (case-lambda
      [(f init l)
       (let loop ([init init]
                  [l l])
         (if (null? l)
           init
           (loop (f (car l) init) (cdr l))))]
      [(f init l . ls)
       (let loop ([init init]
                  [ls (cons l ls)])
         (if (pair? (car ls))
           (loop (apply f (mapadd car ls init)) (map cdr ls))
           init))])))
{% endhighlight %}

We have here a highlight of some nice Racket features: named let construct, which declares a function and subsequently calls it; arity based function dispatch with case-lambda; and the convention of defining functions to operate on different arities, say, + 2 3 or + 2 3 4 5 6 7 (see also [apply](https://docs.racket-lang.org/reference/procedures.html#%28def._%28%28lib._racket%2Fprivate%2Fbase..rkt%29._apply%29%29), the ultimate uncurryer). We can also, however, see the horrible features: everything sort of blends together since the syntax is so regular; sometimes, deeply nested ))))), which you will get with [let, let* and their friends](https://docs.racket-lang.org/reference/let.html), are next to impossible to easily digest. But simple-function is just the foldl function renamed. Can we get something more spicy? Enter the
