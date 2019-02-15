# Types

Values in Inkling, including inputs and outputs of concepts, have defined types.

The AI engine uses types to understand the data it is sent and how to represent the prediction it generates. Inkling is a statically-typed language. Errors will be reported if values are not compatible with their expected type.

Inkling supports the following base types:

* number - a scalar numeric value with double floating point precision and range
* structure - a list of fields, each with its own specified type and unique name
* string (future) - a string of zero or more Unicode characters
* array (future) - a fixed-length list of values each with the same type

## Type Declarations

A symbolic name can be associated with a type, and the name can be used anywhere a type can be specified.

```inkling2
# Numeric type declaration
type Dimension number

# String type declaration
type PortName string

# Structure type declaration
type Place {
  Name: string,
  Longitude: number,
  Latitude: number
}

# Structure types can refer to other types
type SensorState {
  Height: Dimension,
  Width: Dimension
}
```

## Type References

In Inkling, type references always follow a colon, which separates the field, variable, function or parameter from its type.

```inkling2
# Graph input is of type GameState, and output is of type Action
graph (input: GameState): Action {
  # Graph body omitted
}
```

## Range Constraints

Number types can be constrained using range constraints, which consist of a start and end value and an optional step amount.

```inkling2
# A real (floating-point) value from 0 to 5
type TypeA number<0..5>

# An integer value from 0 to 5, inclusive
type TypeB number<0..5 step 1>

# A number in the set {-1, -0.5, 0, 0.5, 1}
type TypeC number<-1..1 step 0.5>

# Constrained types can be further constrained
type TypeD TypeA<1..2>

# This will generate an error
type TypeE TypeA<1..7>
```

## Enumerated Constraints

Number and string types can be constrained using enumerated constraints, which list the allowed values explicitly. Two forms of enumerated constraints are supported. Ordinal enumerations are assumed to have a meaningful order and must be listed in increasing order. Nominal enumerations have names but no meaninful order. 

```inkling2
# Ordinal enumeration
type Angle number<0, 15, 30, 90, 180>

# Nominal enumeration
type Direction number<Left = 0, Right = 1, Straight = 2>

# String ordinal enumeration
type Color string<"Red", "Blue", "Green">
```

## Built-in Numeric Types

Inkling supports built-in names for signed and unsigned integers of various sizes as well as floating point (32 and 64 bit). More details can be found under [Inkling Types][2].

## Complex Types

Inkling supports built-in types for complex objects that are commonly used as data inputs. More details can be found under [Inkling Types][2].

```inkling2
using Image

type SensorInput {
  ForwardCamera: Image.Gray<200, 200>,
  DepthSensor: Image.Gray<100, 50>
}
```

[1]: ./../references/inkling2-reference.html#types
[2]: ./../references/inkling2-reference.html#types
