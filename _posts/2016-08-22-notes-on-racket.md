---
layout: default
title: Notes on Racket
tags: [cs]
---

The Racket language belongs to the LISP family. The main difference between LISP and other languages is that you can name like-this, as opposed to likeThis (or, even worse, like_this), and all operators all prefix. That and homoiconicity. Homoiconicity is a fancy name for "code as data", i.e., LISP code can be manipulated as any other data structure. Such manipulation is achieved by the use of macros- functions that receive code and output code, which in some cases might [as well be black magic](http://www.greghendershott.com/fear-of-macros/).

But let's not get ahead of ourselves. Shall we begin with a simple function?

{% highlight racket %}
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

We have here a highlight of some nice Racket features: named let construct, which declares a function and subsequently calls it; arity based function dispatch with case-lambda; and the convention of defining functions to operate on different arities, say, + 2 3 or + 2 3 4 5 6 7 (see also [apply](https://docs.racket-lang.org/reference/procedures.html#%28def._%28%28lib._racket%2Fprivate%2Fbase..rkt%29._apply%29%29), the ultimate uncurryer). We can also, however, see the horrible features: everything sort of blends together since the syntax is so regular; sometimes, deeply nested ))))), which you will get with [let, let* and their friends](https://docs.racket-lang.org/reference/let.html), are next to impossible to digest. But simple-function is just the foldl function renamed. Can we get something more spicy?

{% highlight racket %}
(define-syntax (define-simple-macro stx)
  (syntax-parse stx
    [(define-simple-macro (~and (macro:id . _) pattern) . body)
     #`(define-syntax macro
         (syntax-parser/template
          #,((make-syntax-introducer) stx)
          [pattern . body]))]))
{% endhighlight %}

define-syntax is the same as define, only for macros. The reader, who always completes all parts noted with "left as exercise to the reader", have it already figured out for sure and thus must be getting bored. More examples?

{% highlight racket %}
(begin-for-syntax
 (define (tx:define-*-syntax-class stx splicing?)
   (syntax-case stx ()
     [(_ header . rhss)
      (parameterize ((current-syntax-context stx))
        (let-values ([(name formals arity)
                      (let ([p (check-stxclass-header #'header stx)])
                        (values (car p) (cadr p) (caddr p)))])
          (let ([the-rhs (parse-rhs #'rhss #f splicing? #:context stx)])
            (with-syntax ([name name]
                          [formals formals]
                          [parser (generate-temporary (format-symbol "parse-~a" name))]
                          [arity arity]
                          [attrs (rhs-attrs the-rhs)]
                          [options (rhs-options the-rhs)])
              #`(begin (define-syntax name
                         (stxclass 'name 'arity
                                   'attrs
                                   (quote-syntax parser)
                                   '#,splicing?
                                   options
                                   #f))
                       (define-values (parser)
                         (parser/rhs name formals attrs rhss #,splicing? #,stx)))))))])))
{% endhighlight %}

Multiple returns are really neat, huh? What about pattern matching as a library (take that, Haskell)?

{% highlight racket %}
(begin-for-syntax
 (define (do-one-contract stx scname stxclass rec pos-module-source)
   (match (stxclass-arity stxclass)
     [(arity minpos maxpos minkws maxkws)
      (let* ([minpos* (length (ctcrec-mpcs rec))]
             [maxpos* (+ minpos* (length (ctcrec-opcs rec)))]
             [minkws* (sort (map syntax-e (ctcrec-mkws rec)) keyword<?)]
             [maxkws* (sort (append minkws* (map syntax-e (ctcrec-okws rec))) keyword<?)])
        (define (err msg . args)
          (apply wrong-syntax scname msg args))
        (unless (<= minpos minpos*)
          (err (string-append "expected a syntax class with at most ~a "
                              "required positional arguments, got one with ~a")
               minpos* minpos))
        (unless (<= maxpos* maxpos)
          (err (string-append "expected a syntax class with at least ~a "
                              "total positional arguments (required and optional), "
                              "got one with ~a")
               maxpos* maxpos))
        (unless (null? (diff/sorted/eq minkws minkws*))
          (err (string-append "expected a syntax class with at most the "
                              "required keyword arguments ~a, got one with ~a")
               (join-sep (map kw->string minkws*) "," "and")
               (join-sep (map kw->string minkws) "," "and")))
        (unless (null? (diff/sorted/eq maxkws* maxkws))
          (err (string-append "expected a syntax class with at least the optional "
                              "keyword arguments ~a, got one with ~a")
               (join-sep (map kw->string maxkws*) "," "and")
               (join-sep (map kw->string maxkws) "," "and")))
        (with-syntax ([scname scname]
                      [#s(stxclass name arity attrs parser splicing? options integrate)
                       stxclass]
                      [#s(ctcrec (mpc ...) (mkw ...) (mkwc ...)
                                 (opc ...) (okw ...) (okwc ...))
                       rec]
                      [arity* (arity minpos* maxpos* minkws* maxkws*)]
                      [(parser-contract contracted-parser contracted-scname)
                       (generate-temporaries #`(contract parser #,scname))])
          (with-syntax ([(mpc-id ...) (generate-temporaries #'(mpc ...))]
                        [(mkwc-id ...) (generate-temporaries #'(mkwc ...))]
                        [(opc-id ...) (generate-temporaries #'(opc ...))]
                        [(okwc-id ...) (generate-temporaries #'(okwc ...))])
            (with-syntax ([((mkw-c-part ...) ...) #'((mkw mkwc-id) ...)]
                          [((okw-c-part ...) ...) #'((okw okwc-id) ...)]
                          [((mkw-name-part ...) ...) #'((mkw ,(contract-name mkwc-id)) ...)]
                          [((okw-name-part ...) ...) #'((okw ,(contract-name okwc-id)) ...)])
              #`(begin
                  (define parser-contract
                    (let ([mpc-id mpc] ...
                          [mkwc-id mkwc] ...
                          [opc-id opc] ...
                          [okwc-id okwc] ...)
                      (rename-contract
                       (->* (any/c any/c any/c any/c any/c any/c any/c any/c
                             mpc-id ... mkw-c-part ... ...)
                            (okw-c-part ... ...)
                            any)
                       `(,(if 'splicing? 'splicing-syntax-class/c 'syntax-class/c)
                         [,(contract-name mpc-id) ... mkw-name-part ... ...]
                         [okw-name-part ... ...]))))
                  (define-module-boundary-contract contracted-parser
                    parser parser-contract #:pos-source #,pos-module-source)
                  (define-syntax contracted-scname
                    (make-stxclass
                     (quote-syntax name)
                     'arity*
                     'attrs
                     (quote-syntax contracted-parser)
                     'splicing?
                     'options
                     #f))
                  (provide (rename-out [contracted-scname scname])))))))])))

{% endhighlight %}

In conclusion, Racket is a nice, expressive, ultra dynamic LISP with Super Cow Powers. I recommend reading the [official racket repository](https://github.com/racket/racket/), whence all our examples were taken, as a starting guide.
