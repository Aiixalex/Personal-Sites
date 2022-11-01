---
title: Decaf Compiler
date: 2021-04-10
math: true
diagram: true
authors:
- admin

---

Compiler of C-like language Decaf.  [***Github Link***](https://github.com/Aiixalex/Decaf-Compiler)

Here is an example Decaf program:

```
extern func print_int(int) void;

package GreatestCommonDivisor {
    var a int = 10;
    var b int = 20;

    func main() int {
        var x, y, z int;
        x = a;
        y = b;
        z = gcd(x, y);

        // print_int is part of the standard input-output library
        print_int(z);
    }

    // function that computes the greatest common divisor
    func gcd(a int, b int) int {
        if (b == 0) { return(a); }
        else { return( gcd(b, a % b) ); }
    }
}
```

Usage
-----------------

To create the default program, go to the answer directory and type in `make default`. To run the default program against the testcases, run the following commands:

```shell
$ python zipout.py -r default
$ python check.py
```

The output files are saved to the `output` directory while all the intermediate LLVM files created for each input Decaf program are saved to the `llvm` directory.

Directory `dev_llvm` contains sample output LLVM assembly for each Decaf program in `testcases/dev`. You can check that your output LLVM assembly is roughly doing the right thing by comparing your output to this sample output.

Here are several shell scripts that helps streamline parts of the process.

| Attempt     | description                                                  | args                         | output                                                       |
| ----------- | ------------------------------------------------------------ | ---------------------------- | ------------------------------------------------------------ |
| build.sh    | Builds the project and runs against testcases in `testcases` directory. | 0                            | None                                                         |
| evaluate.sh | Compares all the project's output files in `output` directory against the given references in `reference` directory to find any failing tests, and where the difference occurs. | 0                            | None if all outputs are correct, file name and position of difference if failure occurred. |
| result.sh   | Provides outputs for user to compare their output against the reference output, as well as the used test case. | file names without extension | Provides the project's original test case, reference output, and the project's output. |

Several use-case examples of the scripts are in `README.md` in directory `scripts`.

Program
---------------

* `makefile`: contains the necessary recipes for building LLVM assembly code, C++ code using LLVM API calls and Lex/Yacc programs that use the LLVM API.
* `decaf-stdlib.c`: the Decaf standard library. Contains the extern functions used commonly in Decaf programs.
* Solution files:
    * `default-defs.h`: the common header file among all the default files
    * `decafcomp.cc`: classes for LLVM code generation
    * `decafcomp.lex`: the lexer for Decaf
    * `decafcomp.y`: the yacc program for a small fragment of Decaf which uses `decafcomp.cc` for LLVM code generation.
* `llvm-run`: the Python program used by `check.py` in order to run a Decaf program using the following steps. Each step assumes some file names but can be changed using command line options so run `llvm-run -h` to see the options and also read the source code of `llvm-run` to understand what it is doing.  Stages are:
    * llvm: source code to LLVM code generation
    * bc: assembly to LLVM bitcode
    * s: bitcode to native code
    * exec: linking to make native executable
    * run: running the final executable

## Implemented Grammar

### Lexer

The lexical analyzer produces a stream of tokens for a given Decaf program. The input is taken from `stdin` (standard input) and the output token stream is sent to `stdout` (standard output). Errors are issued on the `stderr` (standard error) stream.

The lexical analyzer produces the following token stream:

```
T_AND            &&
T_ASSIGN         =
T_BOOLTYPE       bool
T_BREAK          break
T_CHARCONSTANT   char_lit (see section on Character literals)
T_COMMA          ,
T_COMMENT        comment
T_CONTINUE       continue
T_DIV            /
T_DOT            .
T_ELSE           else
T_EQ             ==
T_EXTERN         extern
T_FALSE          false
T_FOR            for
T_FUNC           func
T_GEQ            >=
T_GT             >
T_ID             identifier (see section on Identifiers)
T_IF             if
T_INTCONSTANT    int_lit (see section on Integer literals)
T_INTTYPE        int
T_LCB            {
T_LEFTSHIFT      <<
T_LEQ            <=
T_LPAREN         (
T_LSB            [
T_LT             <
T_MINUS          -
T_MOD            %
T_MULT           *
T_NEQ            !=
T_NOT            !
T_NULL           null
T_OR             ||
T_PACKAGE        package
T_PLUS           +
T_RCB            }
T_RETURN         return
T_RIGHTSHIFT     >>
T_RPAREN         )
T_RSB            ]
T_SEMICOLON      ;
T_STRINGCONSTANT string_lit (see section on String literals)
T_STRINGTYPE     string
T_TRUE           true
T_VAR            var
T_VOID           void
T_WHILE          while
T_WHITESPACE     whitespace (see section on Whitespace)
```

### Parser

The parser of the compiler produces an abstract syntax tree for valid Decaf programs.

An abstract syntax tree (AST) is a high-level representation of the program structure without the necessity of containing all the details in the source code; it can be thought of as an abstract representation of the source code.

The specification for the abstract syntax tree to be produced by your program is below:

```asdl
module Decaf
{
    prog = Program(extern* extern_list, package body)

    extern = ExternFunction(identifier name, method_type return_type, extern_type* typelist)

    decaf_type = IntType | BoolType

    method_type = VoidType | decaf_type

    extern_type  = VarDef(StringType) | VarDef(decaf_type)

    package = Package(identifier name, field_decl* field_list, method_decl* method_list)

    field_decl = FieldDecl(identifier name, decaf_type type, field_size size)
        | AssignGlobalVar(identifier name, decaf_type type, constant value)

    field_size = Scalar | Array(int array_size)

    method_decl = Method(identifier name, method_type return_type, typed_symbol* param_list, method_block block)

    typed_symbol = VarDef(identifier name, decaf_type type)

    method_block = MethodBlock(typed_symbol* var_decl_list, statement* statement_list)

    block = Block(typed_symbol* var_decl_list, statement* statement_list)

    statement = assign
        | method_call
        | IfStmt(expr condition, block if_block, block? else_block)
        | WhileStmt(expr condition, block while_block)
        | ForStmt(assign* pre_assign_list, expr condition, assign* loop_assign_list, block for_block)
        | ReturnStmt(expr? return_value)
        | BreakStmt
        | ContinueStmt
        | block

    assign = AssignVar(identifier name, expr value)
        | AssignArrayLoc(identifier name, expr index, expr value)

    method_call = MethodCall(identifier name, method_arg* method_arg_list)

    method_arg = StringConstant(string value)
        | expr

    expr = rvalue
        | method_call
        | constant
        | BinaryExpr(binary_operator op, expr left_value, expr right_value)
        | UnaryExpr(unary_operator op, expr value)

    constant = NumberExpr(int value)
        | BoolExpr(bool value)

    rvalue = VariableExpr(identifier name)
        | ArrayLocExpr(identifier name, expr index)

    bool = True | False

    binary_operator = Plus | Minus | Mult | Div | Leftshift | Rightshift | Mod | Lt | Gt | Leq | Geq | Eq | Neq | And | Or

    unary_operator = UnaryMinus | Not
}
```

The entire set of rules that describe the `Decaf` grammar specification is below:

```
Program = Externs package identifier "{" FieldDecls MethodDecls "}" .
Externs    = { ExternDefn } .
ExternDefn = extern func identifier "(" [ { ExternType }+, ] ")" MethodType ";" .
FieldDecls = { FieldDecl } .
FieldDecl  = var { identifier }+, Type ";" .
FieldDecl  = var { identifier }+, ArrayType ";" .
FieldDecl  = var identifier Type "=" Constant ";" .
MethodDecls = { MethodDecl } .
MethodDecl  = func identifier "(" [ { identifier Type }+, ] ")" MethodType Block .
Block = "{" VarDecls Statements "}" .
VarDecls = { VarDecl } .
VarDecl  = var { identifier }+, Type ";" .
Statements = { Statement } .
Statement = Block .
Statement = Assign ";" .
Assign    = Lvalue "=" Expr .
Lvalue    = identifier | identifier "[" Expr "]" .
Statement  = MethodCall ";" .
MethodCall = identifier "(" [ { MethodArg }+, ] ")" .
MethodArg  = Expr | string_lit .
Statement = if "(" Expr ")" Block [ else Block ] .
Statement =  while "(" Expr ")" Block .
Statement = for "(" { Assign }+, ";" Expr ";" { Assign }+, ")" Block .
Statement = return [ "(" [ Expr ] ")" ] ";" .
Statement = break ";" .
Statement = continue ";" .
Expr = identifier .
Expr = MethodCall .
Expr = Constant .
UnaryOperator = ( UnaryNot | UnaryMinus ) .
UnaryNot = "!" .
UnaryMinus = "-" .
BinaryOperator = ( ArithmeticOperator | BooleanOperator ) .
ArithmeticOperator = ( "+" | "-" | "*" | "/" | "<<" | ">>" | "%" ) .
BooleanOperator = ( "==" | "!=" | "<" | "<=" | ">" | ">=" | "&&" | "||" ) .
Expr = Expr BinaryOperator Expr .
Expr = UnaryOperator Expr .
Expr = "(" Expr ")" .
Expr = identifier "[" Expr "]" .
ExternType = ( string | Type ) .
Type = ( int | bool ) .
MethodType = ( void | Type ) .
BoolConstant = ( true | false ) .
ArrayType = "[" int_lit "]" Type .
Constant = ( int_lit | char_lit | BoolConstant ) .
```

### Code Generation

Extend the abstract syntax tree (AST) classes to add LLVM API calls for code generation.

### Others

#### Control flow and Loops: 

Support for control flow (`if` statements) and loops (`while` and `for` statements) is added, including support for `else` blocks as well as `break` and `continue` statements.

To complete control flow and loops, I implemented backpatching using the symbol table to mark the entry, continue and exit points for the control flow and loops.

#### Short Circuit

[Short-circuit evaluation](https://en.wikipedia.org/wiki/Short-circuit_evaluation) for boolean expressions is implemented.

#### Semantics

All the semantic checks listed in the Decaf Semantics section of the [**Decaf spec**](http://anoopsarkar.github.io/compilers-class/decafspec.html) are implemented.

