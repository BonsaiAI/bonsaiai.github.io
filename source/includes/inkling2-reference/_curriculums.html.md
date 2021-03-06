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

The `simulator` declaration specifies the simulator name and the types of two input values (the action and configuration). It also specifies the type of the simulator's output (the observable state). In this instance, the configuration type is `BreakoutConfig`. The lessons initialize the configuration type of `BreakoutConfig`.


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
        scenario {
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

This lesson specifies a scenario that defines desired ranges for the `level`, `paddle_width`, and `bricks_percent` fields of the simulator's configuration. When a concept is being trained, a new simulation configuration is passed to the simulator for each new episode. The configuration values are chosen in accordance with the current lesson's scenario. For example, in the lesson above, the `bricks_percent` field is constrained to a value of 1, so the configuration passed to the simulator at the beginning of every episode will contain a value of 1 for this field. The `paddle_width` field is constrained to integers in the range of 1 through 4, so the configuration passed to the simulator at the beginning of each episode will contain a value of 1, 2, 3 or 4 for this field. Values are chosen uniformly at random from the set specified in the scenario.

The `GameState` type describes the data that is passed as an input to the graph. This same data type is provided as an output from the simulator. For Breakout, this data includes a field called `pixels` which refers to an 84x336 grayscale image.

The simulator return type must match (or be castable to) the graph's `input` type. For an explanation of castability, refer to [types][4].

In this example, the simulator declares the action type to be `PlayerMove`. The simulator action type must match the output type for the concept being trained. In our example, the concept `HighScore` trains the BRAIN to select the next move, which is of type `PlayerMove`. 

```inkling2--code
graph (input: GameState) {
  concept KeepPaddleUnderBall(input): PlayerMove {
    curriculum {
      source BreakoutSimulator

      lesson TrackBallWidePaddle {
        scenario {
          level: Number.UInt16<1 .. 100>,
          paddle_width: 4,
          bricks_percent: 1
        }
      }

      lesson TrackBallAnyPaddle {
        scenario {
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

### Defining States, Terminals, Actions, and Rewards in Inkling
When using a simulator as the source of training data, each training iteration involves an input to the simulator (an “action”) and an output from the simulator (the new “state”). In addition, a “reward” value is generated along with a “terminal” flag that indicates whether the training episode should be ended. These four piece of data (state, terminal, action, and reward) are sometimes collectively referred to as “STAR”.

#### Reward Functions
While it’s possible for the simulator code to compute the reward value, it’s often more convenient to implement this reward function in Inkling code. This allows for more rapid experimentation. Simple reward functions can be implemented using the [function][6] capabilities in Inkling.

If a reward function is specified in Inkling, the reward value passed from the simulator is ignored.

An Inkling reward function takes one or two input parameters. The first parameter provides the new state that was returned from the simulator. The second parameter (which can be omitted) is the action that was provided by the model. The reward function must return a numeric value indicating the reward associated with that state and action. The reward function is specified within the curriculum statement using the `reward` keyword.

Reward functions (along with the other functions described below) can be specified as named global functions or inlined, in which case the function name is optional.

```inkling2--code
concept Balance(input: SimState): Action {
    curriculum {
        source MySimulator
        reward GetReward
    }
}

function IsUpright(Angle: number) {
  return Angle > 85 and Angle < 95
}

function GetReward(State: SimState) {
  if IsUpright(State.Angle) {
      return 1.0
  }
  return -0.01
}
```

#### Terminal Functions
As with reward functions, terminal functions can be written within Inkling. If a terminal function is specified, the terminal value passed from the simulator is ignored.

An Inkling terminal function takes one or two input parameters. The first is the new state that was returned from the simulator. The second parameter (which can be omitted) is the action that was provided by the model. The terminal function must return true (1) if the state is a terminal state or false (0) otherwise. The terminal function is specified within the curriculum statement using the `terminal` keyword.

Note that a function is specified inline in the provided example code.

```inkling2--code
concept Balance(input: SimState): Action {
    curriculum {
        source MySimulator
        terminal function (State: SimState) {
          if State.IterationCount > 100 {
            return true
          }
          return State.Position > 20
        }
    }
}
```

#### State Transform Functions
The “state” data returned by a simulator represents the state of the simulated environment, including all observables. In many cases, the simulator is able to produce more observable values than are available in the real world. In such cases, it’s important to train the BRAIN using only the subset of real-world observables. It’s important to distinguish between the “observable state” and the “training state”.

If the training state is a proper subset of the observable state — that is, all of the fields within the training state are present in the observable state and have compatible types, the inkling compiler will automatically transform the observable state into the training state. If a more sophisticated transform is required — to translate units, scale values or combine values, this can be done through the use of a “state transform function”. A state transform is specified within the curriculum statement using the `state` keyword.

```inkling2--code
type SimState {
  AngleInRadians: number
}

type SensorState {
  AngleInDegrees: number
}

simulator MySimulator(action: Action): SimState {
}

function TranslateState(State: SimState): SensorState {
  # Convert from radians to degrees.
  return {
    AngleInDegrees: State.AngleInRadians * 180 / Math.Pi
  }
}

concept MyConcept(input: SensorState): Action {
    curriculum {
        source MySimulator
        state TranslateState
    }
}
```

#### Action Transform Functions
The “action” input to a simulator is typically specified in a manner that is compatible with the predicted action from the concept being trained. However, there are cases where the action spaces differ. For example, a simulator may encode its accepted actions using different numeric values than the real-world system in which the BRAIN will eventually be used. In such cases, an “action transform function” can be used to translate between the two action spaces. It is specified within the curriculum statement using the `action` keyword.

```inkling2--code
type SimAction number<Left=-1, Right=1>

# Note that the action space for the BRAIN encodes
# "left" differently from the action space of the
# simulator.
type BrainAction number<Left=0, Right=1>

simulator MySimulator(action: SimAction): SimState {
}

function TranslateAction(Command: BrainAction): SimAction {
  if Command == BrainAction.Left {
    return SimAction.Left
  } else {
    return SimAction.Right
  }
}

concept MyConcept(input: SimState): BrainAction {
    curriculum {
        source MySimulator
        action TranslateAction
    }
}
```

#### Training Parameters
Certain training parameters can been adjusted using a “training” clause within the curriculum statement. 

```inkling2--code
concept MyConcept(input: SimState): BrainAction {
    curriculum {
      training {
        EpisodeIterationLimit: 250,
        TotalIterationLimit: 100000
      }
    }
}
```

The following parameters can be specified in a training clause. Any parameter that is unspecified will fall back on a default value provided by the platform.

`EpisodeIterationLimit` controls the maximum number of iterations allowed per training episode. If a terminal condition or goal success condition is not reached before the episode iteration limit is reached, the episode will be terminated, and a new episode will begin. The specified value must be a positive integer. If not specified, a default value of 1000 is assumed.

`TotalIterationLimit` controls the maximum number of iterations allowed for the concept. If this limit is reached before training termination conditions are achieved for the concept, training will stop. (Training may proceed slightly beyond the iteration limit to allow the last training batch to complete.) The specified value must be a positive integer, and the default value is 50M. 

`LessonRewardThreshold` controls training termination for lessons when using a reward function. When the average reward value (over a limited window) exceeds this threshold value, the lesson is considered complete, and training proceeds to the next lesson. If this value is not specified, the platform employs a general convergence test to determine when the lesson is complete. This parameter can be used only if the curriculum specifies a reward function.

`LessonSuccessThreshold` controls training termination for lessons when using a goal. When the episode success rate (over the full lesson) exceeds this threshold value, the lesson is considered complete, and training proceeds to the next lesson. The value must be between 0 and 1. If this value is not specified, a default value of 0.90 (90%) is assumed. This parameter can be used only if the curriculum specifies a goal.

#### Training termination

Training will stop when the first of these conditions is met:

* Manually stopped by the user.
* After `TotalIterationLimit` iterations for the concept, described above. 
* When the final lesson has hit its reward or success threshold (see `LessonRewardThreshold` and `LessonSuccessThreshold` above) and the training process converges, meaning that the policy is no longer improving.


[1]: #lessons
[2]: #concepts
[3]: #simulators
[4]: #types
[5]: #constrained-types-and-range-expressions
[6]: #functions
