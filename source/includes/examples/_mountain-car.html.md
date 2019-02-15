# OpenAI Gym: Mountain Car

> ![Mountain Car Control](../images/mountain-car-control.gif)

[**Download the full source code on GitHub**][1] if you want to run this simulator locally. If you want to run Mountain Car remotely on the Bonsai Platform as a managed simulator, create a new BRAIN selecting the Mountain Car demo on [beta.bons.ai][4]

We've used pieces of code from this example in several places, but here we'll walk you through all the various statements that are part of the Mountain Car Inkling file. Each statement is followed by an explanation of the statement.

Mountain Car is a classic control problem. [OpenAI Gym][2] describes it as:

_A car is on a one-dimensional track, positioned between two "mountains". The goal is to drive up the mountain on the right; however, the car's engine is not strong enough to scale the mountain in a single pass. Therefore, the only way to succeed is to drive back and forth to build up momentum._

## Inkling File

###### Types

```inkling2
type GameState {
    x_position: number,
    x_velocity: number
}
```

The `GameState` type defines two fields — `x_position` and `y_position`.

```inkling2
const throttleMin = -1.0
const throttleMax = 1.0
type Action {
    command: number<throttleMin .. throttleMax>
}
```

The `Action` type defines a single field — `command` — and assigns it a constrained type.

```inkling2
type MountainCarConfig {
    deque_size: Number.UInt8
}
```

The `MountainCarConfig` type defines a single field - `deque_size`.

###### Concept Graph

```inkling2
graph (input: GameState): Action {
    concept high_score(input): Action {
        curriculum {
            source mountaincar_continuous_simulator
            lesson get_high_score {
                constraint {
                    deque_size: 1
                }
            }
        }
    }
    output high_score
}
```

The output concept is named `high_score`, and it takes input from the simulator about the state of the game (which is of type `GameState`). It outputs a value of type `Action`. This is the AI's next move in the game.

The curriculum uses the simulator `mountaincar_simulator` as its data source. One lesson is specified, and it configures the simulation by constraining the deque_size field to a value of 1.


###### Simulator

```inkling2
simulator mountaincar_continuous_simulator(action: Action, config: MountainCarConfig): GameState {
}
```

The `mountaincar_continuous_simulator` receives actions of type `Action` and configuration information of type `MountainCarConfig`. It outputs values of type `GameState`.

## Simulator File

```python
import sys
import numpy
import logging
from bonsai_ai import Brain, Config
from bonsai_gym import GymSimulator

log = logging.getLogger('gym_simulator')
log.setLevel(logging.DEBUG)


class MountainCarContinuous(GymSimulator):
    environment_name = 'MountainCarContinuous-v0'
    simulator_name = 'mountaincar_continuous_simulator'

    def gym_to_state(self, observation):
        state = {'x_position': observation[0],
                 'x_velocity': observation[1]}
        return state

    # As an Estimator, continuous mountaincar returns the command
    # as a numpy array.
    def action_to_gym(self, actions):
        # return actions['command']
        return numpy.asarray([actions['command']])


if __name__ == '__main__':
    # create a brain, openai-gym environment, and simulator
    config = Config(sys.argv)
    brain = Brain(config)
    sim = MountainCarContinuous(brain)
    sim.run_gym()
```

This is an OpenAI Gym example which uses the OpenAI environment as its simulator. For more information about the simulator used see the [Bonsai Gym Common GitHub repo][3] which is a python library for integrating a Bonsai BRAIN with OpenAI Gym environments.

[1]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/samples/openai-gym/gym-mountaincar-continuous-sample
[2]: https://gym.openai.com/envs/MountainCarContinuous-v0/
[3]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/bonsai-gym
[4]: https://beta.bons.ai/new

