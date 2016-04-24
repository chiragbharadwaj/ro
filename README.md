# chi
This is compiler for `Chi`, an imperative programming language created by Chirag Bharadwaj at Cornell University. The compiler, which will eventually be written in `OCaml` (a functional language with near-`C` performance), will compile to the `x86-64` assembly language.

To-Do List
----
* Develop a lexer/tokenizer for the language specification.
* Develop a parser for the language specification.
* Write an interpreter for the language specification.
  + Implement an expression evaluator.
  + Implement a type-checker for the language specification.
  + Implement type unification for the expressions.
  + Implement type inference for the expressions.
* Figure out how the heck to write a compiler...

Syntax
----

The syntax of `Chi` is defined below, presented in Backus-Naur normal form (BNF). Notice some similarities to existing languages such as `Python`, `Java`, and `OCaml`. `Chi` draws inspiration from each of these programming languages. Note that terminal productions below are shown in lowercase, while non-terminal symbols are shown in uppercase.

```
predefined symbols: INT, BOOL, STRING, ID, CONSTRUCTOR

PROGRAM ::=
  | IMPORT
    STATEMENTS
  
IMPORT ::=
  |
  | use ID
    IMPORT
    
STATEMENTS ::=
  |
  | STATEMENT STATEMENTS
  
STATEMENT ::=
  | DECL
  | ID = EXPR
  | ID[] = EXPR
  | if EXPR then STATEMENT_1 else STATEMENT_2
  | while EXPR do
      STATEMENTS
    done

DECL ::=
  | type ID = CONSTRUCTOR of TYPE
  | var ID = EXPR
  | var ID : TYPE = EXPR
  | def ID ARGS =
      BLOCK
    end
  | def ID ARGS : TYPE =
      BLOCK
    end
  
EXPR ::=
  | ()
  | INT
  | BOOL
  | STRING
  | ID
  | lambda ID -> EXPR
  | ID (EXPRARGS)
  | EXPR_1 BINOP EXPR_2
  | UNOP EXPR
  | if EXPR_1 then EXPR_2 else EXPR_3
  | (EXPR_1, EXPR_2)
  | left EXPR | right EXPR
  | CONSTRUCTOR EXPR
  | match EXPR with
      case PATTERN_1 -> EXPR_1 end
        ...
      case PATTERN_N -> EXPR_N end
  
EXPRARGS ::=
  |
  | EXPRARG EXPRARGS

EXPRARG ::=
  | EXPR
  | EXPR : TYPE
  
PATTERN ::=
  | () | INT | BOOL | STRING | ID | CONSTRUCTOR PATTERN | (PATTERN_1, PATTERN_2)
  
BLOCK ::=
  |
  | EXPR
    BLOCK
  
BINOP ::=
  | + | - | * | ** | < | > | = | <= | >= | \= | << | >> | ^ | and | or | xor | mod

UNOP ::=
  | - | not

ARGS ::=
  |
  | ARG
    ARGS
  
ARG ::=
  | ID
  | ID : TYPE
  
TYPE ::=
  | unit
  | int
  | bool
  | string
  | TYPE list
  | (TYPE_1, TYPE_2) pair
```

Upcoming Features
----
We will update this list with upcoming features in future builds as we get them up and rolling.

* Adding support for comments.
  + That is, in-line comments as well as out-of-line block-style comments.
* Adding richer type-declarations.

Privacy Policy
----

Copyright &copy; 2015-2016 Chirag Bharadwaj, Cornell University.

There is NO warranty. This software may be freely redistributed with attribution to the original author in all cases. This software may also be modified from its original version and redistributed as long as all changes made are stated very clearly in the redistributed version and attribution to the original author is provided. You may not use ANY part of this software for commercial purposes. All rights reserved. Please report any bugs to the development team by making a pull request to this repository.
