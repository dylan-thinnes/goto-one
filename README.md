# GOTO-ONE - A language on a number line

## Number System

The language is composed almost entirely of arbitrary precision floating point
numbers from -1 to 1 (variable/function names, line numbers, and control flow
all depend on it). They can be written in a few ways:

0.1  (with a leading zero)
.1   (without a leading zero)
1e-1 (in scientific notation)
    12e-1 would not be legal: it is > 1
    1.2e-1 would be legal: it is <= 1
    12e-2 would be legal: it is <= 1
s1 (significand / mantissa form) (useful for bit flags)
    s0 would be 0
    s1 would be 0.1 in binary, or 0.5 in decimal
    s01 would be 0.01 in binary, or 0.25 in decimal
    s11 would be 0.11 in binary, or 0.75 in decimal

All "." are interchangeable with ",", for our European friends.

## Line Numbers and Indexing

Each line starts with a floating point number with arbitrary precision from -1
to 1, followed by a ":". Absence of such a marker indicates the current line is
a continuation of the previous one.

-0.1 : ...
0    : ...
0.1  : ...
       ...
0.11 : ...
0.2  : ...
0.25 : ...
0.3  : ...

The program always starts at 0, and by default progresses towards 1. This makes
the (-1,0) range useful for storing code that you don't want run by default.

Going to a new line is a simple as the "goto <number>" keyword.

Line 1 cannot contain values, and neither can line -1. Going to either
indicates that we exit the program.

## Watching / Waiting (Commiserating)

Normally, the pointer speed is "infinite", in that it will traverse the number
line at the maximum speed possible.

However, one can set the rate at which the pointer traverses the number line
can be set using the "speed <number>" keyword. This will (attempt) to time the
rate at which number lines are traversed, in that the number denotes the
lines/sec.

For example, in the following program, 0.7 will run 100 seconds after 0.6,
because (0.7 - 0.6) / 0.001 = 100.

0.6 : speed 0.001
0.7 : speed infinity ""

This speed can also be negative (including negative infinity), allowing fun
pingpong patterns!

0.0 : speed infinity
0.1 : run_a
0.2 : run_b
0.3 : speed -infinity

This program will cycle infinitely, executing run_a, then run_b, then run_b,
then run_a, then run_a, then run_b, then run_b etc. 
    (Put succinctly: abbaabbaabbaabba...)
    (I call the code sample above the "On and On and On")

### Breakpointing

Set speed to 0 to place a breakpoint in your code, which allows for simple
runtime debugging.

## Subprograms

Subprograms are new (-1,1) "tapes" on which code can be placed. They are
started with a left paren and ended with a right paren

0.3 : (..., ...)

A double colon, "::", indicates the following code should be merged into a
subprogram with the previous line or pending subprogram.

0.3 : 0.1 : ...
   :: 0.5 : ...

desugars to

0.3 : (0.1 : ..., 0.5 : ...)

Subprograms can be arbitrarily nested, but :: will always merge into the most
deeply nested line that precedes it. It does not merge with the previous
subprogram. For example:

0.3 : 0.2 : (..., ...)
   :: 0.4 : line_a

turns into:

0.3 : (0.3 : (..., ...), 0.4 : line_a)

GOTOs can index further into subprograms by providing more arguments. For
example, the line at 0.5 will run "run_b"

0.3 : (0.1 : run_a, 0.5: run_b)
0.4 : run_c
0.5 : goto 0.3 0.5

GOTOs always index from the global scope downwards - if "goto 0.3 0.5" were
called from within the subprogram on line 0.3, the GOTO would still go to the
same place.

A second GOTO keyword, GOTO*, always operates from the current scope.

## Commenting

Comments can be started with ' and ended with a newline
Comments can be started with " and only ended with "

## Variables

Variables' names are only floating point numbers from -1 to 1. 
They are declared and read from using the "get" and "set" directives

set 0.1 0.2
get 0.1     ' will return 0.2
set -0.1 0.3
get -0.1    ' will return 0.3

There exist synonyms for each of these: 
= for set (so 0.1 = 0.2 would be set 0.1 0.2)
. for get (so .0.1 would be "get 0.1", or ..21 would be "get 0.21")

## Precision & Rounding Modes

Flow control can be governed by GOTOs dictated by precision and rounding modes.
There are no conditional statements, such as "if".

prec goto 0.3
prec value 0.3
prec variable 0.3

prec ROUND_TYPE NUMBER
round ROUND_TYPE ROUND_BEHAVIOUR

ROUND_TYPE = VALUE | GOTO | VARIABLE
ROUND_BEHAVIOUR = NEAREST_TIES_EVEN | NEAREST_TIES_AWAY_ZERO 
                | TOWARD_ZERO | TOWARD_POS | TOWARD_NEG

subprograms have their own precision contexts, though they can import the
precision from either their calling or their declaration context if necessary

IMPORT (CALL | DECL) ROUND_TYPE (PREC | ROUND)

### While Loop Examples

Using precision magic / rounding modes:

    round GOTO TOWARD_POS
0.15:
    set 1 run_my_code
    precision GOTO (0.1 * 0.1 ** get 1)
    goto 0.15
0.20:
    precision GOTO infinite

Using everyday goto work:

0.15:
    set 1 run_my_code
    goto (0.20 - 0.05 * get 1)
0.20:

## On Whitespace

Gotone is insensitive to the use of tabs and newlines - all whitespace is
interpreted identically, as a way to delimit tokens.

## Bitwise Operations

Pretty simple. Any bitwise ops treat numbers as significands composed of bits
and then operates as such.

## Strings & Printing

Printing is done using the "int" keyword. Floating point values can be printed
as strings of ascii characters using the "ascii" keyword. Each successive 8
bits in the mantissa represents a new character.

set 0.1 0.5
int (get 0.1)           ' prints 0.5
int (..1)               ' prints 0.5
str m01000001           ' prints "a"
str m0100000101000010   ' prints "ab"

The minimum precision necessary to express the full value in a variable can be
gotten with the "epsilon" keyword.

set 0.1 s0011 ' the same as 0.0011 in binary notation
epsilon 0.1   ' evaluates to s0001

Thus, appending two values that represent strings is as simple as:
(<a> + epsilon <a> * s1 * <b>)
