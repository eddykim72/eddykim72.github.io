---
title: "Sequence Point"
excerpt: "시퀀스 포인트"
share: false
date: 2018-06-22
sidebar:
    nav: "menu_notes"
---

A sequence point is a point in time at which the dust has settled and all side effects which have been seen so far are guaranteed to be complete.

The sequence points listed in the C standard are: 
•at the end of the evaluation of a full expression (a full expression is an expression statement, or any other expression which is not a subexpression within any larger expression); 
•at the ||, &&, ?:, and comma operators; and 
•at a function call (after the evaluation of all the arguments, and just before the actual call). 


The Standard states that 

Between the previous and next sequence point an object shall have its stored value modified at most once by the evaluation of an expression. Furthermore, the prior value shall be accessed only to determine the value to be stored. 


These two rather opaque sentences say several things. First, they talk about operations bounded by the ``previous and next sequence points''; such operations usually correspond to full expressions. (In an expression statement, the ``next sequence point'' is usually at the terminating semicolon, and the ``previous sequence point'' is at the end of the previous statement. An expression may also contain intermediate sequence points, as listed above.) 

The first sentence rules out both the examples 
	i++ * i++

and 	i = i++

from questions 3.2 and 3.3--in both cases, i has its value modified twice within the expression, i.e. between sequence points. (If we were to write a similar expression which did have an internal sequence point, such as 	i++ && i++

it would be well-defined, if questionably useful.) 

The second sentence can be quite difficult to understand. It turns out that it disallows code like 
	a[i] = i++

from question 3.1. (Actually, the other expressions we've been discussing are in violation of the second sentence, as well.) To see why, let's first look more carefully at what the Standard is trying to allow and disallow. 

Clearly, expressions like 
	a = b

and 	c = d + e

which read some values and use them to write others, are well-defined and legal. Clearly, [footnote] expressions like 	i = i++

which modify the same value twice are abominations which needn't be allowed (or in any case, needn't be well-defined, i.e. we don't have to figure out a way to say what they do, and compilers don't have to support them). Expressions like these are disallowed by the first sentence. 

It's also clear [footnote] that we'd like to disallow expressions like 
	a[i] = i++

which modify i and use it along the way, but not disallow expressions like 	i = i + 1

which use and modify i but only modify it later when it's reasonably easy to ensure that the final store of the final value (into i, in this case) doesn't interfere with the earlier accesses. 

And that's what the second sentence says: if an object is written to within a full expression, any and all accesses to it within the same expression must be directly involved in the computation of the value to be written. This rule effectively constrains legal expressions to those in which the accesses demonstrably precede the modification. For example, the old standby i = i + 1 is allowed, because the access of i is used to determine i's final value. The example 
	a[i] = i++

is disallowed because one of the accesses of i (the one in a[i]) has nothing to do with the value which ends up being stored in i (which happens over in i++), and so there's no good way to define--either for our understanding or the compiler's--whether the access should take place before or after the incremented value is stored. Since there's no good way to define it, the Standard declares that it is undefined, and that portable programs simply must not use such constructs. 