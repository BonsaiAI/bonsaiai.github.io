# Curriculums

```inkling2--code
concept MyConcept(input): OutputType {
  curriculum {
    source MySimulator

    # Lessons specified here

  }
}
```

The `curriculum` statement is used within a [concept][2] to specify how the concept should be trained.

The [lessons][1] defined within the curriculum can be used to create a staged teaching plan.

> Curriculum Syntax

```inkling2--syntax
curriculum {
  source <simulatorReference>

  [lessonStatement]*
}
```   

### Usage

There can be only one curriculum per concept, and every concept must have a curriculum.

Every curriculum must provide a `source` clause that specifies the data source for teaching the concept. Currently, only [simulators][3] are supported as data sources, so the `source` keyword must be followed by a simulator declaration or the name of a simulator declared at the global scope of the program.

### Breakout Example

> Breakout Example

```inkling2--code
simulator BreakoutSimulator(action: PlayerMove, config: BreakoutConfig): GameState {
}
```

The `simulator` declaration specifies the simulator name and the types of two input values (the action and configuration). It also specifies the type of the simulator's output (the simulator state). In this instance, the configuration type is `BreakoutConfig`. The lessons initialize the configuration type of `BreakoutConfig`.


```inkling2--code
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
  concept HighScore(input): PlayerMove {
    curriculum {
      source BreakoutSimulator

      lesson ScoreLesson {
        constraint {
          level: Number.UInt16<1 .. 100>,
          paddle_width: Number.UInt8<1 .. 4>,
          bricks_percent: 1
        }
      }
    }
  }

  output PlayBreakout
}
```

The configuration type `BreakoutConfig` is constrained in the lesson. When a concept is being trained, a new simulation configuration is passed to the simulator for each new episode. The configuration values are chosen in accordance with the current lesson's constraint. For example, in the lesson above, the `bricks_percent` field is constrained to a value of 1, so the configuration passed to the simulator at the beginning of every episode will contain a value of 1 for this field. The `paddle_width` field is constrained to integers in the range of 1 through 4, so the configuration passed to the simulator at the beginning of each episode will contain a value of 1, 2, 3 or 4 for this field. Values are chosen uniformly at random from the set specified in the constraint.

The `GameState` type describes the data that is passed as an input to the graph. This same data type is provided as an output from the simulator. For Breakout, this data includes a field called `pixels` which refers to an 84x336 grayscale image.

The simulator return type must match (or be castable to) the graph's `input` type. For an explanation of castability, refer to [types][4].

In this example, the simulator declares the action type to be `PlayerMove`. The simulator action type must match the output type for the concept being trained. In our example, the concept `HighScore` trains the BRAIN to select the next move, which is of type `PlayerMove`. 

```inkling2--code
graph (input: GameState) {
  concept KeepPaddleUnderBall(input): PlayerMove {
    curriculum {
      source BreakoutSimulator

      lesson TrackBallWidePaddle {
        constraint {
          level: Number.UInt16<1 .. 100>,
          paddle_width: 4,
          bricks_percent: 1
        }
      }

      lesson TrackBallAnyPaddle {
        constraint {
          level: Number.UInt16<1 .. 100>,
          paddle_width: Number.UInt8<1 .. 4>,
          bricks_percent: 1
        }
      }
    }
  }

  output KeepPaddleUnderBall
}
```

Another concept that helps with playing Breakout is `KeepPaddleUnderBall`. In the curriculum for this concept, there are two lessons. One of the lessons, `TrackBallAnyPaddle`, uses a wide paddle and is easier to learn. The second lesson, `TrackBallAnyPaddle`, uses a [range expression][5] to vary the paddle width from 1 to 4. The second lesson is more difficult to learn, but by starting with an easier problem, the overall training time is reduced.

[1]: #lessons
[2]: #concepts
[3]: #simulators
[4]: #types
[5]: #constrained-types-and-range-expressions
