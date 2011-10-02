---
layout: post
title: "fig: building a compiler with F# and LLVM"
date: 2011-10-02
---

I've been using most of my spare time lately building fig: a .NET compiler using [LLVM](http://llvm.org/) and [F#](http://msdn.microsoft.com/en-us/fsharp). So, I wanted to create a little post describing what I'm doing, why I'm doing it and where I am.

## What I'm building

The main idea is to build a .NET compiler which is designed to perform well for the kind of computationally intensive problems seen in fields like machine learning, computational finance, computational biology and other computational sciences. With this goal in mind the compiler implementation should:

* optimize aggressively: given that we can expect long runtimes with large memory requirements we are much more willing to trade compile time for efficient machine code. Fortunately LLVM already solves the problem of low level optimizations which should allow me to focus on higher level optimizations specific to the .NET runtime and code structure. Longer compile times also means that being able to do ahead of time (AOT) compilation is a requirement and just in time (JIT) compilation will be treated as a nice-to-have.

* have a GC which scales well to many gigs of memory and many cores since computationally intensive code often means both. The GC must also be reliable over long runs and capable of efficiently handling: memory fragmentation with many small and/or few large heap allocations, short and long living objects. All easier said of course...

## Why I'm building it

I started this project very reluctantly. I work as a scientific programmer and in my ever-humble opinion F# is the best language to use for a good chunk of the problems I work on, but I have not had good luck using mono on computationally intensive problems. This is not at all a knock against mono. I think mono is great for what 90% of .NET users want: reasonably fast performance for typical desktop and web applications, but I need more than that. I was also working on a machine learning project on the side (which is on pause now) which has a somewhat unique requirement of being able to sandbox threads so that they could "crash" in any way without affecting the rest of the process.

Also, generating more efficient code means that your computers will use less electricity which means that less CO2 will be released into the atmosphere, thereby slowing the rise in global temperatures and preserving the dwindling habitat of the majestic polar bear. Who doesn't want to save polar bears?

<center><img src="/images/usgs-polar-bear.jpg"/></center>

Given all that I had to decide between dropping F# for a lot of the problems I wanted to use it for, trying to modify mono or vmkit to do what I wanted it to or to build a new compiler. Luckily building a new compiler doesn't mean building a compiler from scratch. I am leveraging great projects like LLVM and [mono-cecil](http://www.mono-project.com/Cecil) which will save a ton of time and lead to a better final product. Another bonus of building a new compiler is that you can pick whichever language you want for implementation. I'm using ... F# :)

## Where I am

To be completely realistic, building a performant compiler ready for general use is a huge job. Even with all of libraries and tools to help with compiler construction success is in no way guaranteed. Having said that things are starting to come together:

* I've created an F# binding for LLVM that I'm using for my compiler. It's available as an open source project on github using the same license as LLVM so if you want to build your own compiler in F# you can use this (if you decide to do this you should also check out [fslex and fsyacc](http://fsharppowerpack.codeplex.com/)).

* I've gotten the compiler to the point where it can successfully generate linkable object code for some very simple .NET assemblies with no GC or runtime. There is really a ton more to do here since I haven't gotten started on a garbage collector yet and there are many instructions that are not yet implemented.

This project is clearly something I'd like to use for my own work but ultimately I hope that it becomes something that is useful tool for anyone that wants to do high performance computing with .NET. I should also say that I haven't yet given much thought to the issue of licensing but ideally I would like to find some balance between making it possible for people to use it freely for non-commercial work and getting paid for commercial use. Mentioning licensing so early in development seems strange to me but I want to be sure I don't create a false expectation.

## Just for fun, some examples of F# compiled to LLVM

### Simple Add

It doesn't get more basic than adding two ints:

{% highlight ocaml %}
let add (x : int) (y : int) = x + y
{% endhighlight %}

{% highlight llvm %}
define i32 @add(i32 %x, i32 %y) nounwind readnone {
entry:
  %tmpAdd = add i32 %y, %x
  ret i32 %tmpAdd
}
{% endhighlight %}

### Recursion

Some basic recursive function definitions:

{% highlight ocaml %}
let rec gcd (x : int) (y : int) =
    if x = y then
        x
    elif x < y then
        gcd x (y - x)
    else
        gcd (x - y) y
{% endhighlight %}

