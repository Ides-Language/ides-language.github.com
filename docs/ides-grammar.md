---
layout: default
title: Ides Compiler Grammar
permalink: grammar.html
---


    (* non-terminals *)

    start = compound_expr ; (* start rule *)

    constant = BOOL_CONSTANT 
             | INT_CONSTANT 
             | DECIMAL_CONSTANT 
             | STRING_CONSTANT
             | CHAR_CONSTANT
             ;

    operator = OPERATOR
             | '='
             | "match"
             | "as"
             | "if"
             | "else"
             ;

    identifier = OPERATOR
               | IDENTIFIER
               ;

    name = identifier [ '[' arg_list ']' ] ;

    compound_expr = stmt [ ';' ] ;

    stmt = expr
         | fn_decl
         | record_decl
         | module_decl
         ;

    expr = infix_expr ;
    infix_expr = prefix_expr { operator prefix_expr } ;
    prefix_expr = { operator } postfix_expr

    postfix_expr = primary_expr { '(' expr_list ')'      (* value arguments *)
                                | '[' expr_list ']'      (* type arguments *)
                                | '{' compound_expr '}'  (* block expression *)
                                | '.' identifier
                                | "..."
                                } ;

    primary_expr = constant
                 | IDENTIFIER
                 | PLACEHOLDER
                 | val_decl
                 | '(' expr_list ')'      (* tuple value *)
                 | '{' compound_expr '}'  (* lexical scope *)
                 | partial_function
                 ;

    partial_function = '{' 
                           { "case" expr "=>" expr } 
                           [ "case" "else" "=>" expr ]
                       '}' ;

    fn_decl = qual "def" name '(' expr_list ')' ( [ ':' expr ] "=>" expr
                                                | [ ':' expr ] ';'
                                                );

    record_decl = qual ( "struct" 
                       | "class" 
                       | "trait" ) name [ '(' arg_list ')' ] [ ':' expr_list ] "=>" '{' compound_expr '}';

    module_decl = qual "mod" name '{' compound_expr '}' ;

    (* expr, expr, ... expr *)
    expr_list = [ expr { ',' expr } ] ;

    (* v1 : T1, v2 : T2, ... vN : TN *)
    arg_list = [ IDENTIFIER ':' expr { ',' IDENTIFIER ':' expr } ] ;

    val_decl = qual ( "val" | "var" ) name ( ':' expr [ "=>" expr ]
                                           | '=' expr
                                           ) ;


    (* qualifiers (order subject to change) *)
    qual = vis ["abstract"] ["const"] ["extern"] ["locked"] ;
    vis = "public" | "private" | "protected" | "internal" ;



    (* terminal symbols *)

    OP_FIRST = "[!\#\%\^&\*\-\+/\\<>\|?]"
    OPERATOR = "{OP_FIRST}[=:{OP_FIRST}]*" ;
    ID = "[A-Za-z_][A-Za-z0-9]*" ; (* excluding any literals in non-terminal rules *)

    IDENTIFIER = {ID}
               | "`[^`]+`"
               | "{ID}_{OPERATOR}"
               ;

    BOOL_CONSTANT = "true" | "false" ;

    INT_CONSTANT = "[1-9][0-9]*" (* base-10 integer constant *)
                 | "0b[01]+"     (* base-2 integer constant *)
                 | "0[0-9]+"     (* base-8 integer constant *)
                 | "0x[0-9A-F]+" (* base-16 integer constant *)

    DECIMAL_CONSTANT = "[0-9]+\.[0-9]+" ; (* decimal constant *)
                 ;

    STRING_CONSTANT = ".+" ;

    CHAR_CONSTANT = '.' ;
