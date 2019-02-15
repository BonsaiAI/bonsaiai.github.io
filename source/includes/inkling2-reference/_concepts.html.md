# Concepts

The `concept` keyword declares an abstract concept that is to be learned by the system. It can be a feature (such as a curvy line in an image) or a goal (such as high score in a game). Ultimately, a concept represents a function from input to output. By declaring a concept, you are instructing the AI Engine that this is a node in the concept graph that must be learned.

```inkling2--code
concept AbstractConceptName(Antecedent1, Antecedent2): OutputType {
  # Curriculum omitted
}
```

Because concepts are learned, their declarations tend to be fairly simple. The typical components of a concept statement are shown in the accompanying panel. 

> Concept Syntax

```inkling2--syntax
conceptDeclaration :=
concept <conceptName> '(' inputItemList ')' ':' outputType '{'
  curriculumClause
'}'

inputItemList ::=
  [inputItem] | [inputItem ',' inputItemList]

inputItem ::=
  input | <conceptName>  # name of a concept or input

outputType ::= typeReference
```

The inputItemList specifies the concept's inputs. Cycles in the graph are not allowed.

The outputType specifies the concept's output [type][1].

### Usage

The concept statement specifies input sources and an output type. Input sources
are provided in the input list. 

```inkling2--code
concept bar (input): Move {
  # Curriculum omitted
}

concept foo (bar): Action {
  # Curriculum omitted
}
```

Inputs can be the output of other concepts or the `input` data stream. The `input` stream
is the original input to the concept graph.

Input types do not need to be specified because their types are provided in the graph statement (in the case of `input`) or via the concept type (in the case of other concepts). 

### Example

We show Inkling for the concept GetHighScore.

```inkling2--code
concept GetHighScore(input): PlayerMove {
  # Curriculum omitted
}
```

* `conceptName`: GetHighScore
* `inputItemList`: `input`
* `outputType`: PlayerMove