{% highlight llvm %}
define i32 @gcd(i32 %x, i32 %y) nounwind readnone {
entry:
  br label %block_0.outer

block_0.outer:                                    ; preds = %block_11, %entry
  %yAlloca.0.ph = phi i32 [ %y, %entry ], [ %tmpSub, %block_11 ]
  %xAlloca.0.ph = phi i32 [ %x, %entry ], [ %xAlloca.0, %block_11 ]
  %tmp = sub i32 0, %yAlloca.0.ph
  br label %block_0

block_0:                                          ; preds = %block_21, %block_0.outer
  %indvar = phi i32 [ 0, %block_0.outer ], [ %indvar.next, %block_21 ]
  %tmp3 = mul i32 %indvar, %tmp
  %xAlloca.0 = add i32 %xAlloca.0.ph, %tmp3
  %brTest = icmp eq i32 %xAlloca.0, %yAlloca.0.ph
  br i1 %brTest, label %block_5, label %block_7

block_5:                                          ; preds = %block_0
  ret i32 %xAlloca.0

block_7:                                          ; preds = %block_0
  %brTest4 = icmp slt i32 %xAlloca.0, %yAlloca.0.ph
  br i1 %brTest4, label %block_11, label %block_21

block_11:                                         ; preds = %block_7
  %tmpSub = sub i32 %yAlloca.0.ph, %xAlloca.0
  br label %block_0.outer

block_21:                                         ; preds = %block_7
  %indvar.next = add i32 %indvar, 1
  br label %block_0
}
{% endhighlight %}

{% highlight ocaml %}
let rec fib = function
    | 0 -> 0
    | 1 -> 1
    | n -> fib (n - 1) + fib (n - 2)
{% endhighlight %}

{% highlight llvm %}
define i32 @fib(i32 %_arg1) nounwind readnone {
entry:
  switch i32 %_arg1, label %block_14 [
    i32 0, label %block_33
    i32 1, label %block_35
  ]

block_14:                                         ; preds = %entry
  %tmpSub = add i32 %_arg1, -1
  %callResult = tail call i32 @fib(i32 %tmpSub)
  %tmpSub3 = add i32 %_arg1, -2
  %callResult4 = tail call i32 @fib(i32 %tmpSub3)
  %tmpAdd = add i32 %callResult4, %callResult
  ret i32 %tmpAdd

block_33:                                         ; preds = %block_35, %entry
  %merge = phi i32 [ 0, %entry ], [ 1, %block_35 ]
  ret i32 %merge

block_35:                                         ; preds = %entry
  br label %block_33
}
{% endhighlight %}

A mutually recursive function with tail call optimization:

{% highlight ocaml %}
let rec isEven = function
    | 0u -> true
    | n  -> isOdd (n - 1u)
and isOdd = function
    | 0u -> false
    | n  -> isEven (n - 1u)
{% endhighlight %}

{% highlight llvm %}
define i32 @isEven(i32 %_arg1) nounwind readnone {
entry:
  br label %tailrecurse

tailrecurse:                                      ; preds = %block_10.i, %entry
  %indvar = phi i32 [ %indvar.next, %block_10.i ], [ 0, %entry ]
  %tmp = mul i32 %indvar, -2
  %_arg1.tr = add i32 %tmp, %_arg1
  %cond = icmp eq i32 %_arg1.tr, 0
  br i1 %cond, label %isOdd.exit, label %block_10

block_10:                                         ; preds = %tailrecurse
  %cond.i = icmp eq i32 %_arg1.tr, 1
  br i1 %cond.i, label %isOdd.exit, label %block_10.i

block_10.i:                                       ; preds = %block_10
  %indvar.next = add i32 %indvar, 1
  br label %tailrecurse

isOdd.exit:                                       ; preds = %block_10, %tailrecurse
  %callResult1 = phi i32 [ 0, %block_10 ], [ 1, %tailrecurse ]
  ret i32 %callResult1
}

define i32 @isOdd(i32 %_arg2) nounwind readnone {
entry:
  %cond = icmp eq i32 %_arg2, 0
  br i1 %cond, label %block_22, label %block_10

block_10:                                         ; preds = %entry
  %tmpSub = add i32 %_arg2, -1
  %callResult = tail call i32 @isEven(i32 %tmpSub)
  ret i32 %callResult

block_22:                                         ; preds = %entry
  ret i32 0
}
{% endhighlight %}

### Simple Objects

