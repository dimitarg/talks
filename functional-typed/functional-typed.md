# Functional programming is typed
(This post is work in progress)

## Intro

In this post I would like to argue that there is no such thing as non-typed functional programming.
To put it another way, I argue that functional programming is **always** typed, regardless of tools or language
specifics, or our subjective views on the matter.

Then, with the above claim in mind, I would like to show that some parts of the static vs dynamic debate are
nonsensical and/or useless, throw them away, and reduce the debate to an essential, normalized form; and ask
the reader to pick up the debate from there, or suggest another normalized form which is more grounded than mine.



## What functional programming is

I believe that by now there is an established, non-controversial definition of functional programming.

> Functional programming is programming with functions.

(**I believe that this was said by @runarorama - I'm not 100% sure, please contact me if
the attribution is wrong**)

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
- To define a function, one needs to **explicitly define the domain and codomain** of the function

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

If functional programming is programming with functions, and functions are typed, then 
functional programming must be typed.

There is no deeper meaning in this paragraph. It is just a logical argument - hopefully, a sound one.

## Don't take my word for it

Here is a [mathematician on the internet](https://youtu.be/6t6bsWVOIzs?t=428) saying that functions are typed.

P.S. The above keynote, by Dr. Emily Riehl, is really cool and you should watch it if you are interested in computation. It is also
the final thing that prompted me to write this post.


## Your language does not matter - concepts come first

That functional programming itself is typed, if you decide to acknowledge it, does not pertain in any way to whether your
language of choice is statically or dynamically typed. This is not a claim in itself, but an instance of a more
general scheme:

(Mathematical) objects are, well, what they are - they are well-defined by their structure and properties;
it is our degree of understanding of them that may vary, or our representation of these objects in our tools.
Those are external and subjective facets, not intrinsic to the objects of study; the objects stay the same regardless.


## Uni-typed versus implicitly typed

A dynamically-typed language can be viewed as a statically typed language where the sole static type is
a product type of all the types representable at runtime. In fact, it has
[been argued](https://existentialtype.wordpress.com/2011/03/19/dynamic-languages-are-static-languages/)
that dynamic languages not only can be viewed as, but in the end are, 'uni-typed'.

In practice things are worse.




## On the dynamic versus static debate

## On the nature of things



  



