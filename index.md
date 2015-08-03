---
title: oitofelix - QDot 8086
description: >
  QDot 8086 is a mid-level programming language targeting the original
  IBM-PC architecture written as a set of macros for NASM --- the
  Netwide Assembler.
tags: >
  QDot, 8086 Assembly, NASM, IBM-PC
license: CC BY-SA 4.0
layout: oitofelix-homepage
base: http://oitofelix.github.io
#base_local: http://localhost:4000
---
<div id="markdown" markdown="1">
## QDot 8086

__QDot 8086__ is a mid-level programming language targeting the
original IBM-PC architecture written as a set of macros for
[NASM](http://www.nasm.us/) --- _the Netwide Assembler_.  The idea
behind it is to make it easy to write small, fast, correct and
maintainable code in a language almost as expressive as _C_ but
without giving up all control Assembly language grants to programmers.
It features support to functions of an arbitrary number of parameters
and multiple return values, global and function-local variables, loop
and conditional flow-control constructs, evaluation of arbitrarily
complex stack-based expressions, symbol importing and primitive
debugging.  In order to accomplish this, NASM's powerful preprocessing
and assembling capabilities are used to achieve a machinery that very
closely resembles a compiler.  _QDot_ has also a companion standard
library that is fully BIOS-based, thus OS-independent, which provides
array processing, keyboard, video, disk and speaker I/O, timing,
low-level debugging, math functions, user interface procedures and
last but not least a versatile metamorphic boot-loader, that makes it
simple to build a binary that is simultaneously a valid DOS executable
and a bootable image --- a property known as
_run-within-OS-or-bootstrap-itself-without-OS_.  There are already a
couple of programs implemented in _QDot_ as a proof of concept:
[Terminal Matrix 8086](/terminal-matrix-8086) and
[DeciMatrix](/decimatrix-8086).  _QDot_ currently supports only the
tiny memory model (`.COM` binaries --- whose code, data and stack fit
all within 64kb segment boundaries).


### Download

_QDot_ is free software and you can obtain its source code here.

- [QDot 8086 0.1 source code]({{ page.base_local }}{{ site.baseurl }}/qdot-8086-0.1.tar.gz)
- [VCS repository](http://github.com/oitofelix/qdot-8086/)


### Documentation

__Table of contents__

0. [Use]({{page.base_local}}{{site.baseurl}}/#use)
0. [Numerical constants]({{page.base_local}}{{site.baseurl}}/#numerical-constants)
0. [Variables]({{page.base_local}}{{site.baseurl}}/#variables)
0. [Operators]({{page.base_local}}{{site.baseurl}}/#operators)
0. [Expressions]({{page.base_local}}{{site.baseurl}}/#expressions)
0. [Flow-control]({{page.base_local}}{{site.baseurl}}/#flow-control)
0. [Functions]({{page.base_local}}{{site.baseurl}}/#functions)
0. [Symbol importing]({{page.base_local}}{{site.baseurl}}/#symbol-importing)
0. [Debugging]({{page.base_local}}{{site.baseurl}}/#debugging)
0. [Standard library]({{page.base_local}}{{site.baseurl}}/#standard-library)
0. [kernel/memory.qdt]({{page.base_local}}{{site.baseurl}}/#kernelmemoryqdt)
0. [kernel/video.qdt]({{page.base_local}}{{site.baseurl}}/#kernelvideoqdt)
0. [kernel/keyboard.qdt]({{page.base_local}}{{site.baseurl}}/#kernelkeyboardqdt)
0. [kernel/timer.qdt]({{page.base_local}}{{site.baseurl}}/#kerneltimerqdt)
0. [kernel/speaker.qdt]({{page.base_local}}{{site.baseurl}}/#kernelspeakerqdt)
0. [kernel/disk.qdt]({{page.base_local}}{{site.baseurl}}/#kerneldiskqdt)
0. [kernel/boot.qdt]({{page.base_local}}{{site.baseurl}}/#kernelbootqdt)
0. [kernel/debug.qdt]({{page.base_local}}{{site.baseurl}}/#kerneldebugqdt)
0. [math/random.qdt]({{page.base_local}}{{site.baseurl}}/#mathrandomqdt)
0. [os/dos.qdt]({{page.base_local}}{{site.baseurl}}/#osdosqdt)


### Use

In order to use _QDot_ just include it in your main source code file,
like this:

<pre>
%include "qdot/qdot.qdt"
</pre>

Make sure to put _QDot_'s `lib` directory in NASM's include search
path, for instance, by using the `-i` switch.  When compiling your
code, use the plain binary output format by means of the `-fbin`
option.  A typical command-line invocation for compilation of _QDot_
sources looks like:

<pre>
nasm -isrc -isrc/lib/ -fbin -oprog.com src/prog.qdt
</pre>

Where all source code is inside the `src` directory, including
_QDot_'s main `lib` directory, and presuming the main source code is
called `prog.qdt`.  Obviously, every source code file written in
_QDot_ language is also a valid _NASM_ file, provided you include all
of _QDot_'s macro definitions, but by convention, we use the `.QDT`
extension for _QDot_ source code.

_QDot_ is dubbed this way because of the ubiquitous `?` character used
as the first component in the name of all _NASM_ macros defined for
_QDot_.  That character, somewhat unusual for identifier names, makes
it easy to distinguish between code using _QDot_'s facilities and
standard, or third-party, _NASM_ code.


### Numerical constants

Being _QDot_ implemented on top of _NASM_, its syntax and semantics
for numerical constants is the same.  You can read more about it at
[NASM's numerical constants manual section](http://www.nasm.us/xdoc/2.11.08/html/nasmdoc3.html#section-3.4.1).
Every numeric constant in a _QDot 8086_ context is a 16-bit number.
Examples of numerical constants are: `61640`, `0F0C8h` and
`1111000011001000b` --- all of them representing the same number in
decimal, hexadecimal and binary, respectively.  _QDot_ equates the
symbols `?TRUE` and `?FALSE` to numerical constants that represent the
respective boolean values.  Similarly, the `?NULL` symbol equates to a
numerical constant that represents the null pointer.

See the file `lib/qdot/stack.qdt` for the implementation details of
predefined numerical constants.


### Variables

All variables in _QDot 8086_ are 16-bit wide.  To perform byte-wide
operations one has to resort to the couple of special operators:
`@byte` and `@byte=` for reading and writing, respectively.  There are
two scopes for variables _global_ and _function-local_.  The former
are the usual _NASM_ labels pointing to data space and the latter are
[context local labels](http://www.nasm.us/xdoc/2.11.08/html/nasmdoc4.html#section-4.7.2)
defined at the function level.  Therefore, local variable names always
start with `%$` at the top-most level inside a _QDot_ function, and
deeper references must add a `$` character per nesting level, since
unfortunately
[context fall-through lookup](http://www.nasm.us/xdoc/2.11.08/html/nasmdoc4.html#section-4.7.4)
has been removed from _NASM_.  _QDot_ pushes a new context into _NASM_
preprocessor's context stack for each nested flow-control construct.
Examples of global and function-local variable references are
`[video_rows]` and `%$count`, respectively.  The latter would be
referred as `%$$count`, though, if one context deeper --- and so on.


### Operators

An operator is a symbol used in stack-based expressions to transform
the stack and/or variable's values.  _QDot_ has operators analogous to
almost every _C_ language operator and a few more.  Suppose that `a`,
`b` and `c` refer to the three stack's top-most values, in this order
from top to bottom, `var` refers to a global or local variable, and
`id` to a function symbol.  The operators work as follow:

- __Arithmetic__
  - __`+`__: pop `a` and `b`, then push their sum;
  - __`-`__: pop `a` and `b`, then push the difference between `a` and `b`;
  - __`*`__: pop `a` and `b`, then push their product;
  - __`/`__: pop `a` and `b`, then push the quotient of `a` divided by `b`;
  - __`%`__: pop `a` and `b`, then push the remainder of `a` divided by `b`;
  - __`min`__: pop `a` and `b`, then push the lowest of both, or one
    of them if they are equal (unsigned comparison);
  - __`min-`__: pop `a` and `b`, then push the lowest of both, or one
    of them if they are equal (signed comparison);
  - __`max`__: pop `a` and `b`, then push the lowest of both, or one
    of them if they are equal (unsigned comparison);
  - __`max-`__: pop `a` and `b`, then push the lowest of both, or one
    of them if they are equal (signed comparison);
  - __`neg`__: pop `a`, then push its symmetrical additive;
  - __`inc`__: pop `a`, then push its successor;
  - __`dec`__: pop `a`, then push its predecessor;
  - __`abs`__: pop `a`, then push its absolute value;

- __Bitwise__
  - __`~`__: pop `a`, then push its bitwise negation;
  - __`&`__: pop `a` and `b`, then push their bitwise conjunction;
  - __`|`__: pop `a` and `b`, then push their bitwise disjunction;
  - __`^`__: pop `a` and `b`, then push their bitwise exclusive disjunction;
  - __`<<`__: pop `a` and `b`, then push `a` bitwise-left-shifted by
    `b` positions;
  - __`>>`__: pop `a` and `b`, then push `a` bitwise-right-shifted by
    `b` positions;

- __Boolean__
  - __`!`__: pop `a`, then push its boolean negation;
  - __`&&`__: pop `a` and `b`, then push their boolean conjunction;
  - __`||`__: pop `a` and `b`, then push their boolean disjunction;

- __Comparison__
  - __`==`__: pop `a` and `b`, then push `?TRUE` if they are equal,
    `?FALSE` otherwise;
  - __`!=`__: pop `a` and `b`, then push `?TRUE` if they are
    different, `?FALSE` otherwise;
  - __`<`__: pop `a` and `b`, then push `?TRUE` if `a` is less than
    `b`, `?FALSE` otherwise (unsigned comparison);
  - __`<-`__: pop `a` and `b`, then push `?TRUE` if `a` is less than
    `b`, `?FALSE` otherwise (signed comparison);
  - __`<=`__: pop `a` and `b`, then push `?TRUE` if `a` is less than
    or equal to `b`, `?FALSE` otherwise (unsigned comparison);
  - __`<=-`__: pop `a` and `b`, then push `?TRUE` if `a` is less than
    or equal to `b`, `?FALSE` otherwise (signed comparison);
  - __`>`__: pop `a` and `b`, then push `?TRUE` if `a` is greater
    than `b`, `?FALSE` otherwise (unsigned comparison);
  - __`>-`__: pop `a` and `b`, then push `?TRUE` if `a` is greater
    than `b`, `?FALSE` otherwise (signed comparison);
  - __`>=`__: pop `a` and `b`, then push `?TRUE` if `a` is greater
    than or equal to `b`, `?FALSE` otherwise (unsigned comparison);
  - __`>=-`__: pop `a` and `b`, then push `?TRUE` if `a` is greater
    than or equal to `b`, `?FALSE` otherwise (signed comparison);

- __Assignment__
  - __`=, var`__: pop `a`, then assign it to `var`;
  - __`=$, var`__: assign `a` to `var`;
  - __`+=, var`__: pop `a`, then sum it to `var`;
  - __`-=, var`__: pop `a`, then subtract it from `var`;
  - __`*=, var`__: pop `a`, then store in `var` the product of
    both;
  - __`/=, var`__: pop `a`, then store in `var` the quotient of the
    division of `var` by `a`;
  - __`%=, var`__: pop `a`, then store in `var` the remainder of
    the division of `var` by `a`;
  - __`&=, var`__: pop `a`, then store in `var` the bitwise
    conjunction of both;
  - __`|=, var`__: pop `a`, then store in `var` the bitwise
    disjunction of both;
  - __`^=, var`__: pop `a`, then store in `var` the bitwise
    exclusive disjunction of both;
  - __`<<=, var`__: pop `a`, then store in `var` itself
    bitwise-left-shifted by `a` positions;
  - __`>>=, var`__: pop `a`, then store in `var` itself
    bitwise-right-shifted by `a` positions;
  - __`++, var`__: increment `var` by one;
  - __`++$, var`__: increment `var` by one, then push it;
  - __`$++, var`__: push `var`, then increment it by one;
  - __`--, var`__: decrement `var` by one;
  - __`--$, var`__: decrement `var` by one, then push it;
  - __`$--, var`__: push `var`, then decrement it by one;

- __Dereference__
  - __`@word`__: pop `a`, then push the word value it points to;
  - __`@word=`__: pop `a` and `b`, then assign `b` to the word `a`
    points to;
  - __`@byte`__: pop `a`, then push the word extension of the byte it
    points to;
  - __`@byte=`__: pop `a` and `b`, then assign the low byte of `b` to
    the byte `a` points to;

- __Function call__
  - __`call, id`__: call function whose symbol is `id`;
  - __`dcall`__: pop `a`, then call the function `a` points to;

- __Flow:__
  - __`?:`__: pop `a`, `b` and `c`, then push `b` if `a` is
    `?TRUE`, otherwise push `c`;

- __Stack:__
  - __`drop`__: pop `a`;
  - __`dup`__: push `a`;


See the file `lib/qdot/stack.qdt` for the implementation details of
_QDot_'s operators.


### Expressions

_QDot_'s core concept is that of stack-based _expression_.  An
expression is a comma-separated list of numerical constants,
variables, and operators.  They can be used by themselves in order to
achieve side-effects, or as conditions in flow-control constructs or
as calculations of return values.  The elements of the list are read
by the compiler from right to left and each of them change the stack,
and potentially variables, respecting that order; therefore, it's said
that expressions in _QDot_ have "Polish", or "prefix", notation.
Numerical constants are always pushed to the stack.  Variables have
their values pushed onto the stack, unless they lie before an
assignment operator, in which case the top-most stack's value is
popped, possibly operated on, and stored into the respective variable.
Operators pop stack values, possibly pushing results onto there or
into variables.  An example of a stand-alone expression that is
equivalent to the _C_ expression `x += (x - a++) * x + 3` is:

<pre>
? +=, %$x, +, *, -, %$x, $++, %$a, %$x, 3
</pre>

Although _QDot_ syntax and semantics would allow for an expression to
push values onto the stack in order to save them for later use, this
is considered bad practice and can potentially break _QDot_'s
implementation in unexpected ways.  Therefore, it's given as formally
undefined the behavior of programs that make use of this kind of
misleading and perverse hacks.  The only exception being, of course,
the return expressions of a function, that may evaluate to multiple
values that are pushed onto stack prior returning, but that is done
transparently by means of the `?return` and `?endfunc` clauses.  The
rule of thumb for this matter is: stack-based expressions must be
designed to be clean and atomic in the sense that they must leave the
stack in the state they got it initially --- the stack is a medium of
operation within an expression and __NOT__ across expressions ---
unless the expression is a flow-control condition, where a boolean
value is expected, in which case it must evaluate to that, and only
that, value, otherwise it must evaluate to nothing.  For instance, all
stand-alone expressions (those with just a `?` after them), must
evaluate to nothing.  This rule is extended to low-level `push` and
`pop` instructions: pop everything you push.

The current implementation of _QDot 8086_ reserves all processor's
general-purpose registers `ax`, `bx`, `cx` and `dx`, to perform
operations within expressions.  Therefore, you should not presume any
of them remains unchanged after the evaluation of an expression, be it
stand-alone, flow-control or return based.

See the file `lib/qdot/stack.qdt` for the implementation details of
stack-based expressions.


### Flow-control

There is a nearly one-to-one match between _QDot_'s loop and
conditional flow-control constructs and those of the _C_ language.
_QDot_ supports two conditional constructs, `?if` and `?switch`, and
three loop constructs, `?do`, `?while` and `?for`.


The syntax for the `?if` block is:

<pre>
?if condition_0
  ...
?elseif condition_1
  .
  .
  .
?elseif condition_n
  ...
?else
  ...
?endif
</pre>

Where `condition_0`...`condition_n` are mandatory expressions that
must evaluate to a boolean value.  Each of
`condition_0`...`condition_n` are evaluated in sequence until one of
them evaluates to `?TRUE`, in which case its correspondent sub-block is
executed.  If no expression evaluates to `?TRUE`, the `?else` sub-block
is executed, in case there is one, otherwise no code is executed.  The
`?elseif` or `?else` clauses may be omitted.


The syntax for the `?switch` block is:

<pre>
?switch [value]
  ?case condition_0
    .
    .
    .
  ?case condition_n
    ...
  ?default
    ...
?endswitch
</pre>

Where `[value]` is an optional expression that must evaluate to
nothing and `condition_0`...`condition_n` are mandatory expressions
that must evaluate to a boolean value.  The `[value]` expression is
evaluated, if given, then each of `condition_0`...`condition_n` are
evaluated in sequence until one of them evaluates to `?TRUE`, in which
case its correspondent sub-block is executed.  If no expression
evaluates to `?TRUE`, the `?default` sub-block is executed, in case
there is one, otherwise no code is executed.  There is no need for
some form of break statement as in _C_ because there is no
fall-through.  As you can see, this is essentially an
`?if/?elseif/?else/?endif` block with a prior standalone expression
used to initialize a variable, for example.  The `?default` clause may
be omitted.



There are two special clauses that can be used within loop blocks to
set execution to the next iteration or to resume execution at the
first line of code following the block: `?continue` and `?break`,
respectively.  As loop blocks can be nested, those clauses receive the
loop's name they apply to as a mandatory argument.  The loop's name is
given by a label preceding its opening clause, and may be omitted in
case there is no `?continue` nor `?break` clause referring to it.


The syntax for the `?do` block is:

<pre>
label: ?do
  ...
?while condition
</pre>

Where `condition` is a mandatory expression that must evaluate to a
boolean value.  The code within the block is executed and then
`condition` is evaluated.  If it evaluates to `?TRUE`, this process
repeats, otherwise the execution resumes at the first line of code
following the block.


The syntax for the `?while` block is:

<pre>
label: ?while condition
  ...
?endwhile
</pre>

Where `condition` is a mandatory expression that must evaluate to a
boolean value.  The `condition` expression is evaluated.  In case it
evaluates to `?TRUE` the code within the block is executed and this
process repeat, otherwise the execution resumes at the very first line
of code following the block.


The syntax for the `?for` block is:

<pre>
label: ?for [init]
  ?cond condition
  ...
?next [next]
</pre>

Where `[init]` and `[next]` are optional expressions that must
evaluate to nothing and `condition` is a mandatory expression that
must evaluate to a boolean value.  The `[init]` expression is
evaluated exactly once as the very first step.  Then, `condition` is
evaluated.  In case it evaluates to `?TRUE` the code within the block
is executed, and then `[next]` is evaluated and this process repeats,
otherwise the execution resumes at the very first line of code
following the block.


See the file `lib/qdot/flow.qdt` for the implementation details of
flow-control constructs.


### Functions

Functions are defined using the following syntax:

<pre>
?func funcname, %$par_0,...,%$par_n
  ?local %$var_0,...,%$var_n
  ...
  ?return [retvals_0]
  ...
  ?return [retvals_1]
  ...
?endfunc [retvals_n]
</pre>

Where,

- `funcname` is a valid identifier to be used as the function global
  name;
- `%$par_0`,...,`%$par_n` are the function parameter variables;
- `%$var_0`,...,`%$var_n` are the function local variables;
- `[retvals_0]`,...,`[retvals_n]` are the optional expressions for
  return values;

For example, a non-recursive function to calculate the factorial of a
number can be defined like this:

<pre>
?func factorial, %$n
  ?local %$p, %$i
  ?for =, %$$p, 1, =, %$$i, 2
    ?cond <=, %$$i, %$$n
    ? *=, %$$p, %$$i
  ?next ++, %$$i
?endfunc %$p
</pre>

For instance, that function can be called using the `call` operator
and have its return value incremented by one and collected in the
caller's variable `%$f`, like this:

<pre>
? =, %$f, inc, call, factorial, 8
</pre>

Although correct, this example is just illustrative, because this
function wouldn't allow you to calculate factorials of numbers above
8, because _QDot 8086_'s variables are only 16-bit wide.

The order in which arguments are written in calling expressions is the
natural order one familiar with C would expect: first argument first,
last argument last.  Since the compiler parses the expression from
right to left, the last argument of a function call is deeper on the
stack than the first argument, that lies on top.  The same rule
applies to return expressions: first return value on top, last return
value deeper on the stack.

You can return an arbitrary number of values by pushing them onto the
stack using the `?return` and `?endfunc` clauses.  By the way, this is
the only acceptable context where an expression should ultimately
evaluate to multiple values.  A single function can have a different
number of return values depending upon arbitrary conditions, however
the caller must be prepared to pop every single value returned,
therefore every possible combination of the number of return values
and conditions applicable must be known and paid attention to at
compilation time.

The number of arguments in a call to a function must match its number
of parameters.  You can, however, refer to "additional arguments"
pushed to stack prior the call by using the `?argument` macro, like
this: `?argument(i)`, where `i` is a number greater than the index of
the last parameter.  Thus, if a function has 2 parameters, the
expression `?argument(2)` points to the first "additional parameter".
As one would expect, those parameters aren't popped out in the process
of calling and returning from a function and must then be explicitly
dealt with by the caller.

The current implementation of _QDot 8086_ uses the `bp` register to
keep track of function arguments, local variables and the return
address.  Therefore, it's imperative that this register remain
untouched by user code.  At a return point (`?return` or `?endfunc`
clauses), the `sp` register is used to keep track of return values,
and therefore must be the same it were at the function's beginning.

See the file `lib/qdot/func.qdt` for the implementation details of
function definition, calling and returning.


### Symbol importing

A "module" is defined as "a file that provides importable symbols".
With a little help from _NASM_'s single-line macro definitions and
preprocessor includes, in _QDot_ you can define importable symbols
that relies on dependency trees of arbitrary complexity and depth.
The following idiomatic syntax is used to define an importable symbol:

<pre>
%ifdef IMPORT_symbol
%ifndef IMPORTED_symbol
%define IMPORTED_symbol

?import symboldep_0_0,...,symboldep_0_l
%include "file_0"
.
.
.
?import symboldep_n_0,...,symboldep_n_m
%include "file_n"

symbol_definition

%endif ; IMPORTED_symbol
%endif ; IMPORT_symbol
</pre>

Where,

- `symbol` is the symbol being defined;
- `symboldep_i_0`,...,`symboldep_i_j` are the symbols, defined in the
   file `file_i`, on which `symbol` depends upon: namely the symbols
   that its definition makes reference to;
- `symbol_definition` is the definition of `symbol` itself properly.
  It may be anything that defines `symbol` globally, ranging from
  _NASM_'s equates, or labels used with pseudo-instructions, like
  `db`, to _QDot_'s function definitions;

To illustrate this, let's re-define the factorial function to return
`0` in case of overflow.  First we declare an importable symbol
defining the maximum allowable value for the factorial's argument:

<pre>
%ifdef IMPORT_FACTORIAL_ARGMAX
%ifndef IMPORTED_FACTORIAL_ARGMAX
%define IMPORTED_FACTORIAL_ARGMAX

FACTORIAL_ARGMAX equ 8

%endif ; IMPORTED_FACTORIAL_ARGMAX
%endif ; IMPORT_FACTORIAL_ARGMAX
</pre>

Then we re-define the factorial function as an importable symbol
depending upon the previously defined `FACTORIAL_ARGMAX` symbol:

<pre>
%ifdef IMPORT_factorial
%ifndef IMPORTED_factorial
%define IMPORTED_factorial

?import FACTORIAL_ARGMAX
%include "math/factorial.qdt"

?func factorial, %$n
  ?local %$p, %$i
  ?if >, %$$n, FACTORIAL_ARGMAX
    ?return 0
  ?endif
  ?for =, %$$p, 1, =, %$$i, 2
    ?cond <=, %$$i, %$$n
    ? *=, %$$p, %$$i
  ?next ++, %$$i
?endfunc %$p

%endif ; IMPORTED_factorial
%endif ; IMPORT_factorial
</pre>

And finally we could import and use it like this:

<pre>
?import factorial
%include "math/factorial.qdt"
...
  ? =, %$f, inc, call, factorial, 9     ; now '%$f' equals 1
...
</pre>


Notice that, one can in principle define more than one symbol inside a
single importable symbol block.  That feature should be used wisely,
though, in order to maintain module's sanity and cleanness.  A
legitimate use for it is to define multiple symbols that comprise a
single whole, like a 32 bit variable, with one symbol for the high and
other for the low word, like this:

<pre>
;;;;;;;;;;;;;;
; random_seed
;;;;;;;;;;;;;;

%ifdef IMPORT_random_seed
%ifndef IMPORTED_random_seed
%define IMPORTED_random_seed

random_seed_0 dw 0000h
random_seed_1 dw 0000h

%endif ; IMPORTED_random_seed
%endif ; IMPORT_random_seed
</pre>

Notice also, that a module can depend upon symbols defined by itself
--- and that's actually pretty common.  The recursive `%include`
preprocessor directives should not pose a problem for two reasons:
first, an importable symbol must never depend on itself, and second,
no piece of code inside a module, be it an importable symbol block or
otherwise, may be allowed to be included twice.

If the programmer wants the main source file of a program to be a
module, so it can depend on symbols defined by itself --- an elegant
choice --- he must make sure it obeys the two principles above.  For
that end, the execution entry point must be isolated from the rest of
the file and the remaining code must be made into importable symbol
blocks.  The very first code inside the main source module must follow
along the lines:

<pre>
%push SRC_PROG_QDT ; push a preprocessor context for this module

%ifndef SRC_PROG_QDT ; ensure this block won't be included twice
%define SRC_PROG_QDT

CPU 8086
ORG 100h

%include "qdot/qdot.qdt" ; enable QDot language

mov sp, 0FFFEh ; setup stack for QDot expressions
mov bp, sp

call main  ; call the main procedure defined in this very module
           ; somewhere outside this block

?import main  ; import the main symbol used above.  This
              ; will expand to all the code it depends upon, and so on
              ; recursively, thus all the program's source code will
			  ; be compiled and assembled here
%include "src/prog.qdt"

SECTION .bss ; here is the binary end
.
.
.
%endif ; SRC_PROG_QDT
.
. ; here are all importable symbol blocks defined by the
. ; main module
.
%pop SRC_TM_QDT ; the module ends here
</pre>


See the file `lib/qdot/func.qdt` for the implementation details of
symbol importing.


### Debugging

The `.COM` plain binaries don't support any kind of meta-information
since they are memory images in raw form.  Therefore, debugging
information is lacking from any executable built from source code
written in _QDot_.  In order to ease debugging, a practice of
paramount importance in software development of any scale, _QDot_
defines the `?debug` clause.  This construct aids in debugging by
inserting an arbitrary string, and a jump over it, in the assembled
binary which can be used to locate specific part of code at the place
of declaration without changing the executable behavior.  Thus, for
instance, one can find where the code for a function starts by putting
a `?debug` clause in its beginning, then compiling and telling the
debugger to search for the string passed to the `?debug` clause at
that point.

For example, if we would like to easily find, within a debug session,
the entry point for the `factorial` function we defined, the following
code put at the very line before the function definition does the job:

<pre>
?debug 'factorial'
</pre>

Then it suffices to search for the string `'factorial'` in the
debugger.  When defining several points of reference for debugging
make sure to use correlate but distinct strings, like:
`'factorial_0'`,...,`'factorial_n'` and so forth.

See the file `lib/qdot/func.qdt` for the implementation details of
debugging support.


### Standard library

Although the language is, in a strict sense, feature-complete and
should remain stable, the standard library is the result of the needs
I've come across while developing particular projects.  Therefore, one
should expect the standard library to be ever evolving according to
the use case scenarios I shall face in the future.  At this stage it
covers well some specific areas, not so well others and lacks a lot of
features one may expect for projects sufficiently different from the
ones I've developed with _QDot_ to the present date.

The standard library is designed to be deployed at source-level for
each project that depends upon it.  It means that a copy of it should
go along each project that makes use of it.  The library is modular
enough, though, so one may just include the relevant bits of it,
omitting the rest, without breaking anything.

In order to use a particular symbol defined by the library you should
import it as described in
"[Symbol importing]({{page.base_local}}{{site.baseurl}}/#symbol-importing)".
For example, if you want to use the function that clears the screen,
you must code

<pre>
?import video_cls
%include "kernel/video.qdt"
</pre>

within the importable symbol block you are defining for the function
that uses it, like this:

<pre>
;;;;;;;;;;
; cmd_cls
;;;;;;;;;;

%ifdef IMPORT_cmd_cls
%ifndef IMPORTED_cmd_cls
%define IMPORTED_cmd_cls

?import video_cls
%include "kernel/video.qdt"

?func cmd_cls, %$cmd_args
  ? call, video_cls
?endfunc

%endif ; IMPORTED_cmd_cls
%endif ; IMPORT_cmd_cls
</pre>

Inside the `lib` directory, every directory that is not `qdot`
comprises the standard library.  Each of them is associated with a
major area of application, and below them one can find the _QDot_
source code files, each corresponding to a specialization within its
respective area.  Currently the standard library comprehend the
following directories:

- __kernel__: I/O, memory handling, bootstrapping and debugging;
- __math__: Mathematical functions;
- __os__: OS-dependent routines;
- __ui__: High-level user interface procedures;

In order to avoid clashes in the global name space, by convention,
every function symbol has a prefix which corresponds to the file to
which it pertains.  For instance, the above `video_cls` function
pertains to the file `video.qdt`, that in turn is contained inside the
`kernel` directory.


### kernel/memory.qdt

This module could have been called `array.qdt`, but for the sake of
consistency with its hardware-oriented `kernel` fellows, it has been
dubbed `memory.qdt`.  This module defines several functions useful in
processing logical memory structures called "arrays", like characters
(dimension 0), strings (dimension 1) and higher-dimensional arrays.

The standard library defines string as a sequence of zero or more
characters terminated by `END`.  There is a macro specifically
employed to define strings:

- __`string`__: define an `END` terminated sequence of characters;

It's used like this:

<pre>
string 'This is a string!'
</pre>

Besides `END`, there are two more characters defined by the standard
library as having special meaning in strings: `LF` and `COLOR_ESC`.
The former terminates a row and the latter introduces an embedded text
attribute code.  Both are handled as expected by memory and video
routines.  Normally, one don't need to use them directly because there
are higher-level macros for the help (for `COLOR_ESC` see the `cor`
and `bcor` macros of `kernel/video.qdt`):

- __`row`__: define an `LF` terminated sequence of characters;

It's used like this:

<pre>
row 'This is a string!'
</pre>

However, that is not a complete string because it lacks the terminator
character.  In order to define a multi-row and complete string one
needs to use the array definition macros:

- __`array`__: open an array definition block;
- __`endarray`__: end an array definition block;

Use it like this:

<pre>
array
  row "This is the first row."
  row "This is the second row."
  row "This is the third row."
endarray
</pre>

That is an 1-dimensional array, that's to say, a string.  Moreover,
it's possible to define an array of strings, namely, a 2-dimensional
array:

<pre>
array
  string "This string is at index 0"
  string "This string is at index 1"
  string "This string is at index 2"
endarray
</pre>

Or even an array of 2-dimensional arrays --- a 3-dimensional array:

<pre>
array

  array
    string "This is an 1-dimensional element at 0,0"
    string "This is an 1-dimensional element at 0,1"
    string "This is an 1-dimensional element at 0,2"
  endarray

  array
    string "This is an 1-dimensional element at 1,0"
    string "This is an 1-dimensional element at 1,1"
    string "This is an 1-dimensional element at 1,2"
  endarray

  array
    string "This is an 1-dimensional element at 2,0"
    string "This is an 1-dimensional element at 2,1"
    string "This is an 1-dimensional element at 2,2"
  endarray

endarray
</pre>

And so on, recursively, to an arbitrarily higher dimensionality.  The
general definition given by the standard library for an array of
dimension _n_ is "a sequence of characters terminated by _n_
consecutive `END` characters".  With those definitions in mind we can
begin to explore the functions available for array processing.

The two general array functions that can work with arrays of arbitrary
dimensions are:

- __`%$len, memory_array_len, %$array, %$dim`__: supposing `%$array`
  has `%$dim` dimensions, assign the number of elements of dimension
  `%$dim` minus one contained in it to `%$len`;
- __`%$ptr, memory_array_elem, %$array, %$dim, %$i`__: supposing
  `%$array` has `%$dim` dimensions, assign the address of the element
  at index `%$i` to `%$ptr`;

Notice that, because of its very broad definition, there is no way for
a general array function to unambiguously identify the dimensionality
of an array only by inspecting its contents, since there is no hard
end to it.  That meta-information has to be stored externally and
provided in each function call.  Below are the restrict functions for
1-dimensional arrays (strings) and 0-dimensional arrays (characters).

A string has a property called "length", that is defined as "how far
the `END` character is placed from its beginning".  Intuitively,
that's how many characters there are in a string excepting the
terminator character.

- __`%$len, memory_strlen %$str`__: assign the length of the string
  `%$str` to `%$len`;

Another string property is called "width", that is roughly defined as
"the maximum number of characters between two adjacent `LF` characters
within a string".  Intuitively, that's how much horizontal space is
required to draw that string on screen.

- __`%$width, memory_strwidth, %$str`__: assign the width of the
  string `%$str` to `%$width`;

Similarly, there is yet another string property called "height", that
is roughly defined as "the number of `LF` characters within a string".
Intuitively, that's how much vertical space is required to draw that
string on screen.

- __`%$height, memory_strheight, %$str`__: assign the height of the
  string `%$str` to `%$height`;

To ensure a character is uppercase you can use the function:

- __`%$uchar, memory_uppercase_char, %$char`__: assign the uppercase
  correspondent of `%$char`, in case it's in the range _a--z_,
  otherwise `%$char` unchanged, to `%$uchar`;

To apply that procedure in-place to each character of a given string
one may use:

- __`memory_uppercase_str, %$str`__: convert all lowercase
  characters of the `%$str` string to uppercase;

Another string transformation function exists to obfuscate string
contents in a reversibly symmetric way.

- __`memory_rot47, %$str`__: rotate all printable ASCII characters
  (but space) from string `%$str` by 47 positions.  It's its own
  inverse, that is, applying it twice one obtains the original
  string;

One can also compare two strings to see if they are equal --- that is,
whether they have the same length and character sequence;

- __`%$eq, memory_streq, %$str0, %$st1`__: assign `?TRUE` to `%$eq` if
  string `%$str0` is equal to string `%$str1`, `?FALSE` otherwise;

It's also possible to copy one string to another memory location:

- __`memory_copy_str, %$dest, %$orig`__: copy the string at `%$orig`
  to the `%$dest` address;

Sometimes its useful to obtain indexes for specific characters within
strings.  That can be accomplished with the following function:

- __`%$i, memory_str_char_index, %$str, %$char`__: assign the index
  of the first occurrence of character `%$char` within string
  `%$str`, or `-1` in case there is none, to `%$i`;

On the other hand, one can find the first character that doesn't match
a given character for arbitrary memory locations.

- __`%$nptr, memory_skip_char, %$ptr, %$char`__: assign the address of
  the first character not equal to `%$char` starting at `%$ptr` to
  `%$nptr`.  Notice that this function doesn't care about string
  boundaries;

Related to this:

- __`%$count, memory_charseq_len, %$ptr, %$char`__: assign the number
  of consecutive occurrences of `%$char` at `%$ptr` to `%$count`;

See the file `lib/kernel/memory.qdt` for the implementation details of
kernel memory routines.


### kernel/video.qdt

This module is used to query and set some video properties, handle
screen cursor positioning and draw to the screen.  Before any video
operation can be done, the video subsystem has to be probed and
initialized by the following routine:

- __`video_init`__: probe and initialize the video subsystem.  This
  routine supports the holy trinity of `IBM-PC` graphics adapters:
  `CGA`, `EGA` and `VGA`, which it sets to the maximum resolution
  available: 25, 43 and 50 rows, respectively.

There are a dozen global variables used to get and set general video
properties that will regulate how video functions handle the display.
After calling `video_init` the following two couples of read-only
global variables are available for getting video resolution
information:

- __`video_rows`__: number of rows the screen currently shows;
- __`video_cols`__: number of columns the screen currently shows;
- __`video_maxrow`__: its value is `[video_rows]` minus one;
- __`video_maxcol`__: its value is `[video_cols]` minus one;

The following read/write global variables are initialized as well, but
to default values based upon the above variables.  However, they can
be changed at will in order to drive any applicable video function.

- __`video_page`__: video page in which operations take place.  It
  defaults to `0`;
- __`video_win_rows`__: number of rows of the current window.  It
  defaults to `[video_rows]`;
- __`video_win_cols`__: number of columns of the current window.  It
  defaults to `[video_cols]`;
- __`video_win_minrow`__: number of the top-most row of the current
  window.  It defaults to `0`;
- __`video_win_mincol`__: number of the left-most column of the
  current window.  It defaults to `0`;
- __`video_win_maxrow`__: number of the bottom-most row of the current
  window.  It defaults to `[video_maxrow]`;
- __`video_win_maxcol`__: number of the right-most column of the
  current window. I t defaults to `[video_maxcol]`;
- __`video_win_color`__: default text attribute for the current
  window.  It defaults to `color(LIGHT_GRAY,BLACK)`;

Often it's necessary to specify text attributes for output procedures.
The following equates define colors that can be used for specifying
foreground and background colors: __`BLACK`__, __`BLUE`__,
__`GREEN`__, __`CYAN`__, __`RED`__, __`MAGENTA`__, __`BROWN`__,
__`LIGHT_GRAY`__, __`DARK_GRAY`__, __`LIGHT_BLUE`__,
__`LIGHT_GREEN`__, __`LIGHT_CYAN`__, __`LIGHT_RED`__,
__`LIGHT_MAGENTA`__, __`YELLOW`__, __`WHITE`__.

A well-defined text attribute is specified by three parameters: the
foreground color, the background color, and the blinking text status.
The foreground and background colors are specified by the above
equates.  The following macros assist in making a complete text
attribute specification.

- __`color(fore,back)`__: expand to the text attribute that has `fore`
  as its foreground color, `back` as its background color, and
  non-blinking text;

- __`bcolor(fore,back)`__: expand to the text attribute that has
  `fore` as its foreground color, `back` as its background color, and
  blinking text;

Text attributes may also be embedded in strings so as to make it easy
to draw colored text messages or ASCII art by means of the
`video_draw_str` function.  The following macros are used in string
definitions.

- __`cor(fore,back)`__: expand to the string color escape code and
  text attribute that has `fore` as its foreground color, `back` as
  its background color, and non-blinking text;

- __`bcor(fore,back)`__: expand to the string color escape code and
  text attribute that has `fore` as its foreground color, `back` as
  its background color, and blinking text;

To enable blinking text a trade-off has to be made.  One has to give
up the following background colors: __`DARK_GRAY`__, __`LIGHT_BLUE`__,
__`LIGHT_GREEN`__, __`LIGHT_CYAN`__, __`LIGHT_RED`__,
__`LIGHT_MAGENTA`__, __`YELLOW`__, __`WHITE`__, because in that case
the intensity bit becomes the blinking bit.  When blinking is enabled,
__`DARK_GRAY`__ maps to __`BLACK`__, __`YELLOW`__ maps to __BROWN__,
__`WHITE`__ maps to __`LIGHT_GRAY`__, and every light color maps to
its non-light version.  In order to enable or disable blinking text
use the function `video_blink_mode` that gets a boolean, indicating
whether to enable blinking mode, as its solely parameter and returns
nothing.

Sometimes it's useful to draw to a video page that's not the current
one, for instance, to make an off-screen drawing, so the user don't
notice the flickering caused by direct drawing to a visible spot, or
to show transitory information without changing the current page's
content.  Every video function will work on the video page given by
the __`video_page`__ global variable, that user code may modify.  Of
course, that's not necessarily the current video page but one may make
it so by calling the following function:

- __`video_select_page, %$p`__: set `%$p` as the current video page
  and make it the operational page by storing it in `[video_page]`;

When drawing to screen, one may want to disable the screen cursor and
only enable it when reading input from keyboard.  For that end, there
are a couple of functions:

- __`video_disable_cursor`__: disable video cursor;
- __`video_enable_cursor`__: enable video cursor;

The video module has a rich set of cursor position related routines.
Those are divided in two classes: getting and setting.  In the getting
class one find functions that return cursor positions:

- __`%$r, video_row`__: assign the current cursor position row to
  `%$r`;
- __`%$c, video_col`__: assign the current cursor position column to
  `%$c`;
- __`%$r, %$c, video_pos`__: assign the current cursor position row to
  `%$r` and column to `%$c`;

Given a string it's also possible to calculate the starting row or
column at which it has to be drawn in order to appear centered on the
screen.

- __`%$r, video_cent_str_row, %$str`__: assign the starting row at
  which the string `%$str` has to be drawn in order to appear
  vertically centered on the screen to `%$r`;
- __`%$c, video_cent_str_col, %$str`__: assign the starting column at
  which the string `%$str` has to be drawn in order to appear
  horizontally centered on the screen to `%$c`;

In the setting class one can find a larger variety of procedures:

- __`video_setpos, %$r, %$c`__: set the cursor position to the row
  `%$r` and column `%$c`;
- __`video_setrow, %$r`__: set the cursor position to the row `%$r`
  and the current column;
- __`video_setcol, %$c`__: set the cursor position to the current row
  and the column `%$c`;
- __`video_setpos_rel, %$dr, %$dc`__: set the cursor position to the
  current row plus `%$dr` and the current column plus `%$dc`.  Both
  parameter may be negative integers;
- __`video_setrow_cent`__: set the cursor position to the row at the
  middle of the current window, and the current column;
- __`video_setcol_cent`__: set the cursor position to the current row,
  and the column at the middle of the current window;

Furthermore, there are two couples of routines that aid in
terminal-like output:

- __`video_next_col`__: if not at the current window's last column,
  advance cursor by one column.  Otherwise, if not at the current
  window's last row, advance cursor by one row.  Otherwise, scroll up
  the current window by one row and rewind the cursor to the current
  window's first column;
- __`video_prev_col`__: if not at the current window's first column,
  rewind cursor by one column.  Otherwise, if not at the current's
  window first row, rewind cursor by one row and advance cursor to the
  current window's last column.  Otherwise do nothing;
- __`video_next_row`__: if not at the current window's last row,
  advance cursor by one row.  Otherwise scroll up the current window
  by one row;
- __`video_prev_row`__: if not at the current window's first row,
  rewind cursor by one row.  Otherwise do nothing;

As you may have notice in these functions, the current window may be
scrolled.  That's the job of the following procedures:

- __`video_scroll_up_win, %$rows`__: scroll up the current window by
  `%$rows`;
- __`video_scroll_down_win, %$rows`__: scroll down the current window
  by `%$rows`;

Finally, it's possible to draw to the screen by using the following
routines:

- __`video_cls`__: clear the current window and put cursor at the
  minimum allowable position;
- __`video_draw_char, %$char, %$color, %$count`__: draw the character
  `%$char`, with text attribute `%$color`, `%$count` times starting at
  the current cursor position.
- __`video_draw_char_hfull, %$char, %$color, %$dy0, %$dy1`__: draw the
  character `%$char`, with text attribute `%$color`, filling from the
  first to the last column of the current row plus `%$dy0` to the
  current row plus `%$dy1`.  The parameters `%$dy0` and `%$dy1` may be
  negative;
- __`video_draw_str, %$str, %$color`__: draw the string `%$str`, with
  text attribute `%$color`, starting at the current cursor position.
  The `%$color` parameter may be overridden by color escape sequences
  (see `cor` and `bcor` macros) embedded within the string;
- __`video_draw_str_hcent, %$str, %$color`__: same as the
  `video_draw_str`, but centralize the string horizontally;
- __`video_draw_str_vhcent, %$str, %$color`__: same as the
  `video_draw_str`, but centralize the string horizontally and
  vertically;

Reciprocally, it's also possible to read characters that have been
drawn to the screen.

- __`%$char, %$color, video_read_char`__: assign the character at the
  current cursor position to `%$char` and its text attribute to
  `%$color`;

See the file `lib/kernel/video.qdt` for the implementation details of
kernel video routines.


### kernel/keyboard.qdt

This module handles keyboard input.  Prior to reading any input one
might want to setup the keyboard delay and rate of repetition.

- __`keyboard_mindelay_maxrate`__: set keyboard delay and rate of
  repetition to the minimum and maximum values allowed, respectively;

For reading from keyboard a character at a time, one can use these
routines:

- __`%$char, keyboard_read_char`__: in case the keyboard buffer is
  non-empty, remove its next character and assign that to `%$char`,
  otherwise wait for it to become non-empty and then repeat this
  process;
- __`%$char, keyboard_check_char`__: in case the keyboard buffer is
  non-empty, assign its next character to `%$char`, otherwise assign
  `-1` to `%$char`;

There are some equates, that the standard library defines,
corresponding to relevant keyboard keys: `RETURN` and `BACKSPACE`.  In
order to read an entire string at once there is a specialized
procedure:

- __`keyboard_read_str, %$buffer, %$max, %$color, %$outchar`__: read
  at most `%$max` printable ASCII characters, discarding the others,
  from the keyboard buffer, waiting if it is or becomes empty, and put
  them in `%$buffer`, until `RETURN` is read.  If `BACKSPACE` is read,
  discard the last character put into `%$buffer`, in case there is
  one, otherwise do nothing.  If `%$outchar` is `PRINT`, output each
  read character to screen with text attribute `%$color`, else if it
  is `NOPRINT` suppress output, otherwise output the character in
  `%$outchar` with text attribute `%$color`.  After returning
  `%$buffer` is a string (`END` terminated character sequence), and
  there is no `RETURN` character in it;

Notice that a value of, let's say, `'*'` in `%$outchar` is useful to
implement hidden password input.  Similarly, `NOPRINT` in `%$outchar`
can be used when acknowledgement of password length is a concern.

As you may have noticed, the above keyboard input routines don't read
characters directly from the keyboard but rather from its buffer.
More often than not one wants to ensure its buffer is empty prior to
invoking these procedures, as to synchronize reading and inputting,
giving the impression input is being read directly from the keyboard.
For that end, the remaining routines deal with flushing the keyboard
buffer.

- __`keyboard_flush_buffer`__: empty the keyboard buffer;
- __`keyboard_flush_buffer_from_char, %$char`__: keep discarding
  `%$char` from buffer until a different character is read;
- __`%$b, keyboard_flush_buffer_from_char_with_resistence, %$char,
  %$resist`__: if `%$char` is read `%$resist` times in a row, assign
  `?TRUE` to `%$b`, otherwise assign `?FALSE`.  Discard all
  occurrences of `%$char` from the buffer;

See the file `lib/kernel/keyboard.qdt` for the implementation details of
kernel keyboard routines.


### kernel/timer.qdt

This module has facilities that enable programs to wait synchronously
or asynchronously for a given amount of time.  The synchronous wait
functions are:

- __`timer_sleep, %$t`__: wait `%$t` clock ticks before returning.  A
  second has 18.2 clock ticks;
- __`timer_wait, %$m`__: wait `%$m` milliseconds before returning.

The asynchronous wait function is:

- __`%$b, timer_alarm, %$t`__: if `%$t` is non-zero, set the alarm to
  `%$t` clock ticks in the future and return nothing, else assign
  `?TRUE` to `%$b` if the alarm went off, otherwise assign `?FALSE` to
  it.

Notice that for precision sake, this function has to be called
frequently enough in order to make the time between calls
insignificant in comparison to the wait time.

The time can be specified in seconds to the `timer_sleep` and
`timer_alarm` functions by means of an intermediate macro:

- __`timer_seconds2ticks(s)`__: expands to the number of clock ticks
  `s` seconds are comprised of.

See the file `lib/kernel/timer.qdt` for the implementation details of
kernel timer routines.


### kernel/speaker.qdt

This module handles the internal IBM-PC speaker.  Currently it's
comprised solely of the following function:

- __`speaker_beep, %$f, %$t`__: produce a tone of frequency `%$f` for
  the duration of `%$t` clock ticks.

See the file `lib/kernel/speaker.qdt` for the implementation details
of kernel speaker routines.


### kernel/disk.qdt

This module provides low-level disk access and is used by the
`kernel/boot.qdt` module when booting from floppies, hard disks, or
something that emulates them.

- __`%$spt, %$tpc, disk_get_parameters, %$disk`__: assign the number
  of sectors per track and tracks per cylinder of disk `%$disk` to
  `%$spt` and `%$tpc`, respectively;
- __`%$c, %$h, %$s, disk_sector_to_chs, %$disk, %$sector`__: convert
  the logical sector `%$sector` of disk `%$disk` into its _CHS_ address,
  given by `%$c`, `%$h` and `%$s`;
- __`disk_read_sectors, %$buffer_segment, %$buffer_offset, %$disk,
  %$start_sector, %$count`__: read `%$count` sectors starting at the
  logical sector `%$start_sector` from disk `%$disk` to the memory
  buffer segment and offset given by `%$buffer_segment` and
  `%$buffer_offset`, respectively;

See the file `lib/kernel/disk.qdt` for the implementation details of
kernel disk routines.


### kernel/boot.qdt

This module is intended to provide the
_run-within-OS-or-bootstrap-itself-without-OS_ capability for programs
using the standard library.  It's special and must be used in a
different way than other common modules.  To use it one have to make
two changes: its symbol `boot_sector` must be imported at the entry
point block of the main module and the assembler must be instructed to
fill the final binary with zeroes so its size become sector-aligned
(divisible by 512 bytes).

In respect to the first requirement the following code must be put
right after the inclusion of the _QDot_ language definition
(file `qdot/qdot.qdt`):

<pre>
?import boot_sector
%include "kernel/boot.qdt"
</pre>

As for the second requirement, the following code must be placed
immediately above the beginning of the `.bss` section.

<pre>
times 512 - ($ - $$) % 512 db 0
PROGRAM_END:
</pre>

Borrowing the example from the
[Symbol importing]({{page.base_local}}{{site.baseurl}}/#symbol-importing)
section, one should finally have:

<pre>
%push SRC_PROG_QDT ; push a preprocessor context for this module

%ifndef SRC_PROG_QDT ; ensure this block won't be included twice
%define SRC_PROG_QDT

CPU 8086
ORG 100h

%include "qdot/qdot.qdt" ; enable QDot language

?import boot_sector         ; place the standard library's boot sector at
%include "kernel/boot.qdt"  ; the binary's first 512 bytes

mov sp, 0FFFEh ; setup stack for QDot expressions
mov bp, sp

call main  ; call the main procedure defined in this very module
           ; somewhere outside this block

?import main  ; import the main symbol used above.  This
              ; will expand to all the code it depends upon, and so on
              ; recursively, thus all the program's source code will
			  ; be compiled and assembled here
%include "src/prog.qdt"

; Make this program's size a multiple of the sector size (512 bytes)
; so the boot loader can exactly load it.

times 512 - ($ - $$) % 512 db 0

PROGRAM_END: ; This global symbol is used by the boot loader to
             ; calculate the binary's size

SECTION .bss ; here is the binary end
.
.
.
%endif ; SRC_PROG_QDT
.
. ; here are all importable symbol blocks defined by the
. ; main module
.
%pop SRC_TM_QDT ; the module ends here
</pre>

After these changes your program becomes simultaneously a bootable
image and a valid DOS executable.  It can be booted from a floppy
disk, a hard drive, an optical disk or even an USB mass storage
device.  In the case of optical disks, you can generate an ISO image
using the resulting executable as a valid _El Torito_ no-emulation
boot image.  For all other cases it should suffice to write the
executable to the very first sector of the drive.

See the file `lib/kernel/boot.qdt` for the implementation details of
kernel boot routines.


### kernel/debug.qdt

This module provides primitive debugging support for the boot process,
since a proper debugger is not usually available at that stage.

When inspecting the root of a problem, it's important to inspect
variable values and print information to the screen.

- __`debug_print_char, %$char`__: print character `%$char` in teletype
  mode;
- __`debug_print_hex_digit, %$digit`__: print the low hex digit value
  of `%$digit`;
- __`debug_print_byte, %$byte`__: print the low byte value of
  `%$byte`;
- __`debug_print_word, %$word`__: print the word value `%word`;


### math/random.qdt

This module provides a 32-bit Galois LFSR pseudo-random number
generator.  In order to use it one has to first initialize it.

- __`random_init`__: seed the random number generator from the current
  time;

Then, one can generate random numbers with:

- __`%$rnd, random_number, %$a, %$b`__: assign to `%$rnd` a random
  number between `%$a` and `%$b` inclusive;


### os/dos.qdt

This module handles functions specific to _DOS_.

- __`%$b, dos_check_if_running`__: assign `?RETURN` to `%$b` if _DOS_
  is running, `?FALSE` otherwise;
- __`dos_exit`__: quit program and return to _DOS_;





</div>
