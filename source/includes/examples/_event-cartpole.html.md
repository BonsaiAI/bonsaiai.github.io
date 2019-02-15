# Event Pump Cartpole

> ![Cartpole Balance](../images/cart-pole-balance.gif)

[**Download the full source code on GitHub**][1] if you want to run this simulator.

This example demonstrates how instances of the `Event` class can be used to drive a custom simulation loop. Rather than relying on machinery internal to `bonsai_ai.Simulator`, the `CartPoleTrainer` class advances the simulation event by event, which, in turn, allows for custom logic at each stage.

This event-driven interface is useful when you can't squeeze your particular application logic into the prescribed callback interface.

Cartpole is a classic control problem. [OpenAI Gym][2] describes it as:

_A pole is attached by an un-actuated joint to a cart, which moves along a frictionless track. The system is controlled by applying a force of +1 or -1 to the cart. The pendulum starts upright, and the goal is to prevent it from falling over. A reward of +1 is provided for every timestep that the pole remains upright. The episode ends when the pole is more than 15 degrees from vertical, or the cart moves more than 2.4 units from the center._

## Inkling File

The Inkling for this example is exactly the same as the original [Cartpole OpenAI Gym example][4] above. See the previous example for details on how the Inkling works for training cartpole.


## Simulator File

> cartpole_simulator.py

```python
import gym
import sys
from bonsai_ai import Brain, Config
from bonsai_gym import GymSimulator
from bonsai_ai.logger import Logger
from bonsai_ai import EpisodeStartEvent, SimulateEvent, \
    EpisodeFinishEvent, FinishedEvent, UnknownEvent

from star import state, terminal, action, reward, params

log = Logger()


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

    def render(self):
        self._env.render()


class CartPoleTrainer(object):

    def __init__(self, sim):
        self._sim = sim
        self._episode_count = 0
        self._episode_reward = 0

    def run(self):
        event = self._sim.get_next_event()

        if isinstance(event, EpisodeStartEvent):
            log.event("Episode Start")
            observation = self._sim.gym_episode_start(params(event))
            event.initial_state = state(observation)

        elif isinstance(event, SimulateEvent):
            log.event("Simulate")
            obs, rwd, done, _ = self._sim.gym_simulate(action(event))
            event.state = state(obs)
            event.reward = reward(rwd)
            event.terminal = terminal(done)
            self._episode_reward += rwd
            self._sim.render()

        elif isinstance(event, EpisodeFinishEvent):
            log.event("Episode Finish")
            print("Episode {} reward: {}".format(
                self._episode_count, self._episode_reward))
            self._episode_count += 1
            self._episode_reward = 0

        elif isinstance(event, FinishedEvent):
            log.event("Finished")
            return False
        elif event is None:
            return False

        return True


if __name__ == '__main__':
    # create a brain, openai-gym environment, and simulator
    config = Config(sys.argv)
    brain = Brain(config)
    sim = CartPole(brain)
    trainer = CartPoleTrainer(sim)
    while trainer.run():
        pass

```

This is an OpenAI Gym example which uses the OpenAI environment as its simulator. For more information about the simulator used see the [Bonsai Gym][3] folder in the bonsai-sdk GitHub repo which is a python library for integrating a Bonsai BRAIN with OpenAI Gym environments.

You will notice that unlike the original cartpole example, this example imports `EpisodeStartEvent`, `SimulateEvent`, `EpisodeFinishEvent`, `FinishedEvent`, `UnknownEvent` from the `bonsai-ai` library. These classes are subclasses of the [Event class][5] which drive the simulation event by event, allowing for custom logic at each stage.

## Machine Teaching File

> star.py

```python
def state(observation):
    return {'position': observation[0],
            'velocity': observation[1],
            'angle':    observation[2],
            'rotation': observation[3]}


def action(event):
    return event.action['command']


# included for completeness w.r.t. the "STAR" pattern
def terminal(val):
    return bool(val)


# included for completeness w.r.t. the "STAR" pattern
def reward(val):
    return float(val)


def params(event):
    return event.initial_properties
```

This `star.py` file houses the machine teaching aspects of the Bonsai Platform in a single file, instead of keeping them combined with the rest of the simulator code, or hiding inside of the Gym Common library (which is where it is in the original Cartpole file). This is a useful tool when experimenting with different reward functions, terminal conditions, or other aspects of machine teaching.

You will notice that the terminal conditions and reward function are still maintained within the default gym environment for Cartpole, but they have been included here for completeness.


[1]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/samples/openai-gym/gym-cartpole-event-sample
[2]: https://gym.openai.com/envs/CartPole-v1
[3]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/bonsai-gym
[4]: #openai-gym-cartpole
[5]: ../references/library-reference.html#event-class