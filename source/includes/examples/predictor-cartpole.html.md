# Predictor Class Cartpole

> ![Cartpole Balance](../images/cart-pole-balance.gif)

[**Download the full source code on GitHub**][1] if you want to run this simulator.

This example demonstrates how the `Predictor` class can be used to obtain predictions from a trained BRAIN. This is an alternative method to using the CLI to obtain predictions using the `--predict` flag with Bonsai's Python SDK.

Cartpole is a classic control problem. [OpenAI Gym][2] describes it as:

_A pole is attached by an un-actuated joint to a cart, which moves along a frictionless track. The system is controlled by applying a force of +1 or -1 to the cart. The pendulum starts upright, and the goal is to prevent it from falling over. A reward of +1 is provided for every timestep that the pole remains upright. The episode ends when the pole is more than 15 degrees from vertical, or the cart moves more than 2.4 units from the center._

## Inkling File

The Inkling for this example is exactly the same as the original [Cartpole OpenAI Gym example][4] above. See the previous example for details on how the Inkling works for training cartpole.


## Simulator File

> cartpole_simulator.py

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
    # Example Inkling:
    #   curriculum balance_curriculum
    #       train balance
    #       with simulator cartpole_simulator
    #       ....
    simulator_name = 'cartpole_simulator'

    # convert openai gym observation to our state type
    # Example Inkling:
    #   type GameState {
    #       position: number,
    #       velocity: number,
    #       angle: number,
    #       rotation: number
    #   }
    def gym_to_state(self, observation):
        state = {'position': observation[0],
                 'velocity': observation[1],
                 'angle':    observation[2],
                 'rotation': observation[3]}
        return state

    # convert our action type into openai gym action
    # Example Inkling:
    #   type Action {
    #       command: number<0, 1>
    #   }
    def action_to_gym(self, action):
        return action['command']


if __name__ == '__main__':
    # create a brain, openai-gym environment, and simulator
    config = Config(sys.argv)
    brain = Brain(config)
    sim = CartPole(brain)
    sim.run_gym()

```

This is an OpenAI Gym example which uses the OpenAI environment as its simulator. For more information about the simulator used see the [Bonsai Gym][3] folder in the bonsai-sdk GitHub repo which is a python library for integrating a Bonsai BRAIN with OpenAI Gym environments.

## Predictor File

> cartpole_predictor.py

```python
import sys
import gym
from bonsai_ai import Brain, Config, Predictor
from bonsai_ai.logger import Logger

log = Logger()
log.set_enabled("info")


def _state(observation):
    """ Converts gym observation into Inkling state dictionary """
    state = {'position': observation[0],
             'velocity': observation[1],
             'angle':    observation[2],
             'rotation': observation[3]}
    return state


def _action(action):
    """ Converts Inkling action into a gym action """
    return action['command']


def _log_state_and_action(state, action):
    log.info("The BRAIN received the following state: {}".format(state))
    log.info("The BRAIN returned the following action: {}".format(action))


if __name__ == '__main__':
    # Set up predictor
    config = Config(sys.argv)
    brain = Brain(config)
    predictor = Predictor(brain, 'cartpole_simulator')

    # Set up cartpole simulator
    episode_count = 10
    env = gym.make('CartPole-v0')

    # Reset, get state, exchange state for action, and then step the sim
    observation = env.reset()
    state = _state(observation)
    action = _action(predictor.get_action(state))
    _log_state_and_action(state, action)
    observation, reward, done, info = env.step(action)
    env.render()

    # Loop until episode_count is reached
    while episode_count:
        state = _state(observation)
        action = _action(predictor.get_action(state))
        _log_state_and_action(state, action)
        observation, reward, done, info = env.step(action)
        env.render()

        if done:
            episode_count -= 1
            observation = env.reset()

```

This `cartpole_predictor.py` file contains all the code necessary to obtain predictions from a trained BRAIN. This predictor file converts the action and state spaces from the cartpole simulator (this is the same simulator as the original OpenAI Gym example) and logs the prediction results.


[1]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/samples/openai-gym/gym-cartpole-predictor-sample
[2]: https://gym.openai.com/envs/CartPole-v1
[3]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/bonsai-gym
[4]: #openai-gym-cartpole
[5]: ../references/library-reference.html#predictor-class