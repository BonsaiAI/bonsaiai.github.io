# Algorithm Hints

> Syntax

```inkling2--syntax
algorithm {
   <fieldName> ':' <literal>
   [ ',' <fieldName> ':' <literal> ]*
}
```

> Example

```inkling2--code
graph (input: GameState): Action {
    concept balance(input): Action {
        curriculum {
            source MySimulator

            algorithm {
                Algorithm: "TRPO",
                HiddenLayers: [
                    {
                        Size: 64,
                        Activation: "tanh"
                    },
                    {
                        Size: 64,
                        Activation: "tanh"
                    }
                ]
            }
        }
    }
    output balance
}
```

The algorithm clause is for advanced users who want to experiment with changing machine learning algorithms and their tuning parameters. An algorithm statement is a hint to the AI Engine which may be ignored if the AI Engine decides that the specified algorithm or parameters are inappropriate or obsolete. If the hint is followed, it applies only to the concept in which it is specified.

### Valid Algorithm Identifiers

* `DQN` (Deep Q Learning) - classification only
* `DDPG` (Deep Deterministic Policy Gradients) - estimation only
* `DiscreteDDPG` (Deep Deterministic Policy Gradients with Discrete Actions) - classification only
* `QTable` (Q Learning with Q Table) - classification only
* `TRPO` (Trust Region Policy Optimization) - estimation only


### Shared Parameter Details

The following parameters all apply to DQN, DDPG, Discrete DDPG, and TRPO. Additional parameter options are shown in subsequent tables below.

| Parameter (Example use)      | Description |
| -                            | -           |
| HiddenLayers: [HiddenLayerInfo] <br><br> (`HiddenLayers: [{Size: 400, Activation: "relu"}, {Size: 300, Activation: "tanh"}]`) | An array of structures that define the size and (optionally) the activation function for each hidden layer. Sizes must be positive integers, and activation functions must be one of "linear", "tanh", "relu", "logistic", "softmax", "elu", or "default". |
| ConvolutionLayers: [ConvolutionLayerInfo] <br><br> (`ConvolutionLayers: {XSize: 8, YSize: 8, XStride: 4, YStride: 4 FilterCount: 32}”`) | An array of structures that define the size, stride and (optionally) the filter count for each convolution layer. |
| CompressionLayers: [HiddenLayerInfo] <br><br> (`CompressionLayers: [{Size: 20, Activation: "tanh"}, {Size: 10, Activation: "softmax"}]`) | If specified, will add one or more fully-connected hidden layers after the convolutional layers to compress the representation before incorporating other features. The format is the same as the HiddenLayerInfo described above. |
| ConvolutionLimitDimension: string <br> (`ConvolutionLimitDimension: “x”`) | A string that represents which dimension of the input shape to be used as the convolutional filter sizes. Must be one of: "max", "min", "x", "y", or "none". By default, it’s "none", which uses the original input shapes as the filter sizes. "max" uses the maximum of x and y inputs as the filter size. "min" uses the minimum of x and y input. "x" uses the x dimension input shape as the filter size. "y" uses the y dimension input shape. (The example uses the input x dimension shape as the convolutional filter size.) |


### DQN-Specific Parameters

In addition to the first (shared) table of parameters, DQN also supports four additional parameters.

| Parameter (Example use)      | Description |
| -                            | -           |
| ExplorationDecay: float <br> (`ExplorationDecay: 0.00001`) | The exploration decay in DQN. |
| SamplesPerTrain: int <br> (`SamplesPerTrain: 10`) | The number of samples (iterations) collected before training. |
| QLearningRate: float <br> (`QLearningRate: 0.0001`) | The learning rate for training the Q network. |
| DiscountRate: float <br> (`DiscountRate: 0.99`) | The discount rate for the Bellman Equation. |

### DDPG-Specific and DiscreteDDPG Parameters

In addition to the first (shared) table of parameters, DDPP and DiscreteDDPG support three additional parameters.

| Parameter (Example use)      | Description |
| -                            | -           |
| MuLearningRate: float <br> (`MuLearningRate: 0.0001`) | The learning rate for training the Mu network. |
| QLearningRate: float <br> (`QLearningRate: 0.0001`) | The learning rate for training the Q network. |
| DiscountRate: float <br> (`DiscountRate: 0.99`) | The discount rate for the Bellman Equation. |

### TRPO-Specific Parameters

In addition to the first (shared) table of parameters, TRPO supports one additional parameter.

| Parameter (Example use)      | Description |
| -                            | -           |
| SamplesPerTrain: int <br> (`SamplesPerTrain: 10`) | The number of samples (iterations) collected before training. |

### QTable Parameters

QTable supports only three parameters.

| Parameter (Example use)      | Description |
| -                            | -           |
| ExplorationDecay: float <br> (`ExplorationDecay: 0.00001`) | The exploration decay in DQN. |
| QLearningRate: float <br> (`QLearningRate: 0.0001`) | The learning rate for training the Q network. |
| DiscountRate: float <br> (`DiscountRate: 0.99`) | The discount rate for the Bellman Equation. |
