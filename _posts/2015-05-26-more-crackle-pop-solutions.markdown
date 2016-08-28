---
layout: post
title:  "Other Recurser's solutions to CrackePop!"
date:   2015-05-26 14:15:00
categories: recurse-center
comments_enabled: true
---

Some weeks ago a post about "CracklePop" in which I posted two implementations. The post can be found [here][1]. After the post, other Recurser's shared with three more solutions. I decided to post them here.

[Mark Dominus][2]'s solution in Perl.

```perl
my @num = ("") x 101;
$num[$_*3] .= "Crackle" for 1..33;
$num[$_*5] .= "Pop"     for 1..20;
$num[$_] ||= $_ for 1..100;
shift @num;  # discard element 0
print join "\n", @num, "";
```

[James Keene][3]'s solution.

```
def fizzbuzz(a): 
return [a, "Fizz", "Buzz", "FizzBuzz"][(0x1241843 >> ((a % 15) * 2)) & 3] 

map(fizzbuzz,xrange(1,100))
```

At last, with [Jeff Fowler][4]'s solution in Brain Fuck which can be found [here][5]

I am really amazed to see how many different solutions can be to a very trivial problem. 

Mark's solution I could understand just by reading the code. In order to understand James's solution I re-writed in a language I knew and debugged the code in order to understand how it worked. 

Jeff's solution I gave up! It is in Brain Fuck and I really do not understand it at all. I found really cool to get to know a really strange computer languaged which I never heard before.

A big thanks to all who shared their solution!

[1]: http://pietro.menna.net.br/recurse-center/2015/05/08/crackle-pop/
[2]: https://twitter.com/mjdominus
[3]: https://twitter.com/_jak
[4]: https://github.com/urthbound
[5]: http://repl.it/mdb