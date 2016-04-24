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
  | ID := EXPR
  | ID[] := EXPR
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
  | ID[EXPR]
  | (EXPR_1, EXPR_2)
  | left EXPR | right EXPR
  | lambda ID -> EXPR
  | ID (EXPRARGS)
  | EXPR_1 BINOP EXPR_2
  | UNOP EXPR
  | if EXPR_1 then EXPR_2 else EXPR_3
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