A point type (which is a class according to the .NET runtime) with the resulting LLVM getters and constructor ".ctor":

{% highlight ocaml %}
[<ReferenceEquality>]
type Point = {x : float; y : float; z : float}
{% endhighlight %}

{% highlight llvm %}
%"SimpleFunctions/Point" = type { double, double, double }

define double @get_x(%"SimpleFunctions/Point"* nocapture %"0") nounwind readonly {
entry:
  %fieldPtr = getelementptr inbounds %"SimpleFunctions/Point"* %"0", i32 0, i32 0
  %fieldValue = load double* %fieldPtr
  ret double %fieldValue
}

define double @get_y(%"SimpleFunctions/Point"* nocapture %"0") nounwind readonly {
entry:
  %fieldPtr = getelementptr inbounds %"SimpleFunctions/Point"* %"0", i32 0, i32 1
  %fieldValue = load double* %fieldPtr
  ret double %fieldValue
}

define double @get_z(%"SimpleFunctions/Point"* nocapture %"0") nounwind readonly {
entry:
  %fieldPtr = getelementptr inbounds %"SimpleFunctions/Point"* %"0", i32 0, i32 2
  %fieldValue = load double* %fieldPtr
  ret double %fieldValue
}

define void @.ctor(%"SimpleFunctions/Point"* nocapture %"0", double %x, double %y, double %z) nounwind {
entry:
  %fieldPtr = getelementptr inbounds %"SimpleFunctions/Point"* %"0", i32 0, i32 0
  store double %x, double* %fieldPtr
  %fieldPtr3 = getelementptr inbounds %"SimpleFunctions/Point"* %"0", i32 0, i32 1
  store double %y, double* %fieldPtr3
  %fieldPtr5 = getelementptr inbounds %"SimpleFunctions/Point"* %"0", i32 0, i32 2
  store double %z, double* %fieldPtr5
  ret void
}
{% endhighlight %}

Given a point calculate the distance from (0, 0, 0) squared:

{% highlight ocaml %}
let distSq p = p.x * p.x + p.y * p.y + p.z * p.z
{% endhighlight %}

{% highlight llvm %}
define double @distSq(%"SimpleFunctions/Point"* nocapture %p) nounwind readonly {
entry:
  %fieldPtr = getelementptr inbounds %"SimpleFunctions/Point"* %p, i32 0, i32 0
  %fieldValue = load double* %fieldPtr
  %tmpFMul = fmul double %fieldValue, %fieldValue
  %fieldPtr5 = getelementptr inbounds %"SimpleFunctions/Point"* %p, i32 0, i32 1
  %fieldValue6 = load double* %fieldPtr5
  %tmpFMul10 = fmul double %fieldValue6, %fieldValue6
  %tmpFAdd = fadd double %tmpFMul, %tmpFMul10
  %fieldPtr12 = getelementptr inbounds %"SimpleFunctions/Point"* %p, i32 0, i32 2
  %fieldValue13 = load double* %fieldPtr12
  %tmpFMul17 = fmul double %fieldValue13, %fieldValue13
  %tmpFAdd18 = fadd double %tmpFAdd, %tmpFMul17
  ret double %tmpFAdd18
}
{% endhighlight %}

### Simple Arrays

Given an array of floats, calculate it's average:

{% highlight ocaml %}
let avg (xs : float array) =
    let mutable sum = 0.0
    for x in xs do
        sum <- sum + x
    sum / float xs.Length
{% endhighlight %}

{% highlight llvm %}
%0 = type { i32, double* }

