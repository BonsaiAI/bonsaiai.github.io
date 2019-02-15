# Lessons

Lessons provide a staged way to teach a concept starting with easier tasks and moving to more difficult. This approach can reduce the total time it takes to train a concept.

> Lesson Syntax

```inkling2--syntax
lessonStatement ::=
  lesson <lessonName> '{'
    [constraint <typeReference>]?
  '}'
```

The `lesson` declares a lesson within a curriculum. Lessons provide control over the training of a concept, allowing it to be staged from easy to difficult. This approach can reduce the overall training time.

### Usage

Multiple lessons can be used within a curriculum, and they are used at training time in the same order in which they're declared. The `constraint` clause should configure the simulator in a restricted manner. Subsequent lessons should incrementally reduce the restrictions, making the problem more difficult and the exploration space larger. A range constraint used within a lesson should be less restrictive (or at least not more restrictive) than the corresponding range constraint used in a previous lesson.

### Example

```inkling2--code
simulator BreakoutSimulator(action: PlayerMove, config: BreakoutConfig): GameState {
}

type BreakoutConfig {
  level: Number.UInt16,
  paddle_width: Number.UInt8<1 .. 4>,
  bricks_percent: number
}

type PlayerMove number<Left = -1, Stay = 0, Right = 1>

type GameState {
  pixels: Image.Gray<84, 336>
}

graph (input: GameState) {
  concept HighScore(KeepPaddleUnderBall, input): PlayerMove {
    curriculum {
      source BreakoutSimulator

      lesson ConstantBreakout {
        constraint {
          level: 1,
          paddle_width: 4,
          bricks_percent: 1
        }
      }

      lesson VaryBreakout {
        constraint {
          level: Number.UInt16<1 .. 100>,
          paddle_width: Number.UInt8<1 .. 4>,
          bricks_percent: number<0.1 .. 1 step 0.01>
        }
      }
    }
  }

  output HighScore
}
```

In this example, we show lessons that break into stages the task of playing the game Breakout. 

* The first lesson, `ConstantBreakout`, trains the AI with a fixed configuration parameter.
* The second lesson, `VaryBreakout` trains the AI with a configuration parameter that varies.

The types specified after the `constraint` keyword in our example specifies a type that must be compatible with the config type defined in the simulator declaration (`BreakoutConfig`).

