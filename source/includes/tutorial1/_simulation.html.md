# Simulation Overview

The simulation we will be running in this tutorial is called Move a Point. Start by skimming over `move_a_point_sim.py` which is found within the **tutorial1** folder of **bonsai-tutorials**. The simulation, in the `PointSimulation` class, models the problem described in the introduction -- in each episode (started by a call to `reset()`), an agent starts in a location (`current`), and has to get within `PRECISION` of a target point `target`. At each step of the simulation, the agent moves a distance `STEP_SIZE` in a specified direction.

The simulation ends (i.e., `game_over()` returns `True`) when the agent is close enough to the target. 

The `PointSimulation` class is a simple example of a common integration pattern: some often pre-existing code simulates a process, and we write additional code to control the simulation, read out the state, and reset periodically. Next, let's review how to connect a simulation to a BRAIN using the Bonsai SDK (Software Development Kit).

## Using the Bonsai SDK to connect to the BRAIN

```python
import bonsai_ai

class PointBonsaiBridge(bonsai_ai.Simulator):
```

The integration of the simulation with the Bonsai SDK includes three pieces. First, take a look at how we import the `bonsai_ai` module and derive the simulator class from `bonsai_ai.Simulator`.

Implementing the `Simulator` interface requires coding two functions, `episode_start()` and `simulate()`.

> Implementing the Simulator interface

```python
def episode_start(self, parameters=None):
    """Set up the simulation for a new episode. Returns the initial state."""
    ...

def simulate(self, action):
    """Given an action, run one simulation iteration and return a tuple:
           (state, reward, is_terminal)""" 
    ...
```

The `action` parameter to `simulate` is a dictionary, with a key for each action variable defined in your Inkling code. In our case, this is `action["direction_radians"]`. The `state` returned is a dictionary with one key for each state variable defined in your Inkling. In our case, this is implemented by the code in `_get_state(self)`.

In addition to the "game over" condition specified by the simulation, our SDK connection bridge adds a maximum number of steps (`MAX_STEPS`) per episode in `_is_terminal()`. Having a time limit like this helps ensure that episodes end even if the agent keeps moving in the wrong direction.

The `episode_start` and `simulate` functions together define the action loop, which can be written as shown in the code panel.

> Action loop

```python
state = sim.episode_start()
is_terminal = False
while not is_terminal:
    action = policy(state)   # decide what to do next. When connected to a BRAIN, the BRAIN chooses the action.
    (state, reward, is_terminal) = sim.simulate(action)
    # Learning from the new state and the obtained reward happens here
```

Finally, we need to instantiate the simulator class and run it.

> Instantiate the Simulator class

```python
if __name__ == "__main__":
    config = bonsai_ai.Config(sys.argv)
    brain = bonsai_ai.Brain(config)
    sim = PointBonsaiBridge(brain, "move_a_point_sim")
    while sim.run():
        continue
```

The configuration includes the BRAIN name and url, your Bonsai API key, and whether to train the BRAIN or run it in prediction mode (sometimes called test mode). The name passed to the `PointBonsaiBridge` constructor, `"move_a_point_sim"` here, identifies the simulation file to the BRAIN, and must match your Inkling code (described in a later section).

## Run Your Simulator

To test the simulator, we can create a `PointSimulator` object and directly call `episode_start()` and `simulate()`, as described above. You can see an example of how to test the simulator before connecting it to the Bonsai Platform in [`test_simulator.ipynb`][1] (requires [Jupyter Notebook][2]), or in the [`test_sim.py`][2] Python script, both of which can be found within the tutorial1 folder.

> Episode loop

```python
def run_sim_episode(sim, policy):
    """
    Given a sim and a policy, step through some iterations 
    of the simulator, returning the history of states.
    
    Args:
        sim: a PointBonsaiBridge
        policy: a function (SimState -> action dictionary)
    """
    state_history = []
    reward_history = []
    state = sim.episode_start()
    state_history.append(state)

    is_terminal = False
    while not is_terminal:
        action = policy(state)
        (state, reward, is_terminal) = sim.simulate(action)
        state_history.append(state)
        reward_history.append(reward)

    return state_history, reward_history
```

The key code is the episode loop, shown in the code panel.

Then we can define some policies, defining what action to take for a given state. If you'd like to test with other silly policies or define your own, change line 56 in `test_sim.py`: `states, rewards = run_sim_episode(point_sim, random_policy`.

After that the code will run some episodes, plotting the results.

> Define policies

```python
# Some silly policies
def random_policy(state):
    """
    Ignore the state, move randomly.
    """
    return {'direction_radians': random.random() * 2 * math.pi}

def go_up_policy(state):
    return {'direction_radians': math.pi / 2.0}
```

> Run episode and plot results

```python
for i in range(3):
    states, rewards = run_sim_episode(point_sim, random_policy)
    plot_state_history(states)
```

When making or integrating simulations, it is always a good idea to run some sanity checks and verifications before starting BRAIN training.

#### Exercises

* Run the simulator via the above code, either via `test_sim.py`.
* Write two different policy functions for moving in different directions, and plot their behavior.

[1]: https://github.com/BonsaiAI/bonsai-tutorials/blob/master/tutorial1/test_simulator.ipynb
[2]: https://github.com/BonsaiAI/bonsai-tutorials/blob/master/tutorial1/test_sim.py
