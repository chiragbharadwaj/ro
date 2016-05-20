Syntax
----

The syntax of `Boo` is defined below, presented in Backus-Naur normal form (BNF). Notice some similarities to existing languages such as `Python`, `Java`, and `OCaml`. `Boo` draws inspiration from each of these programming languages. Note that terminal productions below are shown in lowercase, while non-terminal symbols are shown in uppercase.

```
# These are symbols that are "primitive" and already well-defined.
predefined symbols: FILE, INT, LONG, BOOL, CHAR, STRING, ID, CONSTRUCTOR

# A program is any number of import statements followed by a series of declarations.
PROGRAM ::=
  | IMPORT
    DECLS
    
# An import statement is either empty or consists of one or more "use <module>" statements.
IMPORT ::=
  |
  | use FILE
    IMPORT

# A declaration is either empty or one declaration followed by (possibly) others.
DECLS ::=
  |
  | DECL DECLS

# A standalone declaration consists of one of the following:
#   • A type declaration, which could consist of one or more constructors
#   • A variable declaration, with or without a type annotation
#   • A list declaration, with or without a type annotation, always fixed-size (empty or full)
#   • A function declaration with or without a type annotation, consisting of one or more arguments and
#       a body of statements
DECL ::=
  | type ID = CONSTRUCTORS
  | var ID = EXPR
  | var ID : TYPE = EXPR
  | list DECLIST [ INT ]
  | DECLIST [ INT ] : TYPE
  | list ID = [ CONTENTS ]
  | ID : TYPE = [ CONTENTS ]
  | def ID ARGS =
      STATEMENTS
    end
  | def ID ARGS : EXTYPE =
      STATEMENTS
    end

# This is a list-type specifically for declarations (i.e. passing in integers as arguments); standard
#   or multidimensional in size.
DECLIST ::=
  | ID
  | DECLIST [ INT ]

# A constructor is either a single constructor or more than one of them.
CONSTRUCTORS ::=
  | CONSTRUCTOR of TYPE
  | CONSTRUCTOR of TYPE or CONSTRUCTORS

# A list is either a standalone identifier or a multidimensional version of a list.
LIST ::=
  | ID
  | LIST [ EXPR ]

# The contents of a list are either empty or non-empty.
CONTENTS ::=
  |
  | NON-EMPTY-CONTENTS

# Non-empty-contents either consist of one or more super-expressions.
NON-EMPTY-CONTENTS ::=
  | ARR
  | ARR ; NON-EMPTY-CONTENTS

# A so-called "super-expression" is either an expression or another nested list.
ARR ::=
  | EXPR
  | [ CONTENTS ]

# Statements are either empty or consist of one or more statements.
STATEMENTS ::=
  |
  | STATEMENT STATEMENTS

# A standalone statement is one of the following:
#   • A declaration of any type, including functions (e.g. local functions within functions)
#   • An assignment of an expression to a variable (which could be a list's name)
#   • An assignment of an expression to a list at some position
#   • An if-then statement or an if-then-else statement
#   • A while statement, whose body contains statements
#   • A return statement, for values returned by a function
STATEMENT ::=
  | DECL
  | ID := EXPR
  | LIST [ EXPR_1 ] := EXPR_2
  | if ( EXPR ) then STATEMENTS end
  | if ( EXPR ) then STATEMENTS_1 else STATEMENTS_2 end
  | while ( EXPR ) do
      STATEMENTS
    done
  | return EXPR
  
# An expression is one of the following:
#   • The unit value or a primitive integer, boolean, or string
#   • An identifier, i.e. a variable's name (which could be a list's name)
#   • A list access at some position
#   • The length of a list (could be a sublist)
#   • A pair of expressions
#   • The projection functions that find the left-half and the right-half of a pair, respectively
#   • An anonymous function from an identifier to expressions (i.e. no curried functions)
#   • Function application (can only apply arguments to a named function)
#   • A binary relation on expressions (binary operations)
#   • A unary relation on expressions (unary operations)
#   • An if-then-else expression (no if-then expressions permitted for simplicity)
#   • A user-defined type constructor, called with an expression as an argument
#   • A pattern-matching device for constructors, pairs, and primitive datatypes
EXPR ::=
  | ()
  | INT
  | LONG
  | BOOL
  | CHAR
  | STRING
  | ID
  | LIST [ EXPR ]
  | length ( PARLIST )
  | ( EXPR_1 , EXPR_2 )
  | left EXPR | right EXPR
  | lambda ID -> EXPR
  | ID ( EXPRARGS )
  | EXPR_1 BINOP EXPR_2
  | UNOP EXPR
  | if EXPR_1 then EXPR_2 else EXPR_3
  | CONSTRUCTOR ( EXPR )
  | match EXPR with
      CASES
    end

# Cases for a pattern match; at least one case must be specified for a given match. If a
#   match is to be used imperatively, then we use the fact that it is an expression.
CASES ::=
  | case PATTERN -> EXPR
  | case PATTERN -> EXPR
    CASES

# A partial list is a sublist of a list (the whole list itself or subportions of it).
PARLIST ::=
  | ID
  | PARLIST [ INT ]

# The contents of an argument are either empty or non-empty.
EXPRARGS ::=
  |
  | NON-EMPTY-EXPRARGS

# A non-empty argument is either a single argument or more than one argument.
NON-EMPTY-EXPRARGS ::=
  | EXPRARG
  | EXPRARG , NON-EMPTY-EXPRARGS

# A standalone argument is an expression, with or without type annotations.
EXPRARG ::=
  | EXPR
  | EXPR : TYPE

# A pattern (to match over) is either a wildcard _, a primitive, a user-defined constructor, or a pair.
#   Note that we do not permit pattern matching over strings or list types. Its use is only apparent
#   for user-defined data types and pairs.
PATTERN ::=
  | _ | () | INT | LONG | BOOL | CHAR | ID | CONSTRUCTOR ( PATTERN ) | ( PATTERN_1, PATTERN_2 )

# A binary operation is one of the following:
#   • Addition, subtraction, multiplication, division, or modular arithmetic
#   • High-multiplication for 64/128-bit support (**)
#   • A comparision (LT, GT, EQ, LE, GE, NE)
#   • String concatenation (^)
#   • Logical or bitwise operations (and, or, xor), depending on the types used as arguments
BINOP ::=
  | + | - | * | / | ** | < | > | = | <= | >= | \= | ^ | and | or | xor | mod

# A unary operation is either logical/bitwise negation (depending on type used) or arithmetic negation.
UNOP ::=
  | - | not

# Arguments to a function are either empty or non-empty.
ARGS ::=
  |
  | ( NON-EMPTY-ARGS )

# A non-empty argument is either a single argument or more than one argument.
NON-EMPTY-ARGS ::=
  | ARG
  | ARG , NON-EMPTY-ARGS

# A standalone argument is an identifier, with or without a type annotation.
ARG ::=
  | ID
  | ID : TYPE

# A type (for annotation purposes) is one of the following:
#   • A unit value
#   • A primitive type, like int, bool, or string
#   • A pair of types
#   • A list, whose elements are homogeneously of one type
TYPE ::=
  | unit
  | int
  | long
  | bool
  | char
  | string
  | TYPE_1 * TYPE_2
  | TYPE list
  
# An extended type is either a regular type or a special void type (for functions: return nothing).
EXTYPE ::=
  | void
  | TYPE
```
