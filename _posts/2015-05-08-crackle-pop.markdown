---
layout: post
title:  "CracklePop"
date:   2015-05-08 15:50:00
categories: recurse-center
comments_enabled: true
---

I remember when I applied to [Recurse Center][1], I got asked to write a program called "Crackle Pop". I just wonder, do all the solutions provided by all the applicants look really different? (I personally doubt it).

## How I wrote it ##

I literally just copied the instructions and transformed it to code. I did not care if it was the best solution, or the most elegant, or even the most perfomance efficient. I actually wrote it in Ruby, but since I am learning Clojure, I will write in Clojure.

The specs:

	Write a program that prints out the numbers 1 to 100 (inclusive). If the number is divisible by 3, print Crackle instead of the number. If it's divisible by 5, print Pop. If it's divisible by both 3 and 5, print CracklePop. You can use any language.

The code:

```clojure
(defn cracklepop [x]
  (cond
    (= 0 (mod x 3) (mod x 5)) (println "CracklePop")
    (= (mod x 3) 0) (println "Crackle")
    (= (mod x 5) 0) (println "Pop")
    :else (println x)))

(map cracklepop (range 1 101))
```

## Let`s try to write it different ##

### Do not use conditionals!!! ###

It seems crazy, right? The spec already contains 3 **ifs** statements on its own. So it is so logical to write it using *ifs* or *conditional* (switch in other languages) statements.

```clojure
(defn cracklepop [x]
  (let [solution-matrix {0 "CracklePop"
                         3 "Crackle"
                         5 "Pop"
                         6 "Crackle"
                         9 "Crackle"
                         10 "Pop"
                         12 "Crackle"}]
  (println (solution-matrix (mod x 15) x))))

(map cracklepop (range 1 101))
```

This solution does not use conditionals, but a "rotating matrix".

## What Solution is better? ##

**I don't know and I have not measured the runtimes of each solution.** I just know that if I was asked during a job interview to write without conditionals, maybe I could have blanked for some minutes. This is because I never thought of the problem with that constrait until now. (Also the preasure of the interview would have added up).

But for sure, I still find the first solution more readable for being closer to the domain of the "requirements".

## Conclusions ##

There are many solutions to a same problem. Deciding on a solution should depend on the contraints defined by the non-functional requirements.

I also still think that solutions should be written as close to as possible to the domain of the problem. This way the code keeps readable.

[1]: http://www.recurse.com