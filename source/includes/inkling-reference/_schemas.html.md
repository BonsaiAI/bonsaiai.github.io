# Schemas

This is the reference for the keyword `schema`. Also covered are the
definitions and uses of [Inkling types][3], including [type constraints][1].  These are used in schema declarations.

## Schema Declarations
```inkling--code
   schema bar       # schema declaration
      Bool field1,
      Int8 field2
   end
```
Schemas are declared with the `schema` statement.

In Inkling a `schema` describes a named record and its contained fields. Each field in a schema has a name and a type. A field may also have a type constraint that constrains the values that the datum described by this field will take.

```inkling--code
   concept foo
      is classifier
      predicts (bar) # schema reference
   end
```
Statements (such as the `concept` statement) can reference schemas by name.

```inkling--code
   concept foo
      is classifier
      predicts (Bool field1, Int8 field2) # anonymous schema
   end
```
Schemas can also be [anonymous][5]. In that case, instead of a schema name, a list of named fields is present.

Schemas describe the structure of data in Inkling streams, such as the
predefined `input` and `output` streams. In addition, many Inkling statements (for
example `concept` and `curriculum`) use schema references to describe the data that flows in and out of the construct.


> Schema Declaration Syntax

```inkling--syntax
schemaStmt := 
schema <name>                      
     fieldDclnList 
end

fieldDclnList   :=  fieldDcln [',' fieldDcln  ]*  

fieldDcln       :=  scalarDcln   | structureDcln 

scalarDcln      :=  primitiveType   rangeExpression? <name>
                                           
structureDcln   :=  structure_type   structure_init <name> 
                                
structure_type  :=  Luminance

structure_init  :=  '(' 
                        luminance_init
                    ')'

luminance_init  :=  integerLiteral  ',' integerLiteral 
```

In the Schema Declaration Syntax, you will see references to Inkling primitive types and structured types. These are discussed in in the [Structured Types][2] section. 


<aside class="warning">
Due to limitations with Inkling’s communication with Tensorflow you can’t end your schema variables with an underscore and number such as `variable_1`, `variable_22`, etc.
</aside>

### Usage

Schema fields must be separated from each other by commas. Schema declarations
are terminated by the `end` keyword.

Field types can be any of the [Inkling primitive types][4] and [Inkling structured types][2]. 

A schema field that has a primitive type can also have a type constraint that
constrains the set of potential values for that field. Examples and syntax of
type constraints are included in the [Constrained Types][1] section.

### Discussion

Schemas tell the BRAIN system how to translate large matrices (tensors) to usable
values.

Note the inkling compiler does not do this translation, that is a runtime
transformation.  However the inkling compiler performs static checks
to verify that the schemas are valid in the context in which they are used. 

```inkling--code
  schema MNIST_schema
    Luminance(28, 28) image # structured type
  end

  schema my_schema
    Int32 x,                # primitive type
    Int32{1:5} z            # primitive type with type constraint
  end
```

The code panel contains a few example schemas. 
The last field `z` in `my_schema` has constrained type `Int32{1:5}`. For more information on this feature see [Constrained Types][1].



## Schema References

```inkling--code
   concept MyConcept
      is classifier
      predicts (MySchema)       # a schema reference
   end
```

Inkling statements can reference schemas by name. 

###### Anonymous Schema 

```inkling--code
   concept MyConcept
      is classifier
      predicts (UInt8 MyField)  # a anonymous schema 
   end
```

Anywhere a schema name can
be referenced, a list of fields can appear. This is an **anonymous schema**. 
Anonymous schema can also be empty (that is, they can contain no
field definitions), if allowed in context.

> Schema Reference Syntax

```inkling--syntax
schemaRef :=                 
     '('   ')'                  # empty anonymous schema
 |   '(' <name> ')'             # named schema
 |   '('  <fieldDclnList> ')    # non-empty anonymous schema
```

## Types

Inkling supports both primitive types and structured types.


###### Primitive Types

> Primitive Types List

