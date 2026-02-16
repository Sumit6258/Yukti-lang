# Yukti Language Grammar Specification

## Lexical Structure

### Keywords
```
prakriya     sthir        parivartan   yadi         anyatha
chakra       pratyek      milaan       sangrah      prakar
vishesha     karma        pratiksha    pavitra      antardhyan
sarvajanik   aayaat       nishkaasan   vapasi       toro
jari         upaaya       swayam       Swayam       videshi
```

### Operators
```
Arithmetic:  +  -  *  /  %  **
Comparison:  ==  !=  <  >  <=  >=
Logical:     &&  ||  !
Bitwise:     &  |  ^  <<  >>  ~
Assignment:  =  +=  -=  *=  /=  %=  &=  |=  ^=  <<=  >>=
Other:       ->  =>  ::  .  ,  ;  :  ?
Range:       ..  ..=
Reference:   &  &parivartan
Dereference: *
```

### Literals
```
Integer:     42  0xFF  0o77  0b1010  1_000_000
Float:       3.14  1.0e-5  2.5E+10
String:      "hello"  "नमस्ते"
Character:   'a'  'अ'  '\n'  '\u{0905}'
Boolean:     satya  asatya
```

### Identifiers
```
identifier ::= [a-zA-Z_][a-zA-Z0-9_]*
            | [अ-औ][अ-औ0-9]*  // Devanagari identifiers
```

### Comments
```
// Single line comment
/* Multi-line
   comment */
/// Documentation comment
```

## Syntax Grammar (EBNF)

### Program Structure
```ebnf
program ::= item*

item ::= function
       | structure
       | enumeration
       | trait
       | implementation
       | constant
       | import
       | export

import ::= "aayaat" path ("::*" | "::" "{" identifier_list "}")?

export ::= "nishkaasan" item
```

### Functions
```ebnf
function ::= visibility? purity? async? "prakriya" identifier 
             generic_params? "(" parameter_list? ")" return_type? block

purity ::= "pavitra"

async ::= "karma"

visibility ::= "sarvajanik" | "antardhyan"

parameter_list ::= parameter ("," parameter)*

parameter ::= pattern ":" type

return_type ::= "->" type

generic_params ::= "<" generic_param_list ">"

generic_param_list ::= identifier ("," identifier)*
```

### Types
```ebnf
type ::= primitive_type
       | reference_type
       | array_type
       | tuple_type
       | function_type
       | path_type
       | generic_type

primitive_type ::= "Shunya" | "Satya" 
                 | "Ank8" | "Ank16" | "Ank32" | "Ank64"
                 | "Uank8" | "Uank16" | "Uank32" | "Uank64"
                 | "Dashamlav32" | "Dashamlav64"
                 | "Akshar" | "Shabd"

reference_type ::= "&" "parivartan"? lifetime? type

array_type ::= "[" type ";" expression "]"
             | "[" type "]"

tuple_type ::= "(" type_list ")"

function_type ::= "prakriya" "(" type_list? ")" return_type?

path_type ::= identifier ("::" identifier)*

generic_type ::= path_type "<" type_list ">"

type_list ::= type ("," type)*

lifetime ::= "'" identifier
```

### Structures
```ebnf
structure ::= visibility? "sangrah" identifier generic_params? 
              "{" field_list? "}"

field_list ::= field ("," field)* ","?

field ::= visibility? identifier ":" type
```

### Enumerations
```ebnf
enumeration ::= visibility? "prakar" identifier generic_params?
                "{" variant_list? "}"

variant_list ::= variant ("," variant)* ","?

variant ::= identifier
          | identifier "(" type_list ")"
          | identifier "{" field_list "}"
```

### Traits
```ebnf
trait ::= visibility? "vishesha" identifier generic_params?
          "{" trait_item* "}"

trait_item ::= function_signature
             | associated_type

function_signature ::= "prakriya" identifier generic_params?
                       "(" parameter_list? ")" return_type? ";"

associated_type ::= "prakar" identifier ";"
```

### Implementations
```ebnf
implementation ::= "upaaya" type_path for_clause? 
                   generic_params? where_clause? "{" impl_item* "}"

for_clause ::= "prakar" type_path

impl_item ::= function
            | associated_type_def

associated_type_def ::= "prakar" identifier "=" type ";"

where_clause ::= "jahan" where_predicate_list

where_predicate_list ::= where_predicate ("," where_predicate)*

where_predicate ::= type ":" trait_bound_list

trait_bound_list ::= trait_bound ("+" trait_bound)*

trait_bound ::= lifetime | type_path
```

### Statements
```ebnf
statement ::= expression_statement
            | declaration_statement
            | item

expression_statement ::= expression ";"

declaration_statement ::= let_statement

let_statement ::= ("sthir" | "parivartan") pattern (":" type)? 
                  ("=" expression)? ";"
```

