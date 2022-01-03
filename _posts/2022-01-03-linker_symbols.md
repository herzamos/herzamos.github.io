---
layout: post
title:  "C linker symbols"
date:   2022-01-03 16:00:00 +0200
categories:  C system-programming
tags: C system-programming
---

Disclaimer: this post is a copy of an answer found on the VIS Community Solution platform. The answer was so well written that I felt it was a shame not to share it with as many people as possible. You can find the original answer [here](https://exams.vis.ethz.ch/exams/hw3wnogr.pdf#yvryiskgee1q7btg), full credits are given to Theo Weidmann ([weidmann@ethz.ch](mailto:tweidmann@ethz.ch)).

## Origin
This answer originated from a discussion between students. The discussion was caused by a question which read as follows:
> For each listed identifier in the following C object file, say whether it is a strong linker symbol, a weak linker symbol, a macro, a symbol local to the compilation unit, or none of these.
The C program contained, together wiht many other stuff, the declaration (but not definition) of a function, the declaration of a `extern char *` and the definition of a `struct`. All three of those were said not to belong to any of the above possibilities, and this generated a lot of confusion.

## What is a symbol?

A symbol is a name for a memory address within assembly code.

Let's look at an example, `test.s`. In the below code doSomething and `theCount` and `doAnotherThing` are symbols:
```
  .globl  theCount
  .bss
  .size   theCount, 4
theCount:

  .zero   4
  .text
  .globl  doSomething
doSomething:
  pushq   %rbp
  movq    %rsp, %rbp
  movl    $0, %eax
  call    doAnotherThing
  popq    %rbp
  ret
```
Note that ``theCount`` and ``doSomething`` appear as labels, i. e. they appear at the beginning of a line followed by a colon, which defines the symbol. By contrast, ``doAnotherThing`` only appears as an instruction operand. We call a symbol that appears as an operand a reference.

So ``test.s`` does define ``theCount`` and ``doSomething`` but it does not define ``doAnotherThing``.

Let's assemble our test file into an object file:
```
gcc test.s -c
```
The assembler creates a symbol table as part of the object file, which is a list of all symbols it encountered. With each symbol, it notes whether the file defined or only referenced the symbol. We can use ``nm`` to look at the generated symbol table:
```
                 U doAnotherThing
0000000000000000 T doSomething
0000000000000000 B theCount
```
``B`` and ``T`` mean that these symbols were defined in the ``bss`` and ``text`` section, respectively. The ``U`` before ``doAnotherThing`` means that the symbol is not defined but was referenced.

## What are symbols and the symbol table good for?

Linking allows us to split up code. It allows us to define ``doAnotherThing`` in one file and use it from another file.

So let's define ``doAnotherThing`` in ``anotherThing.s``:
```
  .globl  doAnotherThing
doAnotherThing:
  pushq   %rbp
  movq    %rsp, %rbp
  popq    %rbp
  ret
```
If assembled, the symbol table looks like this:
```
0000000000000000 T doAnotherThing
```
So far, we have only created individual object files. To create a binary from these object files, we need to involve the linker.

The linker reads all symbol tables and ensures that for every reference to a symbol, there is a definition of that symbol. In practice, this means checking that all symbols with a ``U`` entry in one symbol table have a defining entry in another table from another file.

The actual step of replacing symbol names with addresses has been covered in exercises.

## What about strong and weak?

Everything seems good so far. We can use symbols in one file and provide a definition in another file. But it turns out there are cases where we want to define the same symbol in multiple files.

Take, for example, this (purely hypothetical) situation:

We've built a huge library of very complex operations. Since most applications only ever use one of these complex operations, every operation (which consists of many functions) is provided as a single object file. Some of these operations need a helper function ``performCalculation``. So we define it in the object file of each operation that needs it. But this leads to an interesting problem when someone links two object files that both contain ``performCalculation`` into their program. Which of the two definitions should the linker use?

If we allowed any symbol to be defined more than once, this could have disastrous effects. What if we name two things the same by accident? So by default, the linker will only allow one definition of every symbol. We call symbols that can only be defined one time **strong**. If a strong symbol is defined multiple times, the linker will give an error.

Still, we want to solve our problem. We do so by introducing an option to allow multiple definitions of the same symbol. We call these symbols **weak**. To fix our issue, we make ``performCalculation`` weak in all our files (note the first line):
```
  .weak   performCalculation
performCalculation:
  pushq   %rbp
  movq    %rsp, %rbp
  movl    %edi, -20(%rbp)
  movl    %esi, -24(%rbp)
```
If there are multiple ``performCalculation`` definitions at link-time, the linker will choose any of these. Since all implementations of ``performCalculation`` are the same, this will work out fine. Problem solved.

The relation between strong and weak was thoroughly covered in the lecture ETH "System Programming and Computer Architecture" course, 251-0061-00L.

## At The Level of C

So what does all this mean at the level of C?

First of all, it means that we can only define symbols that were actually defined in C code. If we have
```C
 struct element *pull();
```
we cannot meaningfully define this symbol. We don't even know what this function does!

The keyword extern means exactly the same thing:
```C
extern char *otherstring;
```
It tells the compiler that there is a variable otherstring in one of the other translation units (resulting object files). But this is not the definition. So again, it would not make any sense to emit a symbol definition.

These declarations are only necessary to allow the C compiler to do type checking. (As a matter of fact, early versions of C behaved much like assembly and allowed you to call undeclared, unknown functions...)

This translation unit defines push, head and tail, so they will also result in a symbol definition.

The symbol otherstring will not be referenced in the resulting object file, because we did not use it. If we used it, it would appear as an undefined reference in the symbol table.

Note, that the compiler must always emit definitions as it cannot know which symbols other object files might reference. (The exception being static, of course.)

## Final Remarks

But why are uninitialized globals weak? I would guess for historic reasons. The C standard actually prohibits defining variables more than once.

And how to define a weak function in C? You cannot. But other compilers for other languages do emit weak functions.

Where are you taking this from? Weak and strong were not made up by the lecture. It's widely used. See for example, LLVM, the industry-leading tool for building compilers: [https://llvm.org/docs/LangRef.html#linkage-types](https://llvm.org/docs/LangRef.html#linkage-types). (weak and strong is nonetheless an abstraction, the actual Linux implementation is more complicated.)