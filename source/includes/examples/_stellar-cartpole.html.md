# Stellar Cartpole

> ![Cartpole Balance](../images/cart-pole-balance.gif)

[**Download the full source code on GitHub**][1] if you want to run this simulator locally. If you want to run Cartpole remotely on the Bonsai Platform as a managed simulator, create a new BRAIN selecting the Cartpole demo on [beta.bons.ai][3].

This version of Cartpole expands on the OpenAI gym version of cartpole and exposes machine teaching logic and the rendering modeled by the classic cart-pole system implemented by Rich Sutton et al. This example will showcase the simulator file, connector file, and machine teaching file to explain how each piece is used in conjunction to train a BRAIN.

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

The type `GameState` defines four fields: `position`, `velocity`, `angle`, and `rotation`. This state information is provided by the simulation.

```inkling2
type Action {
    command: Number.Int8<left = 0, right = 1>
}
```

The type `Action` defines a single field `command` and assigns it a constrained type.

```inkling2
type CartPoleConfig {
    episode_length: Number.Int8,
    deque_size: Number.UInt8
}
```

 The type `CartPoleConfig` defines two fields: `episode_length` and `deque_size`. `episode_length` is a signed `Int8` because -1 is used for "run until pole drops".


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

The concept is named `balance`, and it takes input from the simulator of type `GameState`. The balance concept outputs the move the AI should make in the simulator. The output is of type `Action`.

###### Simulator

```inkling2
simulator cartpole_simulator(action: Action, config: CartPoleConfig): GameState {
}
```

Simulator `cartpole_simulator` receives an action of type `Action` and configuration information of type `CartPoleConfig`. It returns a value of type `GameState` that indicates the state of the game after the action is performed.

## Simulator File

> cartpole.py

```python
import math
import numpy as np
import seeding
from bonsai_ai.logger import Logger

log = Logger()

class CartPoleModel():

    def __init__(self):
        self.gravity = 9.8
        self.masscart = 1.0
        self.masspole = 0.1
        self.total_mass = (self.masspole + self.masscart)
        self.length = 0.5  # actually half the pole's length
        self.polemass_length = (self.masspole * self.length)
        self.force_mag = 10.0
        self.tau = 0.02  # seconds between state updates

        # Angle at which to fail the episode
        self.theta_threshold_radians = 12 * 2 * math.pi / 360
        self.x_threshold = 2.4

        self._seed()
        self.viewer = None
        self.state = None

        self.steps_beyond_done = None

    def _seed(self, seed=None):
        self.np_random, seed = seeding.np_random(seed)
        return [seed]

    def _step(self, action):
        state = self.state
        x, x_dot, theta, theta_dot = state
        force = self.force_mag if action == 1 else -self.force_mag
        costheta = math.cos(theta)
        sintheta = math.sin(theta)
        temp = (
            (force + self.polemass_length * theta_dot * theta_dot * sintheta)
            / self.total_mass)
        thetaacc = (
            (self.gravity * sintheta - costheta * temp)
            / (self.length *
               (4.0/3.0 - self.masspole * costheta * costheta /
                self.total_mass)))
        xacc = (
            temp - self.polemass_length * thetaacc * costheta / self.total_mass
        )
        x = x + self.tau * x_dot
        x_dot = x_dot + self.tau * xacc
        theta = theta + self.tau * theta_dot
        theta_dot = theta_dot + self.tau * thetaacc
        self.state = (x, x_dot, theta, theta_dot)
        done = x < -self.x_threshold \
            or x > self.x_threshold \
            or theta < -self.theta_threshold_radians \
            or theta > self.theta_threshold_radians
        done = bool(done)

        if not done:
            reward = 1.0
        elif self.steps_beyond_done is None:
            # Pole just fell!
            self.steps_beyond_done = 0
            reward = 1.0
        else:
            if self.steps_beyond_done == 0:
                log.warning("You are calling 'step()' even though this \
                environment has already returned done = True. You should \
                always call 'reset()' once you receive 'done = True' -- \
                any further steps are undefined behavior.")
            self.steps_beyond_done += 1
            reward = 0.0

        return (np.array(self.state),
                reward,
                done,
                {"steps_beyond_done": self.steps_beyond_done})

    def _reset(self):
        self.state = self.np_random.uniform(low=-0.05, high=0.05, size=(4,))
        self.steps_beyond_done = None
        return np.array(self.state)
```

This simulator file is derived from OpenAI Gym's cartpole environment, but re-factored to have minimal dependency, and rendering factored into a separate file, `render.py`. This file contains the actual simulation of the pole balancing on the cart.

