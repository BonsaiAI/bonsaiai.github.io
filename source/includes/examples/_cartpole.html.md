# OpenAI Gym: Cartpole

> ![Cartpole Balance](../images/cart-pole-balance.gif)

[**Download the full source code on GitHub**][1] if you want to run this simulator locally.

<aside class="notice">
This example has now been replaced by the Stellar Cartpole example as the default Cartpole when you create a "Cartpole" demo on the platform.
</aside>

In this example, we'll walk you through the various statements that are part of the Cartpole Inkling file. Each statement is followed by an explanation of the statement.

Cartpole is a classic control problem. [OpenAI Gym][2] describes it as:

_A pole is attached by an un-actuated joint to a cart, which moves along a frictionless track. The system is controlled by applying a force of +1 or -1 to the cart. The pendulum starts upright, and the goal is to prevent it from falling over. A reward of +1 is provided for every timestep that the pole remains upright. The episode ends when the pole is more than 15 degrees from vertical, or the cart moves more than 2.4 units from the center._

## Inkling File

###### Types

```inkling2
type GameState {
    position: number,
    velocity: number,
    angle: number,
    rotation: number
}
```

The type `GameState` defines four fields - `position`, `velocity`, `angle`, and `rotation` — and assigns a type to each. This information is input from the simulation.

```inkling2
type Action {
    command: number<Left = 0, Right = 1>
}
```

The type `Action` defines a single flield — `action` —  and assigns it a constrained type.

```inkling2
type CartPoleConfig {
    episode_length: Number.Int8,
    deque_size: Number.UInt8
}
```

 The type `CartPoleConfig` defines two fields — `episode_length` and
 `deque_size` — and assigns each of them a type.  `episode_length` is a signed `Int8` because -1 is used for "run until pole drops".


###### Concept Graph

```inkling2
graph (input: GameState): Action {

    concept balance(input): Action {
        curriculum {
            source cartpole_simulator

            lesson balancing {
                constraint {
                    episode_length: -1,
                    deque_size: 1
                }
            }
        }
    }
    
    output balance
}
```

The concept is named `balance`, and it takes input from the simulator. That input is of type `GameState`. The balance concept outputs the move the AI should make in the simulator. This output is of type `Action`.

The curriculum defines one lesson, called `balancing`. It constrains the simulator's configuration such that the `episode_length` is -1 and the `deque_size` is 1 for all episodes.


###### Simulator

```inkling2
simulator cartpole_simulator(action: Action, config: CartPoleConfig): GameState {
}
```

Simulator `cartpole_simulator` receives inputs of two types. The first, `Action`, specifies the action that the AI will take in the simulation. The second, `CartPoleConfig`, specifies the configuration of the simulation. The simulator outputs a value of type `GameState` which contains the state of the simulator at each iteration.

## Simulator File

```python
import sys
import logging
from bonsai_ai import Brain, Config
from bonsai_gym import GymSimulator

log = logging.getLogger('gym_simulator')
log.setLevel(logging.DEBUG)


class CartPole(GymSimulator):
    # Environment name, from openai-gym
    environment_name = 'CartPole-v0'

    # simulator name from Inkling
    simulator_name = 'cartpole_simulator'

    # convert openai gym observation to our state type
    def gym_to_state(self, observation):
        state = {'position': observation[0],
                 'velocity': observation[1],
                 'angle':    observation[2],
                 'rotation': observation[3]}
        return state

    # convert our action type into openai gym action
    def action_to_gym(self, action):
        return action['command']


if __name__ == '__main__':
    # create a brain, openai-gym environment, and simulator
    config = Config(sys.argv)
    brain = Brain(config)
    sim = CartPole(brain)
    sim.run_gym()
```

This is an OpenAI Gym example which uses the OpenAI environment as its simulator. For more information about the simulator used see the [Bonsai Gym Common GitHub repo][3] which is a python library for integrating a Bonsai BRAIN with OpenAI Gym environments.

[1]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/samples/openai-gym/gym-cartpole-sample
[2]: https://gym.openai.com/envs/CartPole-v1
[3]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/bonsai-gym
[4]: https://beta.bons.ai/new
