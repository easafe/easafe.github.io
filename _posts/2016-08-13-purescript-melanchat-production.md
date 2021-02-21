---
layout: default
title: PureScript in production
tags: [cs]
---

[MelanChat](https://melan.chat) is a friendly random chat. Think omegle but without pervs, and you also got to have a full profile. Being a personal project, I enjoy the freedom to pick the stack, but having something reliable with some guarantee of correctness is paramount if I am to save and make the best of my sparse free time. I ended up choosing PureScript -- for both server and client side.

Then, since the app is finally being used by more people than just me, I join the cliché to share my experiences with a less popular language in production. As it is usual, let us divide it into three moral categories:

### The Good

PureScript often gets compared to Haskell (as a matter of fact, I am not sure if there are any PureScript programmers who were not Haskell programmers before), but some of the dissimilarities make its better parts. Records, and row polymorphism, are great. Partial pattern matching should also be a compilation error by default in Haskell. Some features that would be Haskell extensions are always on, e.g., scoped type variables, type literals, existential types, functional dependencies, etc. Sometimes I miss lazy evaluation, specially since it enables ~~O(n^4) loops~~ more straightforward algorithms, but I suppose the debate on the trade offs will go on forever.

The chief reason why MelanChat isn't written in Haskell is also, in my opinion, the biggest selling point of PureScript: sharing code between server and client and reusing types. I know there is GHCJS, PureScript bindings to frameworks and many other bridges between a would be Haskell-server-side, PureScript-client-side application but none of these beat the ergonomics and spared mental overhead of using a single language. Sharing types eliminates a whole class of bugs and a lot of duplication. Then, the cherry on top, finally no more mismatched type issues in HTTP requests (in MelanChat's case, thanks to [purescript-payload](https://pursuit.purescript.org/packages/purescript-payload/)). 

### The Bad

Of course, nothing is perfect. PureScript, in the same vein of Haskell, suffers from slow (and memory hungry) compilation times. Tooling often is not as polished as one would desire, either. As expected of a niche language, library availability, specially server side, can be sparse (In MelanChat, for example, I wrote my own [web framework](https://github.com/easafe/purescript-flame), since none of the existing quite achieved the simplicity and speed ratio I wished for). Being able to access JavaScript libraries, via unsafe FFI bindings, makes up for it, but it is also a double edged sword, since it is, well, unsafe JavaScript.

If, for some reason, you have to rely heavily on FFI, there is currently no way to DRY your code, mainly because PureScript does not support ES modules. This also makes it impossible to use libraries that don't offer alternatives. And what's worse, a PureScript bundled browser app is near useless without ad hoc tree shaking like [zephyr](https://github.com/coot/zephyr) since regular JavaScript tools like webpack can't optimize the dozens of MBs produced CommonJS modules.   

### The Ugly

In the end of the day, your fancy typed constructs are still compiled down to JavaScript. You might, or should I say will, have at some point to debug the output code. From this point on, all the drawbacks of using node.js and run of the mill dynamic typing apply. Since you also share the JavaScript ecosystem, the madness and needless complexity of web programming are also present, should you make anything not trivial. 

Maybe in the future these pain points will be alleviated by wasm, however MelanChat is not a mission critical application -- no one will die if they can't make more karma from chatting -- so so far, so good. Why not [sign up then and give it a try :)](https://melan.chat)? 
