# Graph

The `graph` keyword declares the concept graph. Only one graph declaration is allowed per Inkling program. It defines the input type and output type of the BRAIN. It also contains the list of declared concepts and designates which concept provides the output for the graph.

```inkling2--code
graph (input: InputType): OutputType {
  # Concept declarations omitted

  output MyConcept
}
```

> Graph Syntax

```inkling2--syntax
graphStatement :=
graph '(' input ':' inputType ')' [':' outputType]? '{'
  conceptDeclaration*

  output <conceptName>
'}'

inputType ::= typeReference
outputType ::= typeReference
```

The inputType specifies the graph's input type.

The outputType specifies the graph's output type. If omitted, it is inferred by the output type of the output concept. If provided, the compiler verifies that the specified type matches that of the output concept.

The `output` clause must refer to one of the declared concepts.
