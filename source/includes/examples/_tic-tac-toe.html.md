# Luminance Example

> ![Need an image]

[**Download the full source code on GitHub**][1] to run this simulator.

To demonstrate the use of Luminance in The Bonsai Platform this example will walk you through a simple implementation of tic-tac-toe. THe full Inkling file is displayed with we'll walk you through the various statements that are part of a sample implementation of [EnergyPlus][2] on the Bonsai Platform, including the simulator and the Inkling files. This is a real-world example of how to use the Bonsai Platform for HVAC control using BCVTB and EnergyPlus.

While this BRAIN is training, the Bonsai AI Engine launches the EnergyPlus simulator in the background for every episode. The *energyplus_simulator.py* then drives the simulator forward a step at a time until it finishes the episode and then relaunches it for the next episode, driving the actions into it and sending state results back to the Bonsai AI Engine.

This example also contains an advanced-level algorithm clause in its Inkling File. This clause is not required but it will help the example train much faster. For more information on use of the algorithm clause see the [Inkling Reference][5].

## Inkling File

###### Schema

```inkling
schema GameState
    Luminance(9, 9) image
end
```

The `SimState` schema defines the dictionary returned from the Python simulation's `advance` method to the BRAIN.

```inkling
schema PlayerMove
    Int8{1, 2, 3, 4, 5, 6, 7, 8, 9} move
end
```

The `SimAction` schema defines the 'actions', a dictionary of control signals this AI can send to the climate control. For example: `shade` == night, off, day.

```inkling
schema DummyConfig
    Int8 dummy
end
```

The `SimConfig` schema in this case is not used (but is still required to be defined in Inkling) but it would define the dictionary passed as a parameter to the `set_properties` method of the Python simulator.

###### Concept

```inkling
concept play_tictactoe is classifier
    predicts (PlayerMove)
    follows input(GameState)
    feeds output
end
```

This concept is named `my_concept` which predicts a `SimAction` given a `SimState`. In this simple demo we just ask the Bonsai Platform to generate any model that can learn to control the server using these inputs and outputs.

###### Simulator

```inkling
simulator tictactoe_simulator(DummyConfig)
  action  (PlayerMove)
  state  (GameState)
end
```

The simulator clause declares that a simulator named `energyplus_simulator` will be connecting to the server for training. This code snippet binds the previous schemas to this simulator. To define the training relationship between the simulator and the concept we must begin by defining the simulator. `energyplus_simulator` expects an action defined in the `SimAction` schema as input and replies with a state defined in the `SimState` schema as output.

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

blah blah

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

The curriculum `my_curriculum` trains `my_concept` using `energyplus_simulator`. The BRAIN that runs this Inkling code will try to maximize the value returned from `reward_function` until you stop training. `reward_function` is a method in the Python simulator.

This curriculum contains one lesson, called `my_first_lesson`. It configures the simulation, by setting a number of constraints for the state of the simulator.

## Simulator File

The full simulator file *energyplus_simulator.py* for this example is too long to display in full here but you can see it with the rest of the [luminance-sample code][1] on GitHub.

This is a Python simulator for integrating the EnergyPlus simulator into the Bonsai AI Engine. This *energyplus_simulator.py* file repeatedly runs the EnergyPlus simulator in the background with new actions sent from the Bonsai AI Engine by passing the state from EnergyPlus to the backend, and the action from the backend back to EnergyPlus.

For more information on the functions inside of this simulator class and how to implement them see the [Library Reference][3].

[1]: link needed
[2]: https://energyplus.net/
[3]: ./../references/library-reference.html
[4]: https://beta.bons.ai/new
[5]: ./../references/inkling-reference.html#advanced-algorithms