```inkling--syntax
primitiveType ::=
  Double | Float64 | Float32 | Int8 | Int16 | Int32 |
  Int64 | UInt8 | UInt16 | UInt32  | UInt64 | Bool | String
```

The Inkling set of primitive types includes numeric, string, and boolean types.

<aside class="warning">
Only constrained Strings are currently implemented.
</aside>

In the code panel you will see the set of primitive types which are used in schema declarations. 

The integer suffix indicates the size in bits of the type. 

Integer types beginning with 'U' are unsigned. 

`Double` and `Float64` are synonyms.

###### Structured Types

> Structured Types List

```inkling--syntax
structure_type ::=                      # syntax
  Luminance
```

```inkling--code
schema BallLocationSchema               # example
  Luminance(84, 336) pixels
end
```

Structured types in Inkling are intended to support common
Machine Learning types. The only Machine Learning type currently supported is
`Luminance`. This support will be expanded to `Matrix` and `Vector`, among
others in the future.

`Luminance` is a structured type for inputing a matrix of pixels in an Inkling
schema. If given a small image, as in `Luminance(9, 9)` then likely one
convolutional layer will be built. In the Breakout example, a larger image of `Luminance( 84, 336)`
will cause the AI Engine to build multiple layers and stack them to
aggressively reduce the dimensionality of the output and could also be read
as `Luminance(84, 84, 4)` for 4 layers of an 84x84 image. You can configure
these convolutional layers and filters yourself by using the advanced
[algorithm clause][6].

###### Constrained Types and Range Expressions

```inkling--code
  schema my_schema
    Int32{1:5} z            # primitive type with type constraint
  end
```

A constrained type is a type which is associated with a constraint.
The type plus constraint is effectively a set definition. 
A field declared with a
constrained type can only take values that are a member of the set defined by
the constraint. In the example in the code panel, the field `z` can only take on
values between `1` and `5`. 

Constrained types are supported in schemas and in lesson clauses. Types are
constrained by means of **range expressions**.
A range expression on a type has the effect of constraining the values of the type to values defined by the range expression. 

* In a schema the range expression constrains the values in the field. 
* In lessons the range expression constrains the values of the placeholder being configured.

```inkling--code
schema MyOutput 
   UInt8 {0,1,2,3,4}    label,       # a list of UInt8 values
   Int64 {0:5:100}      x,           # start:step:stop, step= 5, 0..100
   Int64 {0:100}        y,           # start:stop, step= 1, 0..100
   Int64 {0..100:25}    z,           # start:stop, numsteps=25, step= 4, 0..100
   Float32 {0..2:5}     a            # gives (0, .5., 1.0, 1.5, 2.0)
end
```

The syntax for a constrained type is the same for schema fields and placeholder expressions.

In the code panel are some examples of type constraints as they could appear in a schema definition. Curly braces delineate the range expression.

Inkling supports **numeric range expressions** and **value list range
expressions**. 

###### Numeric Range Expressions

> Numeric Range Expression Syntax

```inkling--syntax
 Double | Float64 | Float32 | Int8 | Int16 | Int32 | Int64 | UInt8 | UInt16 | UInt32 | UInt64
  '{' 
      start ':' [ step':']? stop       # 'colon range'
      |  
      start '.' '.' stop ':' numSteps  # 'dot range'
  '}' 
```

For numeric range expressions there are two types: 

* **colon range** 
* **dot range** 

A **dot range** must specify numSteps. numSteps must be a positive integer constant. Today, **dot range** expressions cannot be used in `predicts` schemas with `is estimator` concepts. 

A **colon range** can specifies an optional **step size**. If step size is omitted, the range includes all values of the given type between **start** and **end**. For example, in **colon range** expressions with integer types, if an explicit step is not specified, the default step size is 1.

Step size is required for FloatXX fields in the output schemas for `is classifier` concepts. 

Note that today, all fields in schemas used in the predicts clause of `is estimator` concepts must have type FloatXX with no explicit step.