## Connector File

> bridge.py

```python
import sys
import bonsai_ai
import star
from cartpole import CartPoleModel
from render import _renderer

log = bonsai_ai.logger.Logger()

star = star.Star()
model = CartPoleModel()
the_renderer = None


class ModelConnector(bonsai_ai.Simulator):
    """ A basic simulator class that takes in a move from the inkling file,
    and returns the state as a result of that move.
    """
    info = {}
    started = False
    reward = None
    terminal = None

    def __init__(self, brain, name, config):
        super(ModelConnector, self).__init__(brain, name)

    def episode_start(self, parameters=None):
        """ called at the start of every episode. should
        reset the simulation and return the initial state
        """
        log.info('Episode {} Starting'.format(self.episode_count))
        state = model._reset()
        state = star.reset(state)
        # print out a message for our first episode
        if not self.started:
            self.started = True
            print('started.')

        return state

    def simulate(self, brain_action):
        """ run a single step of the simulation.
        if the simulation has reached a terminal state, mark it as such.
        """
        action = star.action(brain_action)
        (model_state,
         model_reward,
         model_terminal,
         model_info) = model._step(action)
        self.terminal = star.terminal(model_state)
        self.info = model_info
        self.reward = star.reward(model_terminal)
        brain_state = star.state(model_state)

        if the_renderer is not None:
            the_renderer._render()

        return (brain_state, self.reward, self.terminal)

    def get_terminal(self):
        return self.terminal


if __name__ == "__main__":
    config = bonsai_ai.Config(sys.argv)
    brain = bonsai_ai.Brain(config)
    bridge = ModelConnector(brain, 'the_simulator', config)

    if '--render' in sys.argv:
        log.info('rendering')
        the_renderer = _renderer(model)

    log.info('starting simulation...')
    while bridge.run():
        continue
```

This file contains all of the connection code, or Python bridge code, to connect the model with the Bonsai Platform. This code facilitates the training of the BRAIN.

## Machine Teaching File

> star.py

```python
import math
import random
from bonsai_ai.logger import Logger

log = Logger()

class Star():
    iteration_count = 0
    model_state = None
    steps_beyond_done = None

    def __init__(self):
        # Angle at which to fail the episode
        self.theta_threshold_radians = 12 * 2 * math.pi / 360
        self.x_threshold = 2.4

    def state(self, model_state):
        """ This function converts the simulator state into the state
        representation the brain uses to learn. """
        self.model_state = model_state
        brain_state = {
            'position': model_state[0],
            'velocity': model_state[1],
            'angle': model_state[2],
            'rotation': model_state[3],
        }
        print(brain_state)
        return brain_state

    def terminal(self, model_state):
        """ Terminal conditions specify when to end an episode, typically due
        to success, failure, or running out of time. In this case, we only
        terminate when the pole falls down. """
        print(model_state)
        x, x_dot, theta, theta_dot = model_state
        done = (x < -self.x_threshold or
                x > self.x_threshold or
                theta < -self.theta_threshold_radians or
                theta > self.theta_threshold_radians)
        done = bool(done)
        return done

    def action(self, brain_action):
        """ This function converts the action representation the brain learns
        to use into the action representation the simulator uses to act in the
        simulated environment. """
        self.iteration_count += 1
        model_action = 0
        if brain_action['command'] > 0:
            model_action = 1
        return model_action

    def reward(self, done):
        """ Give the brain feedback on how well it is doing in this episode.
        In this case, this is simply 1 every time period that the pole is
        balanced. The brain's job is to learn to maximize the reward it gets
        during the episode, which corresponds to balancing as long as possible.
        """
        if not done:
            reward = 1.0
        elif self.steps_beyond_done is None:
            # Pole just fell!
            self.steps_beyond_done = 0
            reward = 1.0
        else:
            if self.steps_beyond_done == 0:
                log.info("You are calling 'step()' even though this \
                         environment has already returned done = True. \
                         You should always call 'reset()' once you receive \
                         'done = True' -- any further steps are undefined \
                         behavior.")
            self.steps_beyond_done += 1
            reward = 0.0
        return reward

    def reset(self, model_state):
        self.iteration_count = 0
        brain_state = self.state(model_state)
        self.steps_beyond_done = None

        return brain_state
```

This file contains the machine teaching logic, refered to as STAR, which stands for state, terminal, action, and reward functions. Through experimentation this is the code you will be changing to speed up training time or refine your reward function.

[1]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/samples/openai-gym/gym-cartpole-sample
[2]: https://gym.openai.com/envs/CartPole-v1
[3]: https://beta.bons.ai/new
