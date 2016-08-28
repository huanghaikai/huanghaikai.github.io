---
layout: post
title:  "Half Journey"
date:   2015-05-07 14:00:00
categories: recurse-center
comments_enabled: true
---

Today is exactly the middle of my Batch at [Recurce Center][1]. Six weeks has passed and I just realized that I learned some stuff, but I have still much to learn here. 

I just set for myself the objectives for the rest of the my Batch which will be to try to contribute to a OSS project while continuing to learn [Clojure][2].

My plan is to post about what I learn (and my findings) but trying to avoid keeping too personal.

# So what I learned today? #

## Resources for Closure ##

The list below is a **must have favorites** links for every person how would like to learn Clojure or plans to use it.

* [Grimoire][3]: A Cheatsheet of many Clojure functions presented in a nice way!
* [ClojureDocs][4]: Is the community-powered documentation and examples repository.
* [Clojure Werkz][5] A growing collection of open source Clojure libraries

## Closures ##

**What are they? What does it mean to have a Closure?**

First, the definition by wikipedia:

	In programming languages, closures (also lexical closures or function closures) are a technique for implementing lexically scoped name binding in languages with first-class functions. Operationally, a closure is a data structure storing a function together with an environment.

An example, implementing Power of function using Closures.

	(defn pow [initial]
	  (fn [x] (reduce * (repeat initial x))))

Calling the code above in the REPL:

	user=> ((pow 2) 3)
	9
	user=> ((pow 6) 3)
	729
	user=> ((pow 3) 3)
	27

So what just happened? We created a **function** which returns **another function** which happens to have an initial value stored on it's instance in the memory.

I am not sure yet, but I can say that the *initial* variable, is now a "private attribute" of the instance of function *pow* which gets in the run time. 

It seems a very powerful resource!

[1]: https://recurse.com
[2]: http://clojure.org
[3]: http://conj.io/
[4]: http://clojuredocs.org/
[5]: http://clojurewerkz.org/