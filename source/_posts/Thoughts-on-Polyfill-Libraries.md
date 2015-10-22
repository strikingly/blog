title: Thoughts on Polyfill Libraries
date: 2015-10-22 16:22:15
author: glen
tags:
- javascript
- polyfill
categories:
- Frontend
- English
---

This the first post of a series on web API polyfill libraries. In this post, we are going to discuss why polyfill is a good idea and what are some of the cool modern web APIs coming to browsers. In the second one, we will discuss some of the drawbacks current polyfill libraries have and also challenges of writing good polyfill libraries. And finally, we take polyfill libraries to the next level by discussing a whole new way of consuming them.

## Introduction to Polyfill

The term polyfill in front end world means code that provides APIs the browser is supposed to offer natively, if they are currently missing. Here is some sample polyfill code, which makes `Date.now` defined in ECMAScript v5 available to all browsers:

```
if (!Date.now) {
    Date.now = function now() {
        `return new Date().getTime()`
    }
}
```

A polyfill library is in direct contrast to a library like jQuery, where it invents its own APIs to provide the missing implementations:

```
timeLib.epoch = Date.now || function now() {
    `return new Date().getTime()`
}
```

Given the two styles of methods to bridge the gaps between browsers with varying degree of standard conformance, which one is better?

## Comparison of Polyfill Libraries and Alternatives

Inventing custom APIs was a good idea back in the old days, where extending native objects was [_really dangerous_](http://perfectionkills.com/whats-wrong-with-extending-the-dom/), because how they should behave was not well defined. But with the up coming [_Web IDL specification_](http://www.w3.org/TR/WebIDL/) and most nonconforming browsers have lost market share significantly, extending native objects is no longer a pandora box.

Custom APIs also have a very big issue: it creates fragments unnecessarily. A jQuery slider won't work, for example, unless the jQuery library is also included in the page. Same goes for components written using Dojo, Prototype, etc. So these different UI component collections are separated simply because the DOM libraries they depend on have different APIs for doing the same things.

You might say these DOM libraries exist for a reason. Native APIs like `XMLHttpRequest`, DOM operations are too awkward to use comparing to their counterparts in the third party libraries. The good news is that web standard has come a long way since these libraries were written.

We have promise based [fetch](https://fetch.spec.whatwg.org/) to replace callback based `XMLHttpRequest`:

```
// return a promise object
var fetched = fetch('/users.html')
fetched.then(function(response) {
    return response.text()
}).then(function(body) {
    document.body.innerHTML = body
})
```

[DOM operations](https://dom.spec.whatwg.org/) just get immensely better, lots of jQuery like APIs. Just to list a few:

```
node.prepend(childNode)
node.append(childNode)
node.replaceWith(newNode)
// find the first ".foo .bar" element in node
node.query('.foo .bar')
// find all a elements in li elements that are direct children of node
// and yes, the return value is a true array
node.queryAll('>li a')
```

[ECMAScript 2015](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-properties-of-the-promise-prototype-object) for better JavaScript syntax and, and in this case what we are really interested in, better standard library:

```
Object.assign({a: 1}, {b: 2}) // aka, extend
Array.from({'0': 0, '1': 1}) // aka, [].slice.call(...)
```

And [Web animations](http://www.w3.org/TR/web-animations/) for writing sophisticated and butter smooth animations:

```
// fade in node in one second
node.animate([{
    opacity: 0
}, {
    opacity: 1
}], 1000)
```

 Basically every sub component of jQuery has a modern native counterpart now. And there exist polyfill libraries for all these APIs.

You might ask, what's the difference between depending on a polyfill library and a third party library? Why depending on polyfill libraries won't fragment components?

Good questions.

First, components written using modern native APIs don't depends on polyfill libraries. In a competent browser, there is no need to include any polyfill library at all. Instead, they depend on a competent browser, and polyfill libraries upgrade an incompetent browser to a competent one.

And since they only depend on native APIs, which every browser is supposed to agree on, there is really no fragmentation.

## Conclusion

Web platform is getting more and more pleasant to program on, with the prolific web standards that have been pumped out in recent years. Bringing these pleasant experience to as many browsers as possible can be really beneficial for every developer.

But in the next post, we will find out why it's not an easy task, and discuss some of the challenges polyfill libraries might face.
