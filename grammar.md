# Grammar

The full EBNF of RSCP, followed by the semicolon-insertion rules.

```ebnf
program        := statement* EOF ;

statement      := letStmt | fnStmt | classStmt | enumDecl | ifStmt | whileStmt
                | loopStmt | forStmt | tryStmt | throwStmt | importStmt
                | returnStmt | breakStmt | continueStmt
                | block | assignment | incDec | exprStmt | ";" ;

letStmt        := ("let" | "var") IDENT "=" expression [";"] ;
fnStmt         := "fn" IDENT funcBody ;
classStmt      := "class" IDENT [ "(" IDENT ")" ]
                  "{" { classItem } "}" ;
classItem      := fieldList | "fn" IDENT funcBody ;
fieldList      := IDENT { "," IDENT } [ ";" ] ;   (* 26.0.12.6 *)
funcBody       := "(" [ paramList ] ")" ( block | "->" expression ) ;
paramList      := IDENT { "," IDENT } [ "," ] ;   (* trailing comma ok *)

body           := block | statement ;  (* braces, or one statement *)
ifStmt         := "if" expression body
                  [ "else" ( ifStmt | body ) ] ;
whileStmt      := "while" expression body ;
loopStmt       := "loop" body ;
forStmt        := "for" ( forHead ")" block                       (* legacy *)
                            | ["let"|"var"] IDENT "in" expression block ) ;
forHead        := "(" ( forInit? ";" expression? ";" forStep?
                     | ["let"|"var"] IDENT "in" expression ) ;
forInit        := letStmt' | assignment' | exprStmt' ;   (* no trailing ';' *)
forStep        := assignment' | incDec' | exprStmt' ;
returnStmt     := "return" [ expression ] [";"] ;
breakStmt      := "break" [";"] ;
continueStmt   := "continue" [";"] ;
enumDecl       := "enum" IDENT "{" { IDENT [ "(" paramList ")" ] [ "," | ";" ] } "}" ;
tryStmt        := "try" block { catchClause }+ ;
catchClause    := "catch" [ IDENT ] [ "code" STRING | "where" expression ] block ;
importStmt     := "import" STRING [ "as" IDENT ] ;
throwStmt      := "throw" expression [";"] ;
block          := "{" ( statement | ";" )* "}" ;

assignment     := target ( "=" | "+=" | "-=" | "*=" | "/=" | "%=" ) expression [";"] ;
incDec         := target ( "++" | "--" ) [";"] ;
target         := IDENT | postfix "[" expression "]" ;
exprStmt       := expression [";"] ;

expression     := ternary ;
ternary        := range [ "?" expression ":" ternary ] ;
range          := logicOr [ ( ".." | "..=" ) logicOr ] ; (* for iterables only *)
logicOr        := logicAnd { ( "||" | "or" ) logicAnd } ;
logicAnd       := bitOr { ( "&&" | "and" ) bitOr } ;
bitOr          := bitXor { "|" bitXor } ;
bitXor         := bitAnd { "^" bitAnd } ;
bitAnd         := equality { "&" equality } ;
equality       := comparison { ( "==" | "!=" ) comparison } ;
comparison     := shift { ( "<" | "<=" | ">" | ">=" | "instanceof" ) shift } ;
shift          := term { ( "<<" | ">>" ) term } ;
term           := factor { ( "+" | "-" ) factor } ;
factor         := unary { ( "*" | "/" | "//" | "%" ) unary } ;
unary          := ( "!" | "not" | "-" | "~" ) unary | power ;
power          := postfix [ "**" unary ] ;   (* right-assoc, tighter than unary *)
postfix        := primary { "(" [ arguments ] ")" | "[" expression "]" | "." IDENT }
                | "super" "." IDENT "(" [ arguments ] ")" ;   (* inside methods *)
arguments      := expression { "," expression } [ "," ] ;
primary        := INT | FLOAT | STRING | FSTRING | "true" | "false" | "null" | IDENT
                | "(" expression ")" | arrayLit | mapLit | "fn" funcBody
                | matchExpr | closure ;
FSTRING        := 'f"' { litChar | "{{" | "}}" | "{" expression "}" } '"' ;
matchExpr      := "match" expression "{" arm { ( "," | ";" )* arm } [ "," ] "}" ;
arm            := pattern { "|" pattern } [ "if" expression ] "=>"
                  ( expression | block ) ;
pattern        := IDENT "." IDENT [ "(" [ variantField { "," variantField } ] ")" ]
                | INT [ ( ".." | "..=" ) INT ] | FLOAT | STRING
                | "true" | "false" | "null" | IDENT | "else" ;  (* "_" is IDENT *)
variantField   := INT | FLOAT | STRING | "true" | "false" | "null"
                | IDENT ;   (* binding or "_"; literals match the payload *)
closure        := ( "|" paramList "|" | "||" | IDENT "=>"
                   | "(" paramList ")" "=>" ) ( expression | block ) ;
arrayLit       := "[" [ expression { "," expression } [ "," ] ] "]" ;
mapLit         := "{" [ expression ":" expression { "," … } [ "," ] ] "}" ;
```

## Semicolon insertion

A `;` is inserted automatically after a newline (or the end of input) when
the line ends with a value, `)`, `]`, `}`, `++`, `--`, `return`, `break` or
`continue` — and the next token cannot continue an expression (a `}`, `]`,
`)`, `,`, `:`, `?`, `else`, `->`, `=>`, `..` suppresses the insertion). A `{`
at statement position always starts a block; a map literal in that position
must be parenthesized: `var m = {"a": 1};` works, a bare `{"a": 1}` does not
(same rule as JavaScript).
