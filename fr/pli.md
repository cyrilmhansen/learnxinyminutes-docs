---
translators:
    - ["Cyril Hansen"]
---

PL/I (*Programming Language One*, prononcé « PL one ») a été conçu
par IBM dans les années 1960 pour la programmation scientifique,
commerciale et système. Il réunit programmation structurée,
arithmétique décimale, chaînes de caractères et gestion des exceptions.

Ce tour d'horizon utilise **Digital Research PL/I-80** (1980), un
compilateur CP/M fondé sur le projet de sous-ensemble généraliste
(*General-Purpose Subset G*) du PL/I ANSI. D'autres compilateurs PL/I
offrent des fonctionnalités supplémentaires et parfois d'autres
conventions d'entrées-sorties.

Les commentaires du programme restent en ASCII pour faciliter sa
réutilisation avec les outils CP/M historiques.

```text
/* Commentaires : slash-etoile au debut, etoile-slash a la fin.
   PL/I ignore la casse, est en format libre et separe par des ';'. */

LEARN: PROCEDURE OPTIONS(MAIN);
    /* PL/I-80 permet des substitutions de texte a la compilation. */
    %REPLACE LIMIT BY 4;

    /* DCL abrege DECLARE. Les donnees se declarent explicitement. */
    DECLARE I FIXED BINARY(15);
    /* PL/I-80 impose STATIC avec INITIAL. */
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

    /* Une structure utilise des niveaux numeriques imbriques. */
    DECLARE 1 PERSON,
              2 PERSON_NAME CHARACTER(20),
              2 VISITS FIXED BINARY(15);

    /* PUT ecrit sur la console (SYSPRINT) par defaut. */
    PUT SKIP LIST('Hello, world!');

    /* = affecte une valeur. ** calcule une puissance. */
    I = 2 ** 3;                  /* 8 */
    TOTAL = I + 2;               /* 10 */
    AMOUNT = AMOUNT + 1.25;      /* 13.75 */

    PUT SKIP LIST('Total:', TOTAL);
    PUT SKIP LIST('Half:', HALF);

    /* EDIT formate la sortie : F(largeur, decimales). */
    PUT SKIP EDIT(AMOUNT) (F(7,2));

    /* Les chaines fixes sont completees par des espaces. */
    LABEL_TEXT = 'hi';

    /* Les chaines VARYING conservent leur longueur effective. */
    MESSAGE = 'PL/I' || ' on CP/M';  /* || concatene */
    PUT SKIP LIST(MESSAGE, LENGTH(MESSAGE));
    PUT SKIP LIST(SUBSTR(MESSAGE, 1, 4));
    SUBSTR(MESSAGE, 1, 4) = 'PL/1';

    /* Une comparaison donne BIT(1) ; & = ET, | = OU. */
    IF (TOTAL >= 10) & READY THEN DO;
        PUT SKIP LIST('Ready');
    END;
    ELSE DO;
        PUT SKIP LIST('Not ready');
    END;

    /* Les indices des tableaux commencent a 1 par defaut. */
    DO I = 1 TO LIMIT;
        NUMBERS(I) = I * I;
    END;

    DO I = 1 TO LIMIT;
        PUT SKIP LIST(I, NUMBERS(I));
    END;

    /* Autres variantes de boucle : BY et WHILE. */
    DO I = 2 TO 6 BY 2;
        PUT SKIP LIST('Even:', I);
    END;

    I = 3;
    DO WHILE (I > 0);
        I = I - 1;
    END;

    /* Tableaux numeriques et fonctions mathematiques, comme FORTRAN. */
    GRID(0,1) = SQRT(9.0);
    PUT SKIP LIST('Square root:', GRID(0,1));

    /* La memoire pointee peut survivre a son bloc d'origine. */
    ALLOCATE HEAP_VALUE SET(HEAP_PTR);
    HEAP_VALUE = 42;
    PUT SKIP LIST('Heap:', HEAP_PTR -> HEAP_VALUE);
    FREE HEAP_VALUE;
    HEAP_PTR = NULL();

    PERSON.PERSON_NAME = 'Ada';
    PERSON.VISITS = 2;
    PUT SKIP LIST(PERSON.PERSON_NAME, PERSON.VISITS);

    /* CALL appelle une procedure ; une fonction renvoie une valeur. */
    CALL SHOW_NUMBER(TOTAL);
    PUT SKIP LIST('5 factorial:', FACT(5));

    /* ON installe un gestionnaire ; SIGNAL declenche une condition. */
    ON ZERODIVIDE PUT SKIP LIST('Division by zero signalled');
    SIGNAL ZERODIVIDE;
    REVERT ZERODIVIDE;

    /* La lecture sur la console s'ecrit : GET LIST(I); */

    /* Fichier sequentiel CP/M (decommenter pour ecrire OUTPUT.DAT) :
       OPEN FILE(OUTPUT) STREAM OUTPUT;
       PUT FILE(OUTPUT) SKIP LIST('A line in a disk file');
       CLOSE FILE(OUTPUT);
    */

    STOP; /* Evite de poursuivre dans les corps de procedures. */

    SHOW_NUMBER: PROCEDURE(VALUE);
        DECLARE VALUE FIXED BINARY(15);
        PUT SKIP LIST('From procedure:', VALUE);
    END SHOW_NUMBER;

    /* Les procedures recursives exigent RECURSIVE. */
    FACT: PROCEDURE(N) RETURNS(FIXED BINARY(15)) RECURSIVE;
        DECLARE N FIXED BINARY(15);
        IF N <= 1 THEN RETURN(1);
        RETURN(N * FACT(N - 1));
    END FACT;
END LEARN;
```

