JavaScript, maybe with the exception of PHP, is the worst widely used language. It is so hyperbolic awful, akin to when one feels down and listens to sad songs to continue down in the way that you feel the need to use it everywhere for everything. Its meme like terribleness spread [to the server](https://nodejs.org/en/); spread to otherwise [unthinkable uses](https://atom.io/) of once a blinking pop up maker language; the corrosive effect has a permanent mark in the brain, up to when you find yourself accepting its horrible malpractices as commom sense, [no matter what they are](https://www.mongodb.com/).  

Now, the programmer who is safe from the monstrosities of web development may be thinking it can't be so bad. However, as I will show you, JavaScript tears down so much concepts taught in comp.sci. 101 that we can make long, nicely subdivided ordered lists about it (well, the list will probably come out with mixed formation and out of order numbering since it is JavaScript).

1. Functions as only class citizens

Function as first class citizens are really nice, however JavaScript, as it does with everything, takes it a little to the extreme. The only thing which might be as versatile as a JavaScript function is Hokuto Shinken- but that can only be used by a single person at any given time whereas JavaScript is used by millions of unsuspicious programmers every day.

examples of function use

2. Forget proper scoping
JavaScript has no concept of block scoping. Instead, functions (again) determine the scope of variables, which makes you write code as follows,

{% highlight javascript %}
function whereBlockAt(){
  var a = 23;

  function blockIsHere() {
    var a = 25;
  };
};
{% endhighlight %}

just in case you feel like reusing a variable name. Misspelled a variable name? Don't worry, the JavaScript engine will kindly infer you mean something declared elsewhere. Need to hide some implementation details? Have fun with (function(){(function(){}())}()).

Ah, and don't forget the _[this](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/this)_ mess.

3. Fake keywords

JavaScript has a wart unique to itself: a few widely used symbols that look like keywords at a initial glance are, in fact, name which can be redeclared- all of that [without a preprocessor!](http://tigcc.ticalc.org/doc/cpp.html). One of them, the _this_ construct has already been mentioned; undefined, on the other hand, is another example which opens the chance for very pleasant debugging. For instance, this is perfectly valid:

{% highlight javascript %}
//buried somewhelse
undefined = "hello, hello\nI dont' know why you say good bye and you say hello!";

//common js checking
if (beatles == undefined)
    removeFromPlaylist('beatles');

{% endhighlight %}

To top it off, even [strings can be keywords sometimes](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Strict_mode).

* Duck typing

JavaScript brings to the table all the worst of dynamic typing. Like a duck who quacks like a duck, swims like a duck, walks like a duck but is in fact a platypus.


Do not let people fool you with talk of ES6, ES7 or ESn, strinct mode or anyting of the sort- the only viable JavaScript is TypeScript.