### Expressions
```ebnf
expression ::= literal_expression
             | path_expression
             | operator_expression
             | grouped_expression
             | array_expression
             | tuple_expression
             | struct_expression
             | call_expression
             | method_call_expression
             | field_expression
             | index_expression
             | if_expression
             | match_expression
             | loop_expression
             | return_expression
             | break_expression
             | continue_expression
             | block_expression
             | lambda_expression
             | async_expression
             | await_expression

literal_expression ::= integer_literal
                     | float_literal
                     | string_literal
                     | char_literal
                     | boolean_literal

boolean_literal ::= "satya" | "asatya"

path_expression ::= path

operator_expression ::= expression binary_op expression
                      | unary_op expression

binary_op ::= "+" | "-" | "*" | "/" | "%" | "**"
            | "==" | "!=" | "<" | ">" | "<=" | ">="
            | "&&" | "||"
            | "&" | "|" | "^" | "<<" | ">>"
            | ".." | "..="
            | "="
            | "+=" | "-=" | "*=" | "/=" | "%="
            | "&=" | "|=" | "^=" | "<<=" | ">>="

unary_op ::= "-" | "!" | "&" | "&parivartan" | "*"

grouped_expression ::= "(" expression ")"

array_expression ::= "[" expression_list "]"
                   | "[" expression ";" expression "]"

tuple_expression ::= "(" expression_list ")"

struct_expression ::= path_expression "{" field_init_list "}"

field_init_list ::= field_init ("," field_init)* ","?

field_init ::= identifier
             | identifier ":" expression

call_expression ::= expression "(" expression_list? ")"

method_call_expression ::= expression "." identifier 
                          "(" expression_list? ")"

field_expression ::= expression "." identifier

index_expression ::= expression "[" expression "]"

expression_list ::= expression ("," expression)*
```

### Control Flow
```ebnf
if_expression ::= "yadi" expression block
                  ("anyatha" "yadi" expression block)*
                  ("anyatha" block)?

match_expression ::= "milaan" expression "{" match_arm* "}"

match_arm ::= pattern match_guard? "=>" expression ","

match_guard ::= "yadi" expression

loop_expression ::= infinite_loop
                  | while_loop
                  | for_loop

infinite_loop ::= "chakra" block

while_loop ::= "chakra" expression block

for_loop ::= "pratyek" pattern "in" expression block

return_expression ::= "vapasi" expression?

break_expression ::= "toro" expression?

continue_expression ::= "jari"

block_expression ::= "{" statement* expression? "}"

block ::= "{" statement* "}"
```

### Patterns
```ebnf
pattern ::= literal_pattern
          | identifier_pattern
          | wildcard_pattern
          | rest_pattern
          | reference_pattern
          | struct_pattern
          | tuple_struct_pattern
          | tuple_pattern
          | slice_pattern
          | path_pattern
          | or_pattern
          | range_pattern

literal_pattern ::= literal_expression

identifier_pattern ::= "parivartan"? identifier

wildcard_pattern ::= "_"

rest_pattern ::= ".."

reference_pattern ::= "&" "parivartan"? pattern

struct_pattern ::= path_expression "{" field_pattern_list? "}"

field_pattern_list ::= field_pattern ("," field_pattern)* ","?

field_pattern ::= identifier
                | identifier ":" pattern
                | ".."

tuple_struct_pattern ::= path_expression "(" pattern_list? ")"

tuple_pattern ::= "(" pattern_list? ")"

slice_pattern ::= "[" pattern_list? "]"

pattern_list ::= pattern ("," pattern)*

path_pattern ::= path_expression

or_pattern ::= pattern "|" pattern

range_pattern ::= range_start? ".." range_end?
                | range_start? "..=" range_end

range_start ::= literal_expression | path_expression

range_end ::= literal_expression | path_expression
```

### Async/Await
```ebnf
async_expression ::= "karma" block

await_expression ::= "pratiksha" expression
```

### Lambda Expressions
```ebnf
lambda_expression ::= "|" parameter_list? "|" (expression | block)
```

### Path
```ebnf
path ::= path_segment ("::" path_segment)*

path_segment ::= identifier generic_args?

generic_args ::= "<" generic_arg_list ">"

generic_arg_list ::= generic_arg ("," generic_arg)*

generic_arg ::= type | lifetime | literal_expression
```

## Operator Precedence (Highest to Lowest)

1. Path resolution `::`
2. Method call `.`
3. Field access `.`
4. Function call, array indexing `()` `[]`
5. Unary `-` `!` `&` `*`
6. Type cast `as`
7. Multiplication, division, modulo `*` `/` `%`
8. Addition, subtraction `+` `-`
9. Bit shift `<<` `>>`
10. Bitwise AND `&`
11. Bitwise XOR `^`
12. Bitwise OR `|`
13. Comparison `==` `!=` `<` `>` `<=` `>=`
14. Logical AND `&&`
15. Logical OR `||`
16. Range `..` `..=`
17. Assignment `=` `+=` `-=` etc.

## Type System Rules

### Ownership Rules
1. Each value has exactly one owner
2. When the owner goes out of scope, the value is dropped
3. Values can be moved or borrowed

### Borrowing Rules
1. At any given time, you can have either:
   - One mutable reference (&parivartan T)
   - Any number of immutable references (&T)
2. References must always be valid

### Lifetime Rules
1. Every reference has a lifetime
2. Lifetime parameters connect the lifetimes of various values
3. The borrow checker ensures all references are valid

## Type Inference
The compiler performs Hindley-Milner type inference with extensions for:
- Subtyping
- Higher-rank types
- Associated types
- Trait bounds

## Name Resolution
1. Local variables shadow outer scopes
2. Items are resolved through module system
3. Trait methods require trait to be in scope
4. Use declarations bring names into scope

## Memory Model
- Stack allocation by default
- Heap allocation via Box<T>, Suchi<T>, etc.
- No garbage collection
- Deterministic destruction (RAII)
- Move semantics by default
- Copy semantics for simple types

---

This grammar provides the formal specification for parsing Yukti code
and forms the basis for compiler implementation.
