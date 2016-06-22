Syntax
----

The syntax of `Ro` is defined below, presented in Backus-Naur normal form (BNF).
Notice some similarities to existing languages such as `Python`, `Java`, and `OCaml`.
`Ro` draws inspiration from each of these programming languages. Note that terminal
productions below are shown in lowercase, while non-terminal symbols are shown in
uppercase.

```
# These are symbols that are "primitive" and already well-defined.
predefined symbols: FILE, INT, LONG, BOOL, CHAR, STRING, ID, CONSTRUCTOR

# A program is any number of import statements followed by a series of declarations.
PROGRAM ::=
  | IMPORT
    DECLS
    
# An import statement is either empty or consists of one or more "use <module>"
#   statements.
IMPORT ::=
  |
  | use FILE
    IMPORT

# A declaration is either empty or one declaration followed by (possibly) others.
DECLS ::=
  |
  | DECL DECLS

# An optional left parenthesis is either empty or a left parenthesis.
LPAREN ::=
  |
  | (
  
# An optional right parenthesis is either empty or a right parenthesis.
RPAREN ::=
  |
  | )

# A standalone declaration consists of one of the following:
#   • A type declaration, which could consist of one or more constructors
#   • A record type declaration, which could have many fields (always mutable)
#   • An error-type declaration, denoted by an identifier for the name
#   • A standard declaration, which could be normal or mutable (can use := only
#       if mutable)
#   • A function declaration with or without a type annotation, consisting of
#       one or more arguments and a body of statements
DECL ::=
  | type ID = CONSTRUCTORS
  | record ID = struct FIELDS end
  | error ID
  | STDECL
  | const STDECL
  | mutable STDECL
  | def ID ARGS =
      STATEMENTS
    end
  | def ID ARGS : EXTYPE =
      STATEMENTS
    end
    
# A collection of fields is either a single field or more than one field. Always
#   with type annotations.
FIELDS ::=
  | ID : TYPE
  | ID : TYPE ;
    FIELDS

# A standard declaration consists of one of the following:
#   • A variable declaration, with or without a type annotation
#   • A record instantiation (all fields are mutable)
#   • A list declaration, with or without a type annotation
STDECL ::=
  | var ID = EXPR
  | var ID : TYPE = EXPR
  | ID = new ID { FIELDS-DECL }
  | ID : TYPE = new ID { FIELDS-DECL }
  | list ID [ ]
  | ID [ ] : TYPE
  | list ID = EXPR
  | ID : TYPE = EXPR
  
# A declaration of fields for record instantiation.
FIELDS-DECL ::=
  | ID = EXPR
  | ID = EXPR ; FIELDS-DECL

# A constructor is either a single constructor or more than one of them.
CONSTRUCTORS ::=
  | CONSTRUCTOR of TYPE
  | CONSTRUCTOR of TYPE or CONSTRUCTORS

# Statements are either empty or consist of one or more statements.
STATEMENTS ::=
  |
  | STATEMENT STATEMENTS

# A standalone statement is one of the following:
#   • A declaration of any type, including functions (e.g. local functions
#       within functions)
#   • An assignment of an expression to a variable (which could be a list's name
#       or a record's name)
#   • An assignment of an expression to a record field
#   • An assignment of an expression to a list at some position
#   • A call to a subroutine/function defined earlier (i.e. do action with side
#       effects)
#   • An if-then statement or an if-then-else statement
#   • A while statement, whose body contains statements
#   • Raise an exception, with or without a body message
#   • A return statement, for values returned by a function (i.e. give back)
STATEMENT ::=
  | DECL
  | ID := EXPR
  | ID . ID := EXPR
  | ID [ EXPR_1 ] := EXPR_2
  | call ID LPAREN EXPRARGS RPAREN
  | if LPAREN EXPR RPAREN then STATEMENTS end
  | if LPAREN EXPR RPAREN then STATEMENTS_1 else STATEMENTS_2 end
  | while LPAREN EXPR RPAREN do
      STATEMENTS
    done
  | raise ID
  | raise ID STRING
  | give EXPR
  
# An expression is one of the following:
#   • The unit value or a primitive integer/float, boolean, or char/string
#   • An identifier, i.e. a variable's name (which could be a list's name or a
#       record's name)
#   • A record's field name
#   • An optional type with value nil
#   • An optional type with Some expression stored as a value
#   • An empty list ([])
#   • A (non-empty) list with some contents
#   • A list access at some position (must be an integer within bounds)
#   • Element concatenation (at the head of the list)
#   • List concatenation (glue two lists together, maintain relative order)
#   • The length of a list (could be a sublist)
#   • A pair of expressions (i.e. a pairing operator)
#   • The projection functions that find the left-half and the right-half of a
#       pair, respectively
#   • An anonymous function from an identifier to expressions (i.e. no curried
#       functions)
#   • Function application, but can only apply arguments to a named function
#       (i.e. take value of)
#   • A binary relation on expressions (binary operations)
#   • A unary relation on expressions (unary operations)
#   • An if-then-else expression (effectively a ternary operator)
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
  | ID . ID
  | nil
  | Some LPAREN EXPR RPAREN
  | [ ]
  | [ CONTENTS ]
  | LIST [ EXPR ]
  | EXPR_1 :: EXPR_2
  | EXPR_1 <-> EXPR_2
  | length LPAREN ID RPAREN
  | ( EXPR_1 , EXPR_2 )
  | left EXPR | right EXPR
  | lambda ID -> EXPR
  | take ID LPAREN EXPRARGS RPAREN
  | EXPR_1 BINOP EXPR_2
  | UNOP EXPR
  | if EXPR_1 then EXPR_2 else EXPR_3
  | CONSTRUCTOR LPAREN EXPR RPAREN
  | match EXPR with
      CASES
    end

# Contents (of a list) consist of one or more expressions joined together.
CONTENTS ::=
  | EXPR
  | EXPR ; CONTENTS

# Cases for a pattern match; at least one case must be specified for a given match.
#   If a match is to be used imperatively, then we use it as an expression.
CASES ::=
  | case PATTERN -> EXPR
  | case PATTERN -> EXPR
    CASES

# The contents of an argument are either empty or non-empty.
EXPRARGS ::=
  |
  | NON-EMPTY-EXPRARGS

# A non-empty argument is either a single argument or more than one argument.
NON-EMPTY-EXPRARGS ::=
  | EXPRARG
  | EXPRARG , NON-EMPTY-EXPRARGS

# A standalone argument is an expression, with or without special type annotations.
EXPRARG ::=
  | EXPR
  | EXPR : TYPE

# A pattern (to match over) is either a wildcard _, an empty list, a list pattern,
#   a primitive, a user-defined constructor, a pair, or optional-type related patterns.
#   Note that we do not permit pattern matching over strings. Its use is only apparent
#   for user-defined data types and pairs.
PATTERN ::=
  | _ | [ ] | PATTERN_1 :: PATTERN_2 | () | INT | LONG | BOOL | CHAR | ID | nil
  | CONSTRUCTOR LPAREN PATTERN RPAREN | ( PATTERN_1, PATTERN_2 ) | Some LPAREN PATTERN RPAREN

# A binary operation is one of the following:
#   • Addition, subtraction, multiplication, division, or modular arithmetic
#   • High-multiplication for 64/128-bit support (**)
#   • A comparision (LT, GT, EQ, LE, GE, NE)
#   • String concatenation (^)
#   • Logical or bitwise operations (and, or, xor), depending on the types used
#       as arguments
BINOP ::=
  | + | - | * | / | ** | < | > | = | <= | >= | \= | ^ | and | or | xor | mod

# A unary operation is either logical/bitwise negation (depending on type used)
#   or arithmetic negation.
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

# A standalone argument is an identifier, with or without a special type annotation.
ARG ::=
  | ID
  | ID : TYPE

# A type (for annotation purposes) is one of the following:
#   • A unit value
#   • A primitive type, like int, bool, or string
#   • A pair of types
#   • A list, whose elements are homogeneously of one type
#   • A user-defined type/record, specified by an identifier
#   • A function type (for lambda expressions captured by a variable)
#   • An option type (for safe null-ing checks)
TYPE ::=
  | unit
  | int
  | long
  | bool
  | char
  | string
  | TYPE_1 * TYPE_2
  | TYPE list
  | ID
  | TYPE -> TYPE
  | TYPE ?

# An extended type is either a regular type or a special void type (return nothing).
EXTYPE ::=
  | void
  | TYPE
```