### Usage

For some ranges, it is clearer to specify the number of steps in the
range then the step size. For example `Int32{-11003..743299:100}` makes it clear 
that a large range has 100 elements. Using step size for such a range would not
clearly communicate that information. 

In other contexts, the step size is of interest, and we use the colon range. For
example, in the range `Int32{0:2:10}`, we have divided the range into elements (0,2,4,6,8,10).
The step size of two communicates information about the distribution of values
in the range. 

```inkling--code
Int64  { 0:4:1 }     # is invalid. The step size is larger than the range.
Int64  { 0..1:4 }    # is invalid. Values generated are floating point not integer. 
Float32{-1..1:10 }   # is valid. Negative bounds allowed.
Int8   {0:-4:-100}   # is valid. stop < start is valid if and only if step is negative. 
UInt32 {-10:10}      # is invalid. Unsigned integer range contains signed values.
```

The code panel contains some examples of valid and invalid ranges. 

For **colon range**, step can be a floating point number. 

For **dot range**, number of steps (numSteps) must be a positive integer.

For **colon range**, the step size can be negative only if stop is less than start.

For both **colon range** and **dot range**:

* The **start point** is inclusive (it is included in the values of the range) and fixed. 

* The **start point** is exact (to the maximum extent possible if the range expression type is floating point). 

* The **end point** must be reachable from the range start by applying the step. (The range must be bounded.) 

* The **end point** may or may not be included in the values of the range. If you
land on it exactly it is in the range. If you don't land on it exactly it is not
in the range. For example `Int8 {0:3:10}` gives you (0, 3, 6, 9). The end point
`10` is not included. The specification of `10` as the stop point is not an error (because it is a limit, not an endpoint). 

You can think of the **end point** as a limit. It is included only if, after
applying the (step or numSteps) expression, you land on it exactly. 
Otherwise the highest value landed on which is less than the end point is the final value in the range. 

###### Value List Range Expressions
 
> Value List Range Expression Syntax

```inkling--syntax
primitiveType 
    '{' 
        [ integerLiteral [ ',' integerLiteral ]* ]
     |  [ floatLiteral [ ',' floatLiteral ]* ]
     |  [ booleanLiteral [ ',' booleanLiteral ]? ]
    '}' 
```

Inkling supports range expressions for value lists. 

A value list range
expression can be defined for numeric types
and the primitive type `Bool`.

### Usage

```inkling--code
    UInt8  {7, 7, 7, 7}             # is valid
    UInt8  {7, -7, 7, 7}            # is invalid (negative integer in unsigned range)
```

Value list range expressions support allowing arbitrary lists of values to constitute the
range expression set, rather than start and end points. 

A value list range expression must have all its elements be valid values for the
specified type. 

A value list range expression must conform to the signed or unsigned attribute
of the specified type (if there is such an attribute). 

###### Schema Matching

Inkling uses schemas to understand and interpret the data format of streams.  The Inkling compiler performs schema match checking and will report errors if schemas which are expected to match do not. 

```inkling--code
schema GameState
  Luminance(84, 336) pixels
end
 
concept keep_paddle_under_ball 
  is classifier
  predicts (PlayerMove)
  follows ball_location, 
          input(GameState)           # <--- (1) MATCH 
end
 
concept high_score 
  is classifier
  predicts (PlayerMove)
  follows keep_paddle_under_ball, 
          input(GameState)           # <--- (1) MATCH 
  feeds output
end
 
concept ball_location 
  is estimator
  predicts (UInt32 x, UInt32 y)
  follows 
    input(Luminance(84, 336) pixels) # <--- (2) MATCH
end
 
concept ball_X_location is estimator
  predicts (UInt32 X_location)
  follows 
    input(Luminance(84, 330) pixels) # <--- (3) NO MATCH
end
```

Matching for schemas is both structural and name based. Field names matter.
Types matter.