Sur un système CP/M d'origine, enregistrez ce programme dans
`LEARN.PLI`, puis lancez `PLI LEARN`, `LINK LEARN` et `LEARN`.
Le compilateur produit un fichier objet `.REL` ; LINK-80 crée
un exécutable `.COM`. Ces commandes sont propres aux outils
Digital Research.

## Ce qui distingue PL/I

### Pointeurs et gestion explicite de la mémoire

PL/I dispose de pointeurs `POINTER` peu contraints, sans typage du
contenu pointé. Une déclaration `BASED` décrit l'organisation des
données à une adresse : elle n'alloue pas de mémoire par elle-même.
Dans l'exemple, `ALLOCATE ... SET(pointer)` réserve de la mémoire
dynamique et `FREE` la libère. La notation `pointer -> object`
explicite le pointeur utilisé ; `BASED(pointer)` lui associe un
pointeur implicite. `ADDR(variable)` permet aussi de superposer
une vue `BASED` à une variable existante.

Il n'y a pas de ramasse-miettes : le programmeur gère la durée de vie
des allocations. Contrairement à une variable `AUTOMATIC` ordinaire,
une zone `BASED` reste allouée jusqu'à sa libération explicite.
PL/I-80 implante les variables `AUTOMATIC` statiquement, sauf dans
les procédures récursives.

### Arithmétique décimale et conversions automatiques

En plus des entiers binaires et des flottants, PL/I propose des
nombres décimaux à virgule fixe, comme `FIXED DECIMAL(7,2)`.
Ils conviennent notamment aux montants monétaires :
`AMOUNT = AMOUNT + 1.25;` utilise l'arithmétique décimale, avec
une échelle définie, plutôt qu'un flottant binaire approximatif.

PL/I convertit automatiquement de nombreuses représentations
numériques et textuelles. Attention toutefois : **PL/I-80 convertit
un `FIXED DECIMAL` en `FIXED BINARY` lorsqu'on mélange ces types
dans une expression arithmétique** ; la partie fractionnaire
peut être perdue. Pour les calculs sensibles à la précision, gardez
des opérandes décimaux compatibles ou utilisez explicitement
`DECIMAL(...)`, `BINARY(...)` et `DIVIDE(...)`.

### Héritage de FORTRAN

La programmation scientifique a influencé les types `FLOAT BINARY`,
l'opérateur de puissance `**`, les boucles `DO` avec compteur et
les fonctions mathématiques `SQRT`, `SIN` et `LOG`.
Les tableaux multidimensionnels acceptent des bornes explicites :
`GRID(0:2,1:3)` contient trois lignes et trois colonnes.
Contrairement aux anciens sources FORTRAN à colonnes imposées,
PL/I emploie un format libre.

### Héritage de COBOL et écriture abrégée ou explicite

Pour la programmation de gestion, PL/I reprend l'arithmétique
décimale compacte, les structures hiérarchiques à membres numérotés,
les sorties formatées `PUT EDIT`, ainsi que les fichiers en mode
flux ou enregistrements.

PL/I permet des déclarations courtes ou descriptives. Ainsi,
`DCL RATE FIXED DEC(7,2);` et
`DECLARE RATE FIXED DECIMAL(7,2);` sont équivalents.
De même, `DCL TITLE CHAR(20) VAR;` équivaut à
`DECLARE TITLE CHARACTER(20) VARYING;`.
Autres couples : `PROC`/`PROCEDURE`, `PTR`/`POINTER`,
`INIT`/`INITIAL`. Ce sont des variantes de notation,
pas des versions différentes du langage ; les mots complets
favorisent en général la lisibilité.

### Subset G et héritage des grands systèmes IBM

IBM a développé PL/I à l'époque du System/360 pour rapprocher
les usages scientifiques de FORTRAN et les applications de gestion
de COBOL. Le projet de sous-ensemble généraliste (*Subset G*)
en a conservé un noyau utilisable sur davantage de machines.
Le PL/I-80 de Digital Research (1980) suivait ce projet avant
la publication de la norme ANSI X3.74-1981 ; le sous-ensemble G
a ensuite été révisé en 1987.

Même sur un micro-ordinateur 8 bits sous CP/M, PL/I-80 conservait
des mécanismes issus des grands systèmes : gestionnaires de
conditions `ON`, procédures imbriquées, liaison `ENTRY` et
`EXTERNAL`, pointeurs, types décimaux, entrées-sorties par flux
et par enregistrements. Ses mots-clés dépendent aussi du contexte :
beaucoup peuvent servir d'identifiants.

Ce n'est **pas le PL/I IBM complet**. PL/I-80 omet `DEFINED`,
`LIKE`, `FLOAT DECIMAL`, les tableaux dynamiques et l'attribut
de données `PICTURE`. La version 1.3 a toutefois ajouté le format
de sortie distinct `P(...)` pour `PUT EDIT` ; cela ne rétablit pas
l'attribut de données absent. Le sous-ensemble exclut également
le stockage `CONTROLLED` et le multitâche du langage complet.

## Pour aller plus loin

- [Manuel du langage PL/I-80 (Digital Research, décembre 1980)][language].
- [Guide d'applications PL/I-80 (Digital Research, 1980)][apps].
- [Compilateur PL/I Iron Spring pour Linux](https://www.iron-spring.com/)
- [IBM Research : histoire et caractéristiques du PL/I][history].

[history]: https://research.ibm.com/publications/the-early-history-and-characteristics-of-pli
[language]: https://ftpmirror.your.org/pub/misc/bitsavers/pdf/digitalResearch/pl1/PL1-80_Language_Manual_Dec80.pdf
[apps]: https://ftpmirror.your.org/pub/misc/bitsavers/pdf/digitalResearch/pl1/PL1-80_Applications_Guide_Dec80.pdf
