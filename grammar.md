Copyright (c) the Selfie Project authors. All rights reserved. Please see the AUTHORS file for details. Use of this source code is governed by a BSD license that can be found in the LICENSE file.

Selfie is a project of the Computational Systems Group at the Department of Computer Sciences of the University of Salzburg in Austria. For further information and code please refer to:

selfie.cs.uni-salzburg.at

This is the grammar of the C Star (C\*) programming language.

C\* is a tiny subset of the programming language C. C\* features global variable declarations with optional initialization as well as procedures with parameters and local variables. C\* has six statements (assignment, while loop, for loop, if-then-else, procedure call, and return) and standard arithmetic (`+`, `-`, `*`, `/`, `%`), comparison (`==`, `!=`, `<`, `>`, `<=`, `>=`), bitwise (`&`, `|`, `~`, `<<`, `>>`), and Boolean (`&&`, `||`, `!`) operators over variables and procedure calls as well as integer, character, and string literals. C\* includes the unary `*` operator for dereferencing pointers hence the name but excludes data types other than `uint64_t` and `uint64_t*` and many other features. The C\* grammar is LL(1) with 8 keywords and 25 symbols. Whitespace as well as single-line (`//`) and multi-line (`/*` to `*/`) comments are ignored.

C\* Keywords: `uint64_t`, `void`, `sizeof`, `if`, `else`, `while`, `for`, `return`

C\* Symbols: `integer`, `character`, `string`, `identifier`, `,`, `;`, `(`, `)`, `{`, `}`, `+`, `-`, `*`, `/`, `%`, `=`, `==`, `!=`, `<`, `>`, `<=`, `>=`, `<<`, `>>`, `&`, `&&`, `|`, `||`, `~`, `!`, `...`

with:

```
integer    = digit { digit } | "0" ( "x" | "X" ) hexdigit { hexdigit } .

character  = "'" printable_character "'" .

string     = """ { printable_character } """ .

identifier = letter { letter | digit | "_" } .
```

and:

```
digit    = "0" | ... | "9" .

hexdigit = digit | "a" | ... | "f" | "A" | ... | "F" .

letter   = "a" | ... | "z" | "A" | ... | "Z" .
```

C\* Grammar:

```
cstar      = { variable [ initialize ] ";" | procedure } .

variable   = type identifier .

type       = "uint64_t" [ "*" ] .

initialize = "=" [ cast ] [ "-" ] value .

cast       = "(" type ")" .

value      = integer | character .

statement  = assignment ";" | if | while | call ";" | return ";" .

assignment = ( [ "*" ] identifier | "*" "(" expression ")" ) "=" expression .

expression = logicalor .

logicalor  = logicaland { "||" logicaland } .

logicaland = comparison { "&&" comparison } .

comparison = bitwiseor [ ( "==" | "!=" | "<" | ">" | "<=" | ">=" ) bitwiseor ] .

bitwiseor  = bitwiseand { "|" bitwiseand } .

bitwiseand = shift { "&" shift } .

shift      = arithmetic { ( "<<" | ">>" ) arithmetic } .

arithmetic = term { ( "+" | "-" ) term } .

term       = factor { ( "*" | "/" | "%" ) factor } .

factor     = [ cast ] [ "-" ] [ "~" ] [ "!" ] [ "*" ]
             ( "sizeof" "(" type ")" | literal | identifier | call | "(" expression ")" ) .

literal    = value | string .

if         = "if" "(" expression ")"
               ( statement | "{" { statement } "}" )
             [ "else"
               ( statement | "{" { statement } "}" ) ] .

while      = "while" "(" expression ")"
               ( statement | "{" { statement } "}" ) .

for        = "for" "(" assignment ";" expression ";" assignment ")"
               ( statement | "{" { statement } "}" ) .

procedure  = ( type | "void" ) identifier "(" [ variable { "," variable } [ "," "..." ] ] ")"
             ( ";" | "{" { variable ";" } { statement } "}" ) .

call       = identifier "(" [ expression { "," expression } ] ")" .

return     = "return" [ expression ] .
```