define double @avg(%0* nocapture %xs) nounwind readonly {
entry:
  %lenAddr = getelementptr inbounds %0* %xs, i32 0, i32 0
  %len1 = load i32* %lenAddr
  %brTest2 = icmp sgt i32 %len1, 0
  br i1 %brTest2, label %block_15.lr.ph, label %block_37

block_15.lr.ph:                                   ; preds = %entry
  %arrPtrAddr = getelementptr inbounds %0* %xs, i32 0, i32 1
  %arrPtr = load double** %arrPtrAddr
  %tmp = icmp sgt i32 %len1, 1
  %len.op = add i32 %len1, -1
  %0 = zext i32 %len.op to i64
  %.op = add i64 %0, 1
  %tmp8 = select i1 %tmp, i64 %.op, i64 1
  br label %block_15

block_15:                                         ; preds = %block_15, %block_15.lr.ph
  %indvar = phi i64 [ 0, %block_15.lr.ph ], [ %indvar.next, %block_15 ]
  %Alloca.03 = phi double [ 0.000000e+00, %block_15.lr.ph ], [ %tmpFAdd, %block_15 ]
  %elemAddr = getelementptr double* %arrPtr, i64 %indvar
  %elem = load double* %elemAddr
  %tmpFAdd = fadd double %Alloca.03, %elem
  %indvar.next = add i64 %indvar, 1
  %exitcond = icmp eq i64 %indvar.next, %tmp8
  br i1 %exitcond, label %block_37, label %block_15

block_37:                                         ; preds = %block_15, %entry
  %Alloca.0.lcssa = phi double [ 0.000000e+00, %entry ], [ %tmpFAdd, %block_15 ]
  %convVal = sitofp i32 %len1 to double
  %tmpFDiv = fdiv double %Alloca.0.lcssa, %convVal
  ret double %tmpFDiv
}
{% endhighlight %}

Wow! This is a lot of LLVM code to calculate the average of four numbers. Part of the reason is that the LLVM optimizer inlined the "avg" function and the other part is that we have to construct a new array object from the parameters.

{% highlight ocaml %}
let avgOfFour (a : float) (b : float) (c : float) (d : float) =
    avg [|a; b; c; d|]
{% endhighlight %}

{% highlight llvm %}
define double @avgOfFour(double %a, double %b, double %c, double %d) nounwind {
entry:
  %malloccall = tail call i8* @malloc(i32 mul (i32 ptrtoint (double* getelementptr (double* null, i32 1) to i32), i32 4))
  %newArr = bitcast i8* %malloccall to double*
  %malloccall1 = tail call i8* @malloc(i32 ptrtoint (%0* getelementptr (%0* null, i32 1) to i32))
  %newArrObj = bitcast i8* %malloccall1 to %0*
  %lenAddr = getelementptr inbounds %0* %newArrObj, i32 0, i32 0
  store i32 4, i32* %lenAddr
  %arrPtrAddr = getelementptr inbounds %0* %newArrObj, i32 0, i32 1
  store double* %newArr, double** %arrPtrAddr
  store double %a, double* %newArr
  %elemAddr5 = getelementptr double* %newArr, i32 1
  store double %b, double* %elemAddr5
  %elemAddr8 = getelementptr double* %newArr, i32 2
  store double %c, double* %elemAddr8
  %elemAddr11 = getelementptr double* %newArr, i32 3
  store double %d, double* %elemAddr11
  %len1.i = load i32* %lenAddr
  %brTest2.i = icmp sgt i32 %len1.i, 0
  br i1 %brTest2.i, label %block_15.lr.ph.i, label %avg.exit

block_15.lr.ph.i:                                 ; preds = %entry
  %tmp.i = icmp sgt i32 %len1.i, 1
  %len.op.i = add i32 %len1.i, -1
  %0 = zext i32 %len.op.i to i64
  %.op.i = add i64 %0, 1
  %tmp8.i = select i1 %tmp.i, i64 %.op.i, i64 1
  br label %block_15.i

block_15.i:                                       ; preds = %block_15.i, %block_15.lr.ph.i
  %indvar.i = phi i64 [ 0, %block_15.lr.ph.i ], [ %indvar.next.i, %block_15.i ]
  %Alloca.03.i = phi double [ 0.000000e+00, %block_15.lr.ph.i ], [ %tmpFAdd.i, %block_15.i ]
  %elemAddr.i = getelementptr double* %newArr, i64 %indvar.i
  %elem.i = load double* %elemAddr.i
  %tmpFAdd.i = fadd double %Alloca.03.i, %elem.i
  %indvar.next.i = add i64 %indvar.i, 1
  %exitcond = icmp eq i64 %indvar.next.i, %tmp8.i
  br i1 %exitcond, label %avg.exit, label %block_15.i

avg.exit:                                         ; preds = %block_15.i, %entry
  %Alloca.0.lcssa.i = phi double [ 0.000000e+00, %entry ], [ %tmpFAdd.i, %block_15.i ]
  %convVal.i = sitofp i32 %len1.i to double
  %tmpFDiv.i = fdiv double %Alloca.0.lcssa.i, %convVal.i
  ret double %tmpFDiv.i
}
{% endhighlight %}

