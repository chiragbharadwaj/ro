# Ro
This is compiler for `Ro`, an imperative programming language with functional features created by Chirag Bharadwaj and Peter Li at Cornell University. The compiler, which will eventually be written in `OCaml` (a functional language with near-`C` performance), will compile to the `x86-64` assembly language.          

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

Upcoming Features
----
We will update this list with upcoming features in future builds as we get them up and rolling.

* Adding support for comments.
  + That is, in-line comments as well as out-of-line block-style comments.
* Adding richer type-declarations.
* Adding support for interface files, namely those with the `.iro` extension.

Privacy Policy
----

Copyright &copy; 2015-2016 Chirag Bharadwaj and Peter Li, Cornell University.

There is NO warranty. This software may be freely redistributed with attribution to the original author in all cases. This software may also be modified from its original version and redistributed as long as all changes made are stated very clearly in the redistributed version and attribution to the original author is provided. You may not use ANY part of this software for commercial purposes. All rights reserved. Please report any bugs to the development team by making a pull request to this repository.
