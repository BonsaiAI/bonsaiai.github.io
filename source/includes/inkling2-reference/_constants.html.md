# Constants

This is the reference for the keyword **const**. 

```inkling2--code
const MaxVelocity = 100
const MinVelocity = 0
const Environment = "NewYorkCity"
const DebugMode = true
const DestinationPoint = [4.5, 2.7, 9.2]
```

Inkling supports named constants for all Inkling types. The name can be used wherever a value can appear.

Constants make code easier to understand and modify because the constant name
conveys the semantics of the constant. They document the logic of the program.

Types can be specified for constants. If a type is omitted, it is inferred. If a type is specified, the compiler validates that the constant value matches the specified type.

The constant value is specified as an expression that can make use of supported operators, numeric or string literals, and other constants. Cyclical constant references are not allowed.


```inkling2--code
# Valid constant declarations
const NumericConst1: number = 100
const NumericConst4: number<0 .. 2> = 3 * (5 / 2) - 7
const NumericConst5 = NumericConst1 * 2
const StringConst1: string = "Hello"
const ArrayConst1: number[3] = [0, 2, 5]
const StructConst1: {X: number, Y: string} = {X: 3, Y: "Foo"}

# Errors: type mismatch
const NumericConst2: number = "Hello"
const NumericConst3: number<0 .. 2> = 3
const StringConst2: string = ["Hello"]
const StructConst2: {X: number, Y: string} = {X: 3, Y: 3}
const StructConst3: {X: number, Y: string} = {X: 3}

# Error: cyclical reference
const NumericConst6 = NumericConst6
```

Some valid and invalid constant declarations are shown in the accompanying panel.


The constants declaration syntax is shown in the accompanying panel.
> Constant Syntax

```inkling2--syntax
constantDeclaration ::=
  const <constName> [':' typeReference] = constExpression
```

### Usage

* A constant declaration can appear only in the global program scope. It cannot be nested within another statement. 
* A named constant does not have to be declared before it is used. 
* A named constant can be used anywhere a literal can be used.
* The initialized value of the constant must be compatible with the declared type of the constant.
* If no type is declared, it is inferred from the value.
* The declared type of the constant must be compatible with the type of the context where it is used.
