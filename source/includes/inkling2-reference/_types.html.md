# Types

Inkling is a statically-typed language, which means that each variable, constant, and parameter has a well-defined type. The type system in Inkling supports the following basic types:

* Number - numeric values (integer or floating point)
* String (future) - an array of zero or more unicode characters
* Array (future) - a non-zero-length array of same-typed values
* Structure - a collection of typed fields with unique names
* Complex - built-in object types for specialized uses (like images)

## Type Literals

```inkling2--code
  number
  string
  number[3][3]
  { A: number, B: string }
```

Types may be specified as simple type literals. Number and string literals simply use the keywords `number` and `string`. (Note: Strings are not yet implemented.)

Array literals use square brackets surrounding a positive integer that represents the size of the array. Arrays may be nested. (Note: Arrays are not yet implemented.)

Structure literals are denoted with curly braces surrounding a comma-separated list of one or more field definitions. Each field definition consists of a field name and a type separated by a colon.

### Numeric Constraints

```inkling2--code
  number<1..2>
  number<0..8 step 3>
  number<0, 3.5, 7>
  number<200>
  200
  number<Left = 0, Straight = 1, Right = 2>
  number<1..100><2..99><50..51>
```

A `number` type can express any value compatible with a double-precision floating-point representation. Sometimes it's useful to constrain a number type to represent smaller ranges or sets of numbers.

A numeric type with a range constraint specifies a minimum and maximum value plus an optional step value surrounded by angle brackets. For example, number<0..8 step 3> represents a number that can take on the values 0, 4 or 8. If the range does not evenly divide by the step value, the last number in the range will be omitted. In the above example, the range would include the values 0, 3 and 6.

A numeric type with ordinal enumerated constraint specifies discrete, unique values in a comma-separated list. They must be listed in increasing order. For example, number<0, 3.5, 7> is constrained to take on one of those three values.

If an enumerated constraint contains only one value (e.g. number<200>), it can be represented more tersely using the value alone (e.g. 200).

A numeric type with nominal enumerated constraint specifies unordered symbolic identifiers in a comma-separated list. Unique numeric values must be specified for each identifier. For example,  number<Left = 0, Straight = 1, Right = 2> takes on one of three values. Values specified in a nominal enumerated type can be accessed as constants elsewhere in the program by using the type name followed by a dot and the symbolic name of the value -- for example, Direction.Left.

A constraint can be applied to an already-constrained type if it further narrows the constraints on the type.

### Built-in Number Types

```inkling2--code
  using Number
```

```inkling2--code
  Number.UInt8
  Number.Int64
  Number.Float32
  Number.Bool
```

Inkling supports built-in symbolic names for number types that correspond to integer and floating-point types commonly found in other typed languages. These are shortands for constrained number types.

These types make use of the built-in namespace `Number`, which must be referenced somewhere in the program with a `using` statement.

*	Unsigned integers: UInt8, UInt16, UInt32, UInt64
*	Signed Integers: Int8, Int16, Int32, Int64
*	Floating-point: Float32, Float64
*	Boolean: Bool

### String Constraints

```inkling2--code
  string<"A", "B", "C">
  string<"Z">
  "Z"
  number<Left = "L", Straight = "S", Right = "R">
```

As with numbers, a `string` type can be constrained using an enumerated constraint.

### Complex Types

```inkling2--code
  using Image
```

```inkling2--code
  # Default order of parameters is <width, height>
  Image.Gray<100, 200>
  Image.Gray<Height=200, Width=100>
```

Currently, the only supported complex type is Image.Gray, which represents a two-dimensional collection of gray-scale pixels. Each pixel is represented as a Number.UInt8 value (0 is black and 255 is white).

The width and height of the image must be specified using a constant expression.

## Type Declarations
```inkling2--code
  type Height number

  type Coordinate {X: number, Y: number}

  type TicTacToeValue string<"", "X", "O">

  type TicTacToeBoard TicTacToeValue[3][3]
```

A type can be associated with an identifier using a type declaration statement. This identifier can be used anywhere a type literal would be used.

> Type Declaration Syntax

```inkling2--syntax
  typeDeclaration :==
    type <typeName> typeReference

  typeReference :==
    <typeName> | typeLiteral

  typeLiteral :==
    primitiveType |
    typeReference'<' rangeExpression '>' |
    typeReference'<' valueList '>' |
    typeReference'<' typeArgumentList '>' |
    typeReference '[' constExpression ']' |
    '{' typeFieldList '}'

  rangeExpression :==
    constExpression '..' constExpression [step constExpression]?

  valueList :==
    constExpression [',' constExpression]* [',']?

  typeArgumentList :==
    typeArgument [',' typeArgument]* [',']?

  typeArgument :==
    [<paramName> '=']? typeReference

  typeFieldList :==
    typeField [',' typeField]* [',']?

  typeField :==
    <fieldName> ':' typeReference
```

## Type Equivalence

Inkling uses types to understand and interpret the format of data. The Inkling compiler performs type checks to guarantee type compatibility. In some cases, type equivalence is required (i.e. the types must match exactly). In other cases, castability is required (i.e. one type can be trivially transformed into another type).

The table below explains the rules for equivalence and castability.

  Type      |  Rules for equivalence  |  Rules for castability
----------- | ----------------------- | ----------------------- 
Number | Constraints match | Source is more constrained than dest
String | Constraints match | Source is more constrained than dest
Array  | Element type is equivalent<br> Number of dimensions match<br> Size of each dimension matches | Element type is castable<br> Number of dimensions matches<br> Size of each dimension matches
Structure | All fields are present in the same order<br> Field names match<br> Field types are equivalent | Subset of source fields are present in dest, not necessarily in same order<br> Field types of present fields are castable
Complex | Base type matches<br> Generic type arguments match | Base type matches<br> Generic type arguments match

