# Simulators

```inkling2--code
simulator MySimulator(action: MyActionType, config: MyConfigType): MyStateType {
}
```

Curriculum statements reference an associated simulator in the [source clause][3].

The `simulator` statement describes the interface to a simulator program. The Inkling program does not contain code for the simulator itself. Instead the Inkling program defines how the simulator is used to train Inkling concepts.

> Simulator Declaration Syntax

```inkling2--syntax
simulator <simulatorName>'('action <typeReference>
  [',' config <ctypeReference>]? ')' ':' <typeReference> '{'
'}'
```

A `simulator` statement defines the interface to the simulator - its input and output parameter types.

* The action input type
* An optional configuration input type
* The state output type

### Usage

```inkling2--code
curriculum {
  source BreakoutSimulator
}
```

The curriculum statement references a simulator. The example in the code panel
shows the use of simulator `BreakoutSimulator`. 

### Discussion

Simulators are virtual environments designed to simulate the behavior of a real-world pysical environment and the objects or agents within it. Every simulator has "observable state", a representation of the world inside the virtual environment. This state changes over time in response to actions taken by an agent. Typically, the observable state represents the inputs received by the simulated agent's sensors (cameras, temperature guages, depth sensors, etc.).

During training, the AI Engine "drives" the simulator by giving it an action. The simulator then performs the action and returns the resulting state. The AI Engine then selects a new action and sends it to the simulator, and so on. Through this manner, the AI Engine is able to explore the "state space" of the simulated environment.

Our [Library Reference][1] describes the classes and methods used to connect an existing simulator or create a new simulator in Python. [Find the Center][2] is an example of a basic simulator implementation.

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

graph (iput: GameState) {
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

In this example we show some of the Inkling code for training the game Breakout.
The curriculum `source` clause specifies `BreakoutSimulator`, and the
`simulator` statement specifies the action, state, and configuration types. 

[1]: ./library-reference.html
[2]: ./../examples.html#basic-python-c-simulation
[3]: #curriculums
