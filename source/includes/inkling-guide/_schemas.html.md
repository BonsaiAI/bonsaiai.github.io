# Schemas

Schemas describe a record and its fields. They contain a set of named data types that can be used throughout the system. They can include the common basic data types and native data types for working with common media formats (images, audio recordings). You add schemas in Inkling using the `schema` keyword.

Types (or [data types][2]) are representations of values of data. A type informs the system what the meaning of the data is, the possible values of that data is, what operations can be performed on that data, and the way the data can be stored. Types are the most basic building blocks of data manipulation.

In Inkling we need types, because the AI needs to understand the data it is sent. Also the AI needs to know how to represent the prediction it sends back to the program that you have deployed it with.

Inkling is also a strongly-typed language, which means that you are will receive an error if values are not compatible with their expected type. This means you need to pay attention to what types you choose.

Inkling supports various types, including (but not limited to) primitive types, which include types for integers, floats, bytes and strings, as well as types commonly used with machine learning (for example, Luminance). See the section on [Inkling types][3] for more information.

## Constrained Types

Inkling supports [constrained types][4] in schemas (as well as for configuration of lessons). Constrained types use range expressions to constrain the values of the type to values defined by a range expression.

A range expression has the effect of constraining the values of the type to values defined by the range expression. In a schema, this constrains the values in the field. In lessons, this constrains the values of the placeholder being configured. In both cases the syntax is the same.

Here are some examples of constrained types:

```inkling
schema MyOutput
  UInt8  {0,1,2,3,4}   label,    # a list of UInt8 values
  String {"a", "bc"}   category, # a list of Strings
  Int64  { 0:5:100 }   x,        # start:step:stop, {0,5,10,...,95,100}
  Int64  { 0:100 }     y,        # start:stop, default step= 1, 0..100
  Int64  { 0..100:25 } z,        # start..stop:numsteps, so step= 4, {0, 4, 8,...,96, 100}
  Float32 { 0..2:5}    a         # start..stop:numsteps, gives {0, 0.5, 1.0, 1.5, 2.0}
end
```

## Defining Schemas

Examine the input, returned data, and output of your mental model when defining your schemas. You need to match these requirements to Inkling types.

### Inputs and Outputs

**Input** is a stream of information that is fed into your BRAIN (your AI). The Bonsai AI Engine uses this information to help train the BRAIN or make a prediction.

In Inkling, **Output** is a stream of information returned as a prediction that your BRAIN sends back to an application. It could be sent back to a simulator, if training is in process, or it could be send back to your deployed application, where it might be used for control or optimization. The data in input and output streams are described by schemas. Schemas are declared in your Inkling program, and they contain information about the data types contained in the stream. For example, if you want to teach your AI,  to recognize the shape 'square', you might give it a picture of a shape. That picture would be an input. The BRAIN answers yes (this is a square) or no (this is not a square). That answer is the output. Your Inkling file to teach your AI would contain one schema for describing the data type of the input (Luminance, an image type) and another for the output (Bool). For more information about schemas, refer to the [Schema Reference][1].

### How to pick types

* If your output is yes/no, true/false, or another dual relationship, you should use the `bool` type.
* Inkling supports signed and unsigned integers of various sizes as well as floating point (32 and 64 bit). More details can be found under [Inkling Types][3].

[1]: ./../references/inkling-reference.html#schema
[2]: https://en.wikipedia.org/wiki/Data_type
[3]: ./../references/inkling-reference.html#inkling-types
[4]: ./../references/inkling-reference.html#constrained-types
