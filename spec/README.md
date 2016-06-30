Syntax
----

The syntax of `Ro` is defined below, presented in Backus-Naur normal form (BNF).
Notice some similarities to existing languages such as `Python`, `Haskell`, and `OCaml`.
`Ro` draws inspiration from each of these programming languages. Note that terminal
productions below are shown in lowercase, while non-terminal symbols are shown in
uppercase.

```
# These are symbols that are "primitive" and already well-defined.
predefined symbols: FILE, BYTE, INT, LONG, FLOAT, BOOL, CHAR, STRING, ID, CONSTRUCTOR

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
#   • An alias to another type (textual substitution performed at assembler/link-time)
#   • An error-type declaration, denoted by an identifier for the name
#   • A standard declaration, which could be normal or mutable (can use := only
#       if mutable)
#   • A function declaration with or without a type annotation, consisting of
#       one or more arguments and a body of statements
DECL ::=
  | type ID = CONSTRUCTORS
  | record ID = struct { FIELDS }
  | alias ID = TYPE
  | Error ID
  | STDECL
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
#   • A list declaration/initialization, with or without a type annotation
STDECL ::=
  | var ID = EXPR
  | var ID : TYPE = EXPR
  | ID = new ID { FIELDS-DECL }
  | ID [ ]
  | ID [ ] : TYPE
  | ID = EXPR
  | ID : TYPE = EXPR
  
# A declaration of fields for record instantiation.
FIELDS-DECL ::=
  | EXPR
  | EXPR ; FIELDS-DECL
  | ID = EXPR
  | ID = EXPR ; FIELDS-DECL

# A constructor is either a single constructor or more than one of them. Here, "|"
# escapes the meta-syntax: We are using it to mean |.
CONSTRUCTORS ::=
  | NAME
  | NAME "|" CONSTRUCTORS

# A constructor name is just a primitive constructor, with or without arguments
NAME ::=
  | CONSTRUCTOR
  | CONSTRUCTOR of TYPE

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
#       effects... called subroutine MUST have a return type of Void)
#   • An if-then statement or an if-then-else statement
#   • A while statement, whose body contains statements
#   • Raise an exception, with or without a body message
#   • A return statement, for values returned by a function (i.e. give back)
STATEMENT ::=
  | DECL
  | ID := LPAREN EXPR RPAREN
  | ID . ID := LPAREN EXPR RPAREN
  | ID [ EXPR_1 ] := EXPR_2
  | call ID LPAREN EXPRARGS RPAREN
  | if LPAREN EXPR RPAREN then STATEMENTS end
  | if LPAREN EXPR RPAREN then STATEMENTS_1 else STATEMENTS_2 end
  | while LPAREN EXPR RPAREN do
      STATEMENTS
    done
  | raise ID
  | raise ID LPAREN STRING RPAREN
  | give LPAREN EXPR RPAREN
  
# An expression is one of the following:
#   • The unit value or a primitive byte, integer/float, boolean, or char/string
#   • An identifier, i.e. a variable's name (which could be a list's name or a
#       record's name)
#   • A record's field name
#   • An optional type with value None
#   • An optional type with Just some expression stored as a value
#   • An empty list ([])
#   • A (non-empty) list with some contents
#   • A list access at some position (must be an integer within bounds)
#   • Element concatenation (at the head of the list)
#   • List concatenation (glue two lists together, maintain relative order)
#   • The length of a list (could be a sublist)
#   • A pair of expressions (i.e. a pairing operator)
#   • The projection functions that find the left-half and the right-half of a
#       pair, respectively (called the front and back of the pair)
#   • An anonymous function from an identifier to expressions (i.e. no curried
#       functions)
#   • Function application, but can only apply arguments to a named function
#       (i.e. take value of)
#   • A binary relation on expressions (binary operations)
#   • A unary relation on expressions (unary operations)
#   • An if-then-else expression (effectively a ternary operator)
#   • A user-defined type constructor, with or without an expression as an argument
#   • An invocation of the Left or Right parts of an Either type (i.e. wrapping)
#   • A pattern-matching device for constructors, pairs, and primitive datatypes
EXPR ::=
  | ( )
  | BYTE
  | INT
  | LONG
  | FLOAT
  | BOOL
  | CHAR
  | STRING
  | ID
  | ID . ID
  | None
  | Just LPAREN EXPR RPAREN
  | [ ]
  | [ CONTENTS ]
  | LIST [ EXPR ]
  | LPAREN EXPR_1 :: EXPR_2 RPAREN
  | LPAREN EXPR_1 >|< EXPR_2 RPAREN
  | length LPAREN ID RPAREN
  | ( EXPR_1 , EXPR_2 )
  | front LPAREN EXPR RPAREN | back LPAREB EXPR RPAREN
  | lambda ID -> EXPR
  | ID LPAREN EXPRARGS RPAREN
  | EXPR_1 BINOP EXPR_2
  | UNOP EXPR
  | LPAREN if EXPR_1 then EXPR_2 else EXPR_3 RPAREN
  | CONSTRUCTOR | CONSTRUCTOR LPAREN EXPR RPAREN
  | left LPAREN EXPR RPAREN | right LPAREN EXPR RPAREN
  | match EXPR with
      CASES
    end

# Contents (of a list) consist of one or more expressions joined together.
CONTENTS ::=
  | EXPR
  | EXPR ; CONTENTS

# Cases for a pattern match; at least one case must be specified for a given match.
#   Evidently, matches cannot be used imperatively. This is a limitation of the language.
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
  | _ | [ ] | PAT_1 :: PAT_2 | () | BYTE | INT | LONG | FLOAT | BOOL | CHAR | ID | None
  | CONSTRUCTOR LPAREN PATTERN RPAREN | ( PAT_1, PAT_2 ) | Just LPAREN PATTERN RPAREN
  | Left LPAREN PATTERN RPAREN | Right LPAREN PATTERN RPAREN

# A binary operation is one of the following:
#   • Addition, subtraction, multiplication, division, or modular arithmetic
#   • High-multiplication for 64/128-bit support (**)
#   • A comparision (LT, GT, EQ, LE, GE, NE)
#   • String concatenation (^)
#   • Bit-shifting operations (sll=<<, srl=>>, sra=>>>)
#   • Logical or bitwise operations (and=&, or=|, xor=(+)), depending on the types used
#       as arguments (lots of type-checking necessary)
BINOP ::=
  | + | - | * | / | ** | < | > | = | <= | >= | \= | ^ | << | >> | >>> | & | "|" | (+) | mod

# A unary operation is either logical/bitwise negation (depending on type used)
#   or arithmetic negation.
UNOP ::=
  | - | not

# Arguments to a function are either empty or non-empty.
ARGS ::=
  |
  | NON-EMPTY-ARGS

# A non-empty argument is either a single argument or more than one argument.
NON-EMPTY-ARGS ::=
  | LPAREN ARG RPAREN
  | LPAREN ARG RPAREN NON-EMPTY-ARGS

# A standalone argument is an identifier, with or without a special type annotation.
ARG ::=
  | ID
  | ID : TYPE

# A type (for annotation purposes) is one of the following:
#   • A unit value
#   • A primitive type, like Byte, Int/Integer, Long, Float, Bool, Char, or String
#   • The unsigned versions of these aforementioned primitive types
#   • A pair of types
#   • A list, whose elements are homogeneously of one type
#   • A user-defined type/record/alias, specified by an identifier
#   • A function type (for lambda expressions captured by a variable)
#   • An option/maybe type (for safe null-ing checks)
#   • An either type, which can be one or the other between two types (nest-able)
#   • A polymorphic type (only up to 26, due to Greek limitations)
TYPE ::=
  | Unit
  | NUMTYPE
  | Unsigned NUMTYPE
  | Bool
  | Char
  | String
  | LPAREN TYPE_1 * TYPE_2 RPAREN
  | [ TYPE ]
  | ID
  | LPAREN TYPE_1 -> TYPE_2 RPAREN
  | Maybe TYPE | TYPE ?
  | Either < TYPE_1 , TYPE_2 >
  | 'CHAR

# Numerical types are kinds that represent actual numbers.
NUMTYPE ::=
  | Byte
  | Int | Integer
  | Long
  | Float

# An extended type is either a regular type or a special void type (return nothing).
EXTYPE ::=
  | Void
  | TYPE
```
