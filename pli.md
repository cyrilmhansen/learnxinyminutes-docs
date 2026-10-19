---
name: PL/I
filename: learnpli.pli
contributors:
    - ["Cyril Hansen"]
---

PL/I (Programming Language One, pronounced "PL one") was designed by IBM
in the 1960s for scientific, business, and systems programming. It combines
structured programming, decimal arithmetic, strings, and exception handling.

This tour uses **Digital Research PL/I-80** (1980), a CP/M implementation
based on the proposed ANSI PL/I General-Purpose Subset (Subset G).
Other PL/I compilers offer additional features and sometimes different I/O
conventions.

```text
/* Comments start with slash-star and end with star-slash.
   PL/I is case-insensitive, free-format, and uses semicolons. */

LEARN: PROCEDURE OPTIONS(MAIN);
    /* PL/I-80 adds a simple compile-time substitution facility. */
    %REPLACE LIMIT BY 4;

    /* DCL is short for DECLARE. Data must be declared explicitly. */
    DECLARE I FIXED BINARY(15);
    /* PL/I-80 requires STATIC for declarations using INITIAL. */
    DECLARE TOTAL FIXED BINARY(15) STATIC INITIAL(0);
    DECLARE AMOUNT FIXED DECIMAL(7,2) STATIC INITIAL(12.50);
    DECLARE HALF FLOAT BINARY(24) STATIC INITIAL(0.5);
    DECLARE READY BIT(1) STATIC INITIAL('1'B);
    DECLARE LABEL_TEXT CHARACTER(12) STATIC INITIAL('hello');
    DECLARE MESSAGE CHARACTER(40) VARYING;
    DECLARE NUMBERS(LIMIT) FIXED BINARY(15);
    DECLARE GRID(0:2,1:3) FLOAT BINARY(24);
    DECLARE HEAP_PTR POINTER;
    DECLARE HEAP_VALUE FIXED BINARY(15) BASED(HEAP_PTR);
    DECLARE OUTPUT FILE;

    /* A structure uses numeric nesting levels, like a record. */
    DECLARE 1 PERSON,
              2 PERSON_NAME CHARACTER(20),
              2 VISITS FIXED BINARY(15);

    /* PUT outputs to the console (SYSPRINT) by default. */
    PUT SKIP LIST('Hello, world!');

    /* = assigns a value. ** is exponentiation. */
    I = 2 ** 3;                  /* 8 */
    TOTAL = I + 2;               /* 10 */
    AMOUNT = AMOUNT + 1.25;      /* 13.75 */

    PUT SKIP LIST('Total:', TOTAL);
    PUT SKIP LIST('Half:', HALF);

    /* EDIT offers formatted output: F(width, decimal places). */
    PUT SKIP EDIT(AMOUNT) (F(7,2));

    /* Fixed-length strings are padded with spaces. */
    LABEL_TEXT = 'hi';

    /* VARYING strings keep their current length. */
    MESSAGE = 'PL/I' || ' on CP/M';  /* || concatenates */
    PUT SKIP LIST(MESSAGE, LENGTH(MESSAGE));
    PUT SKIP LIST(SUBSTR(MESSAGE, 1, 4));
    SUBSTR(MESSAGE, 1, 4) = 'PL/1';

    /* Comparisons yield BIT(1); & means AND, | means OR. */
    IF (TOTAL >= 10) & READY THEN DO;
        PUT SKIP LIST('Ready');
    END;
    ELSE DO;
        PUT SKIP LIST('Not ready');
    END;

    /* Arrays default to one-based indexing. */
    DO I = 1 TO LIMIT;
        NUMBERS(I) = I * I;
    END;

    DO I = 1 TO LIMIT;
        PUT SKIP LIST(I, NUMBERS(I));
    END;

    /* Alternative loop forms: BY and WHILE. */
    DO I = 2 TO 6 BY 2;
        PUT SKIP LIST('Even:', I);
    END;

    I = 3;
    DO WHILE (I > 0);
        I = I - 1;
    END;

    /* FORTRAN-like numerical arrays and mathematical functions. */
    GRID(0,1) = SQRT(9.0);
    PUT SKIP LIST('Square root:', GRID(0,1));

    /* Pointer-based storage can outlive the creating block. */
    ALLOCATE HEAP_VALUE SET(HEAP_PTR);
    HEAP_VALUE = 42;
    PUT SKIP LIST('Heap:', HEAP_PTR -> HEAP_VALUE);
    FREE HEAP_VALUE;
    HEAP_PTR = NULL();

    PERSON.PERSON_NAME = 'Ada';
    PERSON.VISITS = 2;
    PUT SKIP LIST(PERSON.PERSON_NAME, PERSON.VISITS);

    /* CALL invokes a procedure; a function returns a value. */
    CALL SHOW_NUMBER(TOTAL);
    PUT SKIP LIST('5 factorial:', FACT(5));

    /* ON installs an exception handler; SIGNAL raises a condition. */
    ON ZERODIVIDE PUT SKIP LIST('Division by zero signalled');
    SIGNAL ZERODIVIDE;
    REVERT ZERODIVIDE;

    /* Input from the console would be: GET LIST(I); */

    /* Stream file I/O on CP/M (uncomment to write OUTPUT.DAT):
       OPEN FILE(OUTPUT) STREAM OUTPUT;
       PUT FILE(OUTPUT) SKIP LIST('A line in a disk file');
       CLOSE FILE(OUTPUT);
    */

    STOP; /* Do not fall through into the procedure bodies. */

    SHOW_NUMBER: PROCEDURE(VALUE);
        DECLARE VALUE FIXED BINARY(15);
        PUT SKIP LIST('From procedure:', VALUE);
    END SHOW_NUMBER;

    /* Recursive procedures must explicitly say RECURSIVE. */
    FACT: PROCEDURE(N) RETURNS(FIXED BINARY(15)) RECURSIVE;
        DECLARE N FIXED BINARY(15);
        IF N <= 1 THEN RETURN(1);
        RETURN(N * FACT(N - 1));
    END FACT;
END LEARN;
```

