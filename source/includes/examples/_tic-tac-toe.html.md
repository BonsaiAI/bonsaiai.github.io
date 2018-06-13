# Tic-tac-toe Example

> ![Tic-Tac-Toe](../images/Tic-Tac-Toe.gif)

[**Download the full source code on GitHub**][1] to run this simulator.

To demonstrate the use of Luminance in The Bonsai Platform this example will walk you through a simple implementation of tic-tac-toe. The full Inkling file is presented to walk through each part of the file and the rest of the code to run the example is found at the link above.

```python
letters = {"X": [[1, 0, 1], [0, 1, 0], [1, 0, 1]],
           "O": [[0, 1, 0], [1, 0, 1], [0, 1, 0]],
           " ": [[0, 0, 0], [0, 0, 0], [0, 0, 0]]}
           ```

This example will train a BRAIN using a 9x9 pixel grid (an input matrix) to represent the tic-tac-toe board. The representation of each letter, X, O, and a blank space, each are represented by a 3x3 matrix as shown in the code panel. There are 9 options for movement to correspond with the 9 spaces on the board.

This example also contains an advanced-level algorithm clause in its Inkling File. This clause is not required but it will help the example train much faster. For more information on use of the algorithm clause see the [Inkling Reference][2].

## Inkling File

###### Schema

```inkling
schema GameState
    Luminance(9, 9) image
end
```

The `GameState` schema uses `Luminance` to create a 9x9 pixel grid (an input matrix) to represent all possible states of the tic-tac-toe board. This is because each letter (or blank space) is a 3x3 matrix of its own so a 3x3 grid of 3x3 letters (or blanks) results in a 9x9 grid.

```inkling
schema PlayerMove
    Int8{1, 2, 3, 4, 5, 6, 7, 8, 9} move
end
```

The `PlayerMove` schema classifies all 9 possible moves the AI can make, one for each space on the tic-tac-toe board.

```inkling
schema DummyConfig
    Int8 dummy
end
```

The `DummyConfig` schema in this case is not used (but is still required to be defined in Inkling). It would define the configuration for the `highscore` lesson if needed but in our case ends up getting set to the default value of -1 (unused).

###### Concept

```inkling
concept play_tictactoe is classifier
    predicts (PlayerMove)
    follows input(GameState)
    feeds output
end
```

This concept named `play_tictactoe` is a classifier, which means it has to pick one of a set number of 9 options in the `PlayerMove` schema. The AI picks this action based on the current state of the game, `GameState` as an input.

###### Simulator

```inkling
simulator tictactoe_simulator(DummyConfig)
  action  (PlayerMove)
  state  (GameState)
end
```

The simulator clause declares that a simulator named `tictactoe_simulator` will be connecting to the server for training. This `tictactoe_simulator` expects an action defined in the `PlayerMove` schema as input and replies with a state defined in the `GameState` schema as output.

###### Algorithm

```inkling
algorithm My_DQN_Settings
    is DQN
    hidden_layer_size => "32",
    hidden_layer_activation_descriptor => "'relu'",
    conv_layer_descriptor => "3x3:3:3:2",
    conv_compression_size_descriptor => "32",
    conv_compression_activation_descriptor => "'relu'"
end
```

This algorithm schema sets the algorithm used to be `DQN` and also sets some parameters for this algorithm. One to note is the `conv_layer_descriptor` which has the syntax of `“x_sizexy_size:x_stride:y_stride:filters”` which translates to: "Make a convolutional layer composed of two filters, each of size 3 pixels by 3 pixels, and with a vertical and horizontal stride of 3. One of the filters is for X, one filter for O (exactly the size of the X or O) with a stride of exactly one character's width."

This clause is not required for this example but it will help the example train much faster. For more information on use of the algorithm clause see the [Inkling Reference][2].

###### Curriculum

```inkling
curriculum ticatactoe_curriculum
    train play_tictactoe
    using algorithm My_DQN_Settings
    with simulator tictactoe_simulator
    objective get_reward
        lesson highscore
            configure
                constrain dummy with Int8{-1}
            until
                maximize get_reward
end
```

The curriculum `tictactoe_curriculum` trains `play_tictactoe` using `tictactoe_simulator`. The BRAIN that runs this Inkling code will try to maximize the value returned from `get_reward` until you stop training. The BRAIN has an incentive to win the game of tic-tac-toe from `get_reward` returning a reward of -1 if the computer wins, 0.5 if a tie, and 1 if it wins.

This curriculum contains one lesson, called `highscore`. It configures the simulation with a default (unused) value of -1 because there is no need for a configuration in this example.

## Simulator File

The full simulator file *tictactoe_simulator.py* for this example is too long to display in full here but you can see it with the rest of the [tic-tac-toe sample code][1] on GitHub.

For more information on the functions inside of this simulator file and how to implement them see the [Library Reference][3].

[1]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/samples/tic-tac-toe
[2]: ./../references/inkling-reference.html#advanced-algorithms
[3]: ./../references/library-reference.html
