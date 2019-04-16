# Advanced: Algorithms

> Syntax (in the curriculum)

```inkling-syntax
using algorithm <algorithmIdentifier>
   [
     <parameterName> => <literal>
     [ ‘,’ <parameterName> => <literal> ]*
   end
]?
```

> Example (in the curriculum)

```inkling-code
schema GameState
   Float32 position,
   Float32 velocity,
   Float32 angle,
   Float32 rotation
end

schema Action
   Float32 command
end

schema CartPoleConfig
   Int8 episode_length,
   UInt8 deque_size
end

simulator the_simulator(CartPoleConfig)
   action (Action)
   state (GameState)
end

concept balance is estimator
   predicts (Action)
   follows input(GameState)
   feeds output

end

curriculum balance_curriculum
   train balance
   using algorithm TRPO
      hidden_layer_descriptor => [64, 64]
      hidden_layer_activation_descriptor => [“tanh”, “tanh”] 
   end

   with simulator the_simulator
   objective open_ai_gym_default_objective

       lesson balancing
           configure
               constrain episode_length with Int8{-1},
               constrain deque_size with UInt8{1}
           until
               maximize open_ai_gym_default_objective
end
```



> Syntax (in the global scope)

```inkling--syntax
algorithm <localAlgorithmId>
   is <globalAlgorithmId>
   [
     <parameterName> => <literal>
     [ ‘,’ <parameterName> => <literal> ]*
   ]?
end
```

> Example (in the global scope)

```Inkling--code
schema GameState
   Float32 position,
   Float32 velocity,
   Float32 angle,
   Float32 rotation
end

schema Action
   Float32 command
end

schema CartPoleConfig
   Int8 episode_length,
   UInt8 deque_size
end

simulator the_simulator(CartPoleConfig)
   action (Action)
   state (GameState)
end


algorithm My_TRPO_Settings
   is TRPO
   hidden_layer_descriptor => [64, 64]
   hidden_layer_activation_descriptor => [“tanh”, “tanh”] 
end

concept balance is estimator
   predicts (Action)
   follows input(GameState)
   feeds output

end

curriculum balance_curriculum
   train balance
   using algorithm My_TRPO_Settings
      # optional to override parameter with same type
      hidden_layer_descriptor => [128, 128]
   end

   with simulator the_simulator
   objective open_ai_gym_default_objective

       lesson balancing
           configure
               constrain episode_length with Int8{-1},
               constrain deque_size with UInt8{1}
           until
               maximize open_ai_gym_default_objective
end
```

The algorithm clause is for advanced users who want to experiment with changing machine learning algorithms and their tuning parameters. Inkling supports an algorithm use clause inside the curriculum statement (the common method) which applies only to that curriculum, as well as an algorithm definition clause in global scope for use in multiple curriculums. In the less common use case, a user can define named algorithm settings they can re-use in multiple curriculums (when using multiple concepts). It is also possible to override a global parameter inside of a curriculum if needed.

### Valid Algorithm Identifiers

* `APEX` (Distributed Deep Q Network) - discrete action spaces only
* `PPO` (Proximal Policy Optimization) - continuous action spaces only


### Shared Parameter Details

The following parameters all apply to APEX and PPO. Additional parameter options are shown in subsequent tables below.

| Parameter (Example use)      | Description |
| -                            | -           |
| hidden_layer_size_descriptor => [int, int, int, …] <br><br> (`hidden_layer_size_descriptor => [400, 300]` | The sizes of each hidden layer. One integer size per hidden layer. |
| hidden_layer_activation_descriptor => [string, string, string, …] <br><br> (`hidden_layer_activation_descriptor => [“relu”, “tanh”]`) | An array of activation functions (corresponding to the hidden layer size descriptor): string is the activation function type. Strings supported: “linear”, “tanh”, “relu”, “logistic”, “softmax”, “elu”. (Using this should be combined with `hidden_layer_size_descriptor`, in the given example “relu” is the activation function for the first layer, and “tanh” for the second one.) |
| conv_layer_descriptor => “x_sizexy_size:x_stride:y_stride:filters; …” <br><br> (`conv_layer_descriptor => “8x8:4:4:32; 3x3:1:1:16”`) | A semicolon delimited string of a convolutional network configuration (filters is number of filters). |
| conv_compression_size_descriptor => [int, int, int, ...] <br><br> (`conv_compression_size_descriptor => [20, 10]`) | An array of convolutional compressed size: int is the hidden layer size. If specified, will add one or more fully connected layers after the convolutional layers to compress the representation before incorporating other features. |
| conv_compression_activation_descriptor => [string, string, string, …] <br><br> (`conv_compression_activation_descriptor => [“softmax”, “tanh”]`) | An array of activation functions for convolutional network compression: string is the activation function type. Strings supported: “linear”, “tanh”, “relu”, “logistic”, “softmax”, “elu”. (Using this should be combined with `conv_compression_size_descriptor`, in the given example  “softmax” is the activation function for the first layer, and “tanh” for the second one.) |
| conv_limit_dim => string <br> (`conv_limit_dim => “x”`) | A string that represents which dimension of the input shape to be used as the convolutional filter sizes. Strings supported: “max”, “min”, “x”, “y”, “0”, “1”. By default, it’s `None`, which uses the original input shapes as the filter sizes. “max” uses the maximum number of x and y inputs as the filter sizes. “min” uses the minimum number of x and y inputs as the filter sizes. “x” or “0” uses the x dimension input shape as the filter sizes. “y” or “1” uses the y dimension input shape as the filter sizes. (The example uses the input x dimension shape as the convolutional filter sizes.) |


### APEX-Specific Parameters

In addition to the first (shared) table of parameters, APEX also supports these additional parameters.

| Parameter (Example use)      | Description |
| -                            | -           |
| exploration_decay => float <br> (`exploration_decay => 0.00001`) | The decay rate for the exploration policy. |
| q_learning_rate => float <br> (`q_learning_rate => 0.0001`) | The learning rate for training the Q network. |

### PPO-Specific

In addition to the first (shared) table of parameters, PPO supports these additional parameters.

| Parameter (Example use)      | Description |
| -                            | -           |
| policy_learning_rate => float <br> (`policy_learning_rate => 0.0001`) | The learning rate for training the policy network. |

