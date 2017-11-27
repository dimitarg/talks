# Functional programming is typed
(This post is work in progress)

## Intro

In this post I would like to argue that there is no such thing as untyped functional programming.
To put it another way, I argue that functional programming is **always** typed. Therefore terms such as 
'untyped functional programming' / 'non-typed functional programming' are ill-defined; and by extension,
arguments based on such terms are ill-defined in their genesis.

The above claim should read exactly as it is written - all functional programming is typed. Specifically, 
this post is not a dynamic vs static languages post - although I will, inevitably, touch on the subject.

## What functional programming is

I believe that by now there is an established, non-controversial definition of functional programming.

> Functional programming is programming with functions.

'Functions' here means referentially transparent, total functions, as in 'mathematical functions'.
Side effects aside, the majority of our program is comprised of mathematical functions. We do this
because functions compose, and therefore our programs, consisting of functions, compose. 

This composition is a good thing and it is the raison d'être of functional programming.


## Digression - 'Side effects aside'
*For completeness of argument let us note that what functional programmers usually do is keep most of the program pure,
and allow for side effects in an isolated, disciplined matter at the 'end of the world'. We will not talk about the latter
here as it is irrelevant to our discussion; the subject is very vast; and I am (yet) not qualified to 
talk about it.*

*Google, in no particular order, 'IO Monad', 'Free Monad', 'Interpreter',
'Kleisli', for starting points on this subject.*

## What a function is

Hopefully, at this point
- We have established that functional programming means programming with functions, to the greatest extent feasible;
- The above claim is non-controversial to the reader.

Let us go back to school then, and explicitly say **what a function is**.

> A function takes each element of a set `A` (the input set) to exactly one element of a set `B` (the output set)

An example of a relation R that is not a function is one where one element of `A` relates to more than one element of `B`,
for example:

```
A = {1,2,3}
B = {true, false}
R = {(1, true), (2, false), (3, true), (3, false)}
```
`3` in `A` maps to both `true` and `false` in B, therefore R is not a function.

Let us now be specific about a couple of clarifications arising from the above definition of function:
- A function is **total** - each element of the domain (input set) must be mapped to some element of the codomain (output set)
- To define a function, one needs to explicitly define the domain and codomain of the function

Let us expand on the latter point.

## Functions are typed

We do need an explicit domain and codomain to define a function. If there is no domain and codomain, what we are
looking at is not a function. Let's give an example:

```
f(x) = x * 3
```

Is the above formula a function? By itself, no. Does it work on integers? Does it work on reals?
We cannot say, because it is not written anywhere.

One might say, 'it works on numbers', as in 'number-like things that have multiplication defined'. 
What about vectors, then? The way it's written, it might as well be a function where x is a vector.

```
f([1,2,3])
>[3, 6, 9]
```

Probably not what we were expecting when we looked at the formula.
Fortunately there is an easy way to spare us these headaches. Just stick to maths, and keep things explicit - 
if the domain or codomain is not defined, it is not a function.

## Functional programming is typed

If functional programming is programming with functions, and functions are typed, then functional programming is typed.
There is no deeper meaning in this paragraph. It is just a logical argument - hopefully, a sound one.

## Your language does not matter - concepts come first

## Uni-typed versus implicitly typed

## On the nature of things



  