To build on an original PL/I-80 CP/M system, save the source as
`LEARN.PLI`, then run `PLI LEARN`, `LINK LEARN`, and `LEARN`.
The compiler produces a `.REL` object; LINK-80 produces a `.COM`
program. These commands are specific to the Digital Research toolchain.

## What makes PL/I distinctive

### Pointers and explicit storage management

PL/I has relatively permissive, untyped `POINTER` values. A `BASED`
declaration describes a layout at an address, rather than allocating
storage itself. As shown above, `ALLOCATE ... SET(pointer)` reserves
heap space and `FREE` releases it. `pointer -> object` explicitly
qualifies an access; `BASED(pointer)` supplies an implicit pointer.
`ADDR(variable)` also permits a `BASED` view of existing storage.

There is no garbage collector: the programmer manages the lifetime of
allocated blocks. Unlike ordinary `AUTOMATIC` storage, `BASED` storage
remains allocated until explicitly freed. PL/I-80 implements
`AUTOMATIC` variables statically except in recursive procedures.

### Decimal arithmetic and automatic conversions

Alongside binary integers and floating point, PL/I offers fixed-point
decimal numbers such as `FIXED DECIMAL(7,2)`. This is useful for money:
`AMOUNT = AMOUNT + 1.25;` performs decimal arithmetic with a specified
scale rather than approximate binary floating-point arithmetic.

PL/I automatically converts many numeric and string representations.
But conversions are not always lossless: **PL/I-80 converts
`FIXED DECIMAL` to `FIXED BINARY` when those types are mixed in an
arithmetic expression**, potentially discarding a fractional part.
Use compatible decimal operands or explicit `DECIMAL(...)`,
`BINARY(...)`, and `DIVIDE(...)` conversions where precision matters.

### FORTRAN heritage

Scientific programming influenced PL/I's `FLOAT BINARY`, `**`
exponentiation, counted `DO` loops, and mathematical functions such
as `SQRT`, `SIN`, and `LOG`. Multi-dimensional arrays can specify
their bounds: `GRID(0:2,1:3)` has three rows and three columns.
Unlike FORTRAN's historical fixed-column source, PL/I is free-format.

### COBOL heritage and long versus abbreviated spelling

Business programming shows up in packed-decimal arithmetic, hierarchical
structures with numbered members, formatted `PUT EDIT` output, and
both stream and record-oriented file I/O.

PL/I also offers terse and descriptive forms of the same declarations.
For example, `DCL RATE FIXED DEC(7,2);` and
`DECLARE RATE FIXED DECIMAL(7,2);` are equivalent. Likewise,
`DCL TITLE CHAR(20) VAR;` means the same as
`DECLARE TITLE CHARACTER(20) VARYING;`.
Other pairs include `PROC`/`PROCEDURE`, `PTR`/`POINTER`, and
`INIT`/`INITIAL`. These are interchangeable spellings, not
different language versions; full words generally aid readability.

### Subset G and the IBM mainframe legacy

IBM developed PL/I for the System/360 era to span scientific FORTRAN
and commercial COBOL workloads. The proposed General-Purpose Subset G
retained a core suited to more machines. Digital Research's 1980
PL/I-80 followed that proposal before ANSI X3.74-1981 was published;
the Subset G standard was later revised in 1987.

Even on an 8-bit CP/M computer, PL/I-80 preserved sophisticated
mainframe-era facilities: `ON` condition handlers, nested procedures,
`ENTRY` and `EXTERNAL` linkage, pointers, decimal types, and both
stream and record I/O. Keywords are context-sensitive, too: many can
also be used as identifiers.

It is **not full IBM PL/I**. PL/I-80 omits `DEFINED`, `LIKE`,
`FLOAT DECIMAL`, dynamic arrays, and the `PICTURE` **data attribute**.
Version 1.3 did add the distinct `P(...)` picture-style output format
for `PUT EDIT`; this does not add the missing data attribute.
It also omits full-language facilities such as `CONTROLLED` storage
and multitasking.

## Further reading

- [Digital Research PL/I-80 Language Manual (December 1980)][language].
- [Digital Research PL/I-80 Applications Guide (December 1980)][apps].
- [Iron Spring PL/I compiler for Linux](https://www.iron-spring.com/)
- [IBM Research: The Early History and Characteristics of PL/I][history].

[history]: https://research.ibm.com/publications/the-early-history-and-characteristics-of-pli
[language]: https://ftpmirror.your.org/pub/misc/bitsavers/pdf/digitalResearch/pl1/PL1-80_Language_Manual_Dec80.pdf
[apps]: https://ftpmirror.your.org/pub/misc/bitsavers/pdf/digitalResearch/pl1/PL1-80_Applications_Guide_Dec80.pdf
