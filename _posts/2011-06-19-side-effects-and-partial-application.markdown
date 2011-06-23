---
layout: post
title: "F#: Use care mixing side-effects and partial application"
date: 2011-06-19
---

Here's a lesson I learned while writing
[llvm-fs](https://github.com/keithshep/llvm-fs). Do you see the problem in my
buggy_iprintfn function?

{% highlight ocaml %}
(* an indented version of the printf function *)
let buggy_iprintfn depth fmt =
    for i = 0 to depth - 1 do
        printf "\t"
    printfn fmt
{% endhighlight %}

It works fine if you apply all arguments each time you call it like:

{% highlight ocaml %}
buggy_iprintfn 2 "hey %s, what's up" "billy"
buggy_iprintfn 2 "hey %s, what's up" "bob"
buggy_iprintfn 2 "hey %s, what's up" "billy-bob"
{% endhighlight %}

Output:
<pre>
        hey billy, what's up
        hey bob, what's up
        hey billy-bob, what's up
</pre>

But fails to indent anything after the first name if you call it like:

{% highlight ocaml %}
List.iter (buggy_iprintfn 2 "hey %s, what's up") ["billy"; "bob"; "billy-bob"]
{% endhighlight %}

Output:
<pre>
        hey billy, what's up
hey bob, what's up
hey billy-bob, what's up
</pre>

The problem occurs because buggy_iprintfn returns a
[function value](http://msdn.microsoft.com/en-us/library/dd233158.aspx) and the
(print "\t") side effects occur before the (printfn fmt) function value
is returned. As a side-note this bug is a good argument for the haskell approach
of isolating side effects to [monads](http://www.haskell.org/haskellwiki/Monad)
where this bug cannot occur. The iprintfn function
below fixes the bug by moving all side effects into printIndented which will not
execute until all arguments are applied to the returned function value.

{% highlight ocaml %}
(* an indented version of the printf function *)
let iprintfn depth fmt =
    let printIndented s =
        for i = 0 to depth - 1 do
            printf "\t"
        printfn "%s" s
    Printf.ksprintf printIndented fmt

List.iter (iprintfn 2 "hey %s, what's up") ["billy"; "bob"; "billy-bob"]
{% endhighlight %}

Output:
<pre>
        hey billy, what's up
        hey bob, what's up
        hey billy-bob, what's up
</pre>

