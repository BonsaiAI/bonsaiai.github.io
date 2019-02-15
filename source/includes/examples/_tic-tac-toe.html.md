# Luminance tic-tac-toe

> ![Tic-Tac-Toe](../images/Tic-Tac-Toe.gif)

[**Download the full source code on GitHub**][1] to run this simulator.

To demonstrate the use of images in the Bonsai platform, this example will walk you through a simple implementation of tic-tac-toe. The full Inkling file is presented to walk through each part of the file and the rest of the code to run the example is found at the link above.

```python
letters = {"X": [[1, 0, 1], [0, 1, 0], [1, 0, 1]],
           "O": [[0, 1, 0], [1, 0, 1], [0, 1, 0]],
           " ": [[0, 0, 0], [0, 0, 0], [0, 0, 0]]}
```

This example will train a BRAIN using a 9x9 grayscale pixel grid (an input matrix) to represent the tic-tac-toe board. The representation of each letter, X, O, and a blank space, each are represented by a 3x3 matrix as shown in the code panel. There are 9 options for movement to correspond with the 9 spaces on the board.

This example also contains an advanced-level algorithm clause in its Inkling File. This clause is not required, but it will help the example train much faster. For more information on use of the algorithm clause see the [Inkling Reference][2].

## Inkling File

###### Types

```inkling2
type GameState {
    image: Image.Gray<9, 9>
}
```

The `GameState` type defines a field `image` that uses the complex type `Image.Gray` to create a 9x9 pixel grid (an input matrix) to represent all possible states of the tic-tac-toe board. Each letter (or blank space) is a 3x3 image, so a 3x3 grid of 3x3 letters (or blanks) results in a 9x9 grid.

```inkling2
type PlayerMove {
    move: Number.Int8<1, 2, 3, 4, 5, 6, 7, 8, 9>
}
```

The `PlayerMove` type classifies all 9 possible moves the AI can make, one for each space on the tic-tac-toe board.

###### Concept Graph

```inkling2
graph (input: GameState): PlayerMove {
    concept play_tictactoe(input): PlayerMove {
        curriculum {
            algorithm {
                Algorithm: "DQN",
                ConvolutionLayers: [{
                    XSize: 3,
                    YSize: 3,
                    XStride: 3,
                    YStride: 3,
                    FilterCount: 2
                }]
            }
            source tictactoe_simulator
        }
    }
    output play_tictactoe
}
```

The concept, `play_tictactoe`, returns a value of type `PlayerMove` which indicates which of nine possible actions should be taken. The AI picks this action based on the current state of the game, `GameState` as an input.

###### Simulator

```inkling2
simulator tictactoe_simulator(action: PlayerMove): GameState {
}
```

The simulator clause declares that a simulator named `tictactoe_simulator` will be connecting to the server for training. This `tictactoe_simulator` expects an action of type `PlayerMove` as input and replies with a value of type `GameState`.

The algorithm clause provides a hint to the AI Engine that a `DQN` algorithm should be used. It also sets some parameters for this algorithm. This hint is not required for this example but it will help the example train much faster. For more information on use of the algorithm clause see the [Inkling Reference][2].

## Simulator File

The full simulator file *tictactoe_simulator.py* for this example is too long to display in full here but you can see it with the rest of the [tic-tac-toe sample code][1] on GitHub.

For more information on the functions inside of this simulator file and how to implement them see the [Library Reference][3].

[1]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/samples/tic-tac-toe
[2]: ./../references/inkling2-reference.html#advanced-algorithms
[3]: ./../references/library-reference.html
