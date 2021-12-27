---
layout: post
title:  "Deciphering C declaration swith the right-left rule"
date:   2021-12-27 15:30:00 +0200
categories:  C system-programming
tags: C system-programming
---

Disclaimer: this post is an adaptation of [this](https://cseweb.ucsd.edu//~ricko/rt_lt.rule.html) blog post, which I felt was not polished enough.

## Introduction
Have you ever looked at a C declaration, something like `int *x(int *, int)[]` and wondered "Well, what the hell is that?" before proceeding to get an headache trying to understand how to read it? If the answer is yes, then we are in the same boat! Or at least we were, until I discovered this simple rule which makes reading C declarations as easy as it can get.

## The Rule
For the following, keep this table in mind:

| Symbol(s) | To be read as |
| `*` | "pointer to" |
| `[]` | "array of" |
| `()` | "function returning" |

The process is really simple, and it works as follow: 
1. Find the identifier.  This is your starting point.  Then say to yourself, "identifier is."  You've started your declaration.
2. Look at the symbols on the right of the identifier.  If, say, you find `()` there, then you know that this is the declaration for a function.  So you would then have "identifier is function returning".  Or if you found a  `[]` there, you would say "identifier is array of".  Continue right until you run out of symbols *OR* hit a *right* parenthesis `)`.  (If you hit a  left parenthesis, that's the beginning of a `()` symbol, even if there is stuff in between the parentheses.  More on that below.)
3. Look at the symbols to the left of the identifier.  If it is not one of our symbols above (say, something like "int"), just say it.  Otherwise, translate it into English using that table above.  Keep going left until you run out of symbols *OR* hit a *left* parenthesis `(`.  

Now repeat steps 2 and 3 until you've formed your declaration.

## Some Examples
(Taken from the autumun semester exam of the "System Programming and Computer Architecture" course at ETH, 251-0061-00L)
Say you want to decipher this not so simple C declaration: 
```c
int **x[10]
```
Start by finding the identifier
```c
int **x[10]
      ^
```
The declaration reads now as "x is.". Proceed with step 2 and we get to the end of the line.
```c
int **x[10]
       ^^^^
```
The declaration is now "x is an array 10 of.".
Proceed with step 3 and we run out of symbols:
```c
int **x[10]
^^^^^^
```
Our declaration is finally "x is an array 10 of pointer to pointer to int.".
The solution was indeed “declare x as array 10 of pointer to pointer to int”.

Say you now want to decipher the following declaration:
```c
int *x(int *, int)
```
Start with steps 1 and 2:
```c
int *x(int *, int)
     ^^^^^^^^^^^^^
```
The declaration is, up until now, "x is a function (pointer to int, int) returning."
Now step 3:
```c
int *x(int *, int)
^^^^^
```
We obtain "x is a function (pointer to int, int) returning pointer to int."
