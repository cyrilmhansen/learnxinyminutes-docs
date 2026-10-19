---
name: PL/I
filename: learnpli.pli
contributors:
    - ["cyrilmhansen", "https://github.com/cyrilmhansen"]
---

PL/I (Programming Language One, pronounced "PL one") was designed by IBM
in the 1960s for scientific, business, and systems programming. It combines
structured programming, decimal arithmetic, strings, and exception handling.

This tour uses **Digital Research PL/I-80** (1980), a CP/M implementation
based on ANSI PL/I Subset G. Other PL/I compilers support additional features
and sometimes different I/O conventions.

```text
/* Comments start with slash-star and end with star-slash.
   PL/I is case-insensitive, free-format, and uses semicolons. */

LEARN: PROCEDURE OPTIONS(MAIN);
    /* PL/I-80 adds a simple compile-time substitution facility. */
    %REPLACE LIMIT BY 4;

    /* DCL is short for DECLARE. Data must be declared explicitly. */
    DECLARE I FIXED BINARY(15);
    DECLARE TOTAL FIXED BINARY(15) INITIAL(0);
    DECLARE AMOUNT FIXED DECIMAL(7,2) INITIAL(12.50);
    DECLARE HALF FLOAT BINARY(24) INITIAL(0.5);
    DECLARE READY BIT(1) INITIAL('1'B);
    DECLARE LABEL_TEXT CHARACTER(12) INITIAL('hello');
    DECLARE MESSAGE CHARACTER(40) VARYING;
    DECLARE NUMBERS(LIMIT) FIXED BINARY(15);
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

## Important distinctions

- `FIXED BINARY(15)` is a signed 16-bit integer on PL/I-80.
  `FIXED DECIMAL(p,s)` supports packed-decimal arithmetic.
- PL/I-80 has `DO`, `IF`, `PROCEDURE`, nested blocks, and `ON`
  conditions, but not every feature of full IBM PL/I.
- PL/I-80 lacks `PICTURE`, `DEFINED`, and dynamic arrays; do not
  assume a sample from a modern PL/I compiler works unchanged.

## Further reading

- Digital Research's *PL/I-80 Language Manual* (December 1980)
  and *PL/I-80 Applications Guide* (December 1980) are in the
  [DR PL/I manual collection][manuals].
- [Iron Spring PL/I compiler for Linux](https://www.iron-spring.com/)

[manuals]: https://github.com/cyrilmhansen/drpli/tree/main/manuals