* Two references by name to the same schema match because a schema matches itself. 
* Two references to different schema names match if both schemas define the same list of field types in the same order with the same names. For structured types whose declaration includes size, the sizes must be equal.
* Two anonymous schemas match if both define the same field types in the same order with the same names. 
* A schema referenced by name matches an anonymous schema if both define the same field types in the same order with the same names. 

The associated example shows successful schema matching and failed schema matching. 
The rule being checked is that all uses of `input` use the same schema. 

* (1) shows a valid schema match by name (for `input`). 
* (2) shows a valid match by field type, field name, and size (for `input`). 
* (3) shows a failed match (size is not equal, for `input`). 

Every Inkling program has the predefined stream `input` available to it. Since there is a single stream associated with the keyword `input`, there can be only one definition of the data format of that stream. Once defined, the data format of a stream cannot change dynamically.  If that schema changes inkling compiler will flag that change as an error. Thus any reference to the `input` stream must have a schema that matches all other schemas used with the `input` stream. 

###### Constraint Compatibility in the Lesson Configure Clause

```inkling--code
simulator breakout_simulator(BreakoutConfig)
   state (GameState)
   action(PlayerMove)
End
 
schema BreakoutConfig
  UInt32 level,
  UInt8{1:4} paddle_width,
  Float32 bricks_percent
end
 
curriculum ball_location_curriculum
  train ball_location
  with simulator breakout_simulator
  objective ball_location_distance
   
    lesson more_bricks follows no_bricks
      configure
        constrain bricks_percent with Float32{0.1:0.01:1.0},
        constrain level with UInt32{1:100},
        constrain paddle_width with UInt8{1:4}
      until
        maximize ball_location_distance
end
```

Constraints can be specified in the lesson configure clause. 
These constraints must be compatible with the configuration schema of the
associated simulator. 
The code panel contains some Inkling code from the breakout program to show an example. 

The constraints in a lesson configure clause are constraints on the fields of
the configuration schema of the simulator.
Here, the lesson configure constraints are constraining the fields of schema `BreakoutConfig`.

Given simulator `breakout_simulator`, the constraints in the lesson configure section must have these characteristics to be compatible with schema `BreakoutConfig`:

1. They must specify exactly the same type and field name as the schema declaration. A constraint cannot use an Int8 where a UInt32 was originally specified.
2. For the range expression, the range in the configure must specify exactly the same range or a subrange as the schema declaration.
3. Step size or numSteps should not be specified in the configuration schema of
the simulator. They can be specified in the lesson configure clause.
Specifically, in the `BreakoutConfig` schema above, `UInt8{1:4} paddle_width` is valid but `UInt8{1:2:4} paddle_width` would not be valid.

Here is the constraint for `level` that is specified in the lesson `more_bricks` configure clause:

`constrain level with UInt32{1:100}`

Here is the constraint for `level` in the simulator configuration schema `BreakoutConfig`:

`UInt32 level`

> Examples of valid constraints:

```inkling--code
constrain paddle_width with UInt8{1:3}    # ok: a subset of constraint set

constrain paddle_width with UInt8{2}      # ok: value in constraint set
```

> Examples of invalid constraints

```inkling--code
constrain paddle_width with UInt8{1:5}    # not a subset of constraint set

constrain paddle_width with UInt8{7}      # value not in constraint set
```

The lesson configure clause constraint is compatible with the simulator
configuration constraint because the latter declared
no range at all and any range expression is a subrange of all the values
available to a type. 

The field `paddle_width` in the simulator configuration schema `BreakoutConfig`
does specify a range expression: 

`UInt8{1:4} paddle_width`

The corresponding range expression in the lesson configure clause is identical:

`constrain paddle_width with UInt8{1:4}`

These are compatible.

In the examples in the code panel, we show more examples of valid and invalid lesson
configure constraints for the field `paddle_width`.

[1]:#constrained-types-and-range-expressions
[2]:#structured-types
[3]:#types
[4]:#primitive-types
[5]:#anonymous-schema
[6]:#advanced-algorithms
