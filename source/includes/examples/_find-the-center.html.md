# Find the Center Example

In this example, we'll walk you through the various statements that are part of the Find the Center project, including the simulator file and the Inkling file. Find the Middle was created as a very basic example of Inkling and how to connect to a simulator.

The general premise of Find the Center is that the computer (via Inkling) takes a step (move) of -1, 0, or 1, and gets rewarded for staying in the center (1) for each round. The max is 2 and the min is 0.

## Inkling File

###### Schema

```inkling
schema GameState
    Int32 value
end
```

The `GameState` schema has one record, `value`, assigned the constrained type of Int32.

```inkling
schema PlayerMove
    Int32{-1, 0, 1} delta
end
```

The `PlayerMove` schema has one record, `delta`, assigned the constrained type of Int32 to allow for negative and positive values.

```inkling
schema SimConfig
    Int32 dummy
end
```

The `SimConfig` schema has one record, `dummy`, because there is no configuration needed in this particular example.

###### Concept

```inkling
concept find_the_center
    is classifier
    predicts (PlayerMove)
    follows input(GameState)
    feeds output
end
```

This concept is named `find_the_center`, and it takes input from the simulator about the state of the game (`GameState` schema). It outputs to the `PlayerMove` schema. This is the AI's next move in the simulation.

###### Simulator

```inkling
simulator find_the_center_sim(SimConfig)
    action (PlayerMove)
    state (GameState)
end
```

The `simulator` is called `find_the_center_sim` (shown in #simulator-file) and takes the schema input of `SimConfig` (even though it isn't configuring anything, it's required by the simulator). The simulator is going to output the `action` and the `state` of the simulator which is sent to the lesson.

###### Curriculum

```inkling
curriculum find_the_center_curriculum
    train find_the_center
    with simulator find_the_center_sim
    objective time_at_goal
        lesson seek_center
            configure
                constrain dummy with Int32{-1}
            until
                maximize time_at_goal
end
```

The curriculum is named `find_the_center_curriculum`, and it trains the `find_the_center` concept using the `find_the_center_sim`. This curriculum contains one lesson, called `seek_center`. It configures the simulation, by setting a number of constraints for the state of the simulator.

The lesson trains until the AI has maximized the objective `time_at_goal`.


## Simulator File

```python
import bonsai
import sys
from bonsai.simulator import SimState
from random import randint

""" If you would like to debug your code add this back in.
def debug(*args):
    print(*args, file=sys.stderr)
"""


class BasicSimulator(bonsai.Simulator):
    """ A basic simulator class that takes in a move from the inkling file,
    and returns the state as a result of that move.
    """

    min = 0
    max = 2
    goal = 1

    def __init__(self):
        super(self.__class__, self).__init__()
        self.goal_count = 0
        self.value = randint(self.min, self.max)
        self.old_value = self.min

    def get_terminal(self):
        """ Function to restart the simulation if the Inkling move was out of bounds.
        """
        if (self.value < self.min or self.value > self.max):
            # (self.value == self.goal and self.old_value == self.goal)):

            # debug("terminal")
            self.reset()
            return True
        else:
            return False

    def start(self):
        """ Function to start the episode by guessing a random integer between
        the min and max.
        """
        # debug("start")
        self.goal_count = 0
        self.old_value = self.min
        self.value = randint(self.min, self.max)

    def stop(self):
        """ Function to stop the simulator.
        """
        # debug("stop")
        pass

    def reset(self):
        """ Function to reset the simulation variables.
        """
        # debug("reset")
        self.goal_count = 0
        self.old_value = self.min
        self.value = randint(self.min, self.max)

    def advance(self, actions):
        """ Function to make a move based on input from Inkling file and
        if in the center, increases the total goal count by 1.
        """
        # debug("advance", actions["delta"])
        self.value += actions["delta"]
        if self.value == self.goal:
            self.goal_count += 1

    def get_state(self):
        """ Gets the state of the simulator, whether it be a valid value or terminal.
        """
        # debug("get_state")
        # debug("state", self.value)
        self.old_value = self.value
        return SimState(state={"value": self.value},
                        is_terminal=self.get_terminal())

    def distance_from_goal(self):
        """ Function to determine how far away the move is from the center.
        """
        dist = abs(self.goal - self.value)
        # debug("dist", dist)
        return -1*dist

    def time_at_goal(self):
        """ Function to return how long the simulation has maintained the goal.
        """
        return self.goal_count

if __name__ == "__main__":
    base_args = bonsai.parse_base_arguments()
    sim = BasicSimulator()
    bonsai.run_with_url('find_the_center_sim', sim,
                        base_args.brain_url, base_args.access_key)
```

This is a Basic simulator for learning the simulator interface. In this case it is used to find the center between two numbers, 0 and 2. The goal, as outlined in the Inkling file, is to reach 1. The moves that the simulator is able to make are sent from the Inkling file to the simulator and the state of the simulator is sent back to Inkling.

[1]: https://gym.openai.com/envs/MountainCar-v0
