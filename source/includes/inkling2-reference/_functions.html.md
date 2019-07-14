# Functions

Inkling is not a general-purpose programming language, but it does support a few simple functional programming capabilities, including:

* Functions with typed input and output parameters
* Constants and variables
* Conditional (if/else) statements
* Standard mathematical and logical operators
* Built-in library functions for mathematical operations

Inkling does _not_ support the following:

* Statefulness (values that persist across function invocations)
* Loop constructs
* Recursion
* Exceptions

These limitations make more sense when considering that programming logic defined in Inkling is meant to be executed in the context of data streams that flow through the concept graph.

### Function Declarations

Functions are declared using a `function` statement followed by the function’s name, a list of zero or more typed input parameters, and an optional output parameter type. The function name can be omitted for inlined functions but must be present for functions defined in the program’s global namespace.

```inkling2--syntax
functionDeclaration :==
  function [<functionName>] '(' [<paramName>: <paramType>, ]* ')' [':' <returnType>] '{'
    [functionStatement]*
  '};

functionStatement :==
  variableDeclaration |
  variableAssignment |
  conditionalStatement |
  returnStatement
```   

All functions in Inkling must return a value, but the type of this value can be inferred by the compiler.

For more details about the Inkling type system, refer to the [types documentation][1].

Here are a few examples of function declarations:

```inkling2--code
function RadiansToDegress(Radians: number) {
  # Return type is inferred to be 'number'.
  return Radians * 180 / Math.Pi
}

function GetReward(State: SimState): number {
  if IsTerminal(State) {
    return -10.0
  } else {
    return Math.Abs(State.Position - 10) / 10.0
  }
}
```

#### Types & Implicit Casts
The inkling compiler will validate that all values assigned to variables or input parameters and return parameters are of a compatible type. Note that compatibility doesn’s require an exact type match. Consider the following examples:

```inkling2--code
# This is allowed because the value of the
# expression fits within the constrained type.
var OneOrTWo: number<1, 2> = 1 + 1

# This is allowed because the type of "OneOrTWo" always
# fits within the range 0..100.
var ZeroTo100: number<0..100> = OneOrTWo

# This will generate a compiler error because "ZeroTo100"
# may not match the values of the enumerated
# type of "OneOrTWo".
OneOrTWo = ZeroTo100

var ZeroTo50: number<0..50 step 1> = 50

# This will generate a compiler warning because "ZeroTo50"
# may require clipping to fit within the specified range.
ZeroTo50 = ZeroTo100
```

For more details about compatible types, refer to the [types documentation][1].


#### Runtime Errors & Debugging
The Inkling compiler attempts to detect and report most errors statically. However, there are certain circumstances where runtime may occur. Examples include undefined mathematical operations like division by zero. Runtime errors are logged and reported during training and may terminate a training session.


### Function Statements

#### Variable Declarations

Variables can be declared within a function. All variables have function-level scope even if they are declared within a sub-scope such as an if/else statement. Each variable within a function must have a unique name and a type. The type can be omitted if it can be inferred from an initial value assigned to the variable.

```inkling2--syntax
variableDeclaration :==
  var <varName> [':' <varType>] ['=' <expression>]
```

A variable that is not assigned an initial value must be assigned a value along all code paths before it is used.

```inkling2--code
# Variable with no initial assignment.
var TwoOrThree: number<2, 3>

if IsCondition() {
  TwoOrThree = 2
}

# This will generate a compiler error because the
# variable may not have a defined value.
return TwoOrThree
```

#### Variable Assignments
A previously-declared variable can be assigned a new value as long as that value matches the type of the variable.

```inkling2--syntax
variableAssignment :==
  <varName> '=' <expression>
```

#### Conditional Execution
Inkling supports if/else statements, which can be chained together with “else if” clauses. The braces that surround the code blocks within an `if` or `else` clause are required.

```inkling2--syntax
conditionalStatement :==
  if <expression> '{'
    [functionStatement]*
  '}' [else if <expression> '{'
    [functionStatement]*
  '}']* [else '{'
    [functionStatement]*
  '}']
```   

#### Return Statements
A return statement terminates execution of the function and returns the specified value to its caller. The type of the expression must match the declared return type of the function.

```inkling2--syntax
returnStatement :==
  return <expression>
```

### Expressions
Inkling supports basic expressions and a variety of unary and binary operators.

#### Built-in Operators
| Operation        | Syntax        | Description |
| -                | -             | -           |
| Function call    | X()           | Invokes a function with zero or more arguments |
| Array index      | X[Y]          | Indexes into an array |
| Member access    | X.Y           | References a field within a structure |
| Neg/pos          | -, +          | Negate numeric value |
| Logical not      | not           | Logical not (produces 0 or 1) |
| Exponent         | **            | X raised to the Y power |
| Mult/div/mod     | *, /, %       | X times Y, X divided by Y, X mod Y |
| Add/subtract     | +, -          | X plus Y, X minus Y |
| Relational       | <, <=, >, >=  | Less, less-or-equal, greater, greater-or-equal |
| Equality         | ==, !=        | Equal, not equal |
| Logical and      | and           | Left expression if truthy, else right expression |
| Logical or       | or            | Left expression if falsy, else right expression |


As in algebra, parentheses can be used to make order of evaluation explicit. If parentheses are omitted, the table above defines the order of operations.

All mathematical, logical and relational operators work only with numerical operands. Operands of other types (strings, arrays, structures, etc.) are flagged as errors by the compiler.


### Function Calls
A function call temporarily suspends execution of the current function and resumes execution temporarily in the context of a new function. The caller may specify zero or more arguments to be passed to the function. The types of these arguments must be compatible with the declared types of the target function’s parameters.

Inkling does not support direct or indirect recursion. Recursive calls are flagged as errors by the compiler.

```inkling2--syntax
<subroutine>'('[<expression> ',' ] ')'
```

#### Literals
Expression may contain literal values. The following code shows examples of literal values of various types.

```inkling2--code
# Numeric literal
var Num = 3.4

# String literal
var Str = "Hello"

# Array literal
var Array = [3, 4, 5]

# Struct literal
var Struct = { X: 3, Y: 4, Z: 4 }
```   


### Math Namespace
Inkling supports a variety of built-in mathematical operators. It also provides a simple math library to access various useful mathematical constants and functions. These are available through the “Math” namespace, which can be accessed by adding the statement “using Math” to the Inkling program.

#### Constants

`const Math.E: number` Base of natural logarithm

`const Math.Pi: number` Ratio of a circle’s circumference to its diameter

#### Functions

`function Math.Abs(X: number): number` Absolute value

`function Math.ArcCos(X: number): number` Arccosine of value; return value in radians

`function Math.ArcSin(X: number): number` Arcsin of value; return value in radians

`function Math.ArcTan(X: number): number` ArcTan of value; return value in radians

`function Math.ArcTan2(X: number, Y: number): number` ArcTan2 of ratio of two values

`function Math.Ceil(X: number): number` Nearest >= integer value

`function Math.Cos(X: number): number` Cosine of value, specified in radians

`function Math.Floor(X: number): number` Nearest <= integer value

`function Math.Hypot(X: number, Y: number): number` Hypotenuse of right triangle with legs of length X and Y

`function Math.Log(X: number): number` Natural logarithm of value; runtime error if not positive

`function Math.Sin(X: number): number` Sine of value, specified in radians

`function Math.Tan(X: number): number` Tangent of value, specified in radians


[1]: #types
