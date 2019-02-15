# Househeat Example

> Inkling code for Househeat

```inkling2
inkling "2.0"

type HouseheatState {
    heat_cost: number,
    temperature_difference: number,
    temperature_difference_t1: number,
    temperature_difference_t2: number,
    temperature_difference_t3: number,
    temperature_difference_t4: number,
    temperature_difference_t5: number,
    outside_temp_change: number
}

type HouseheatAction {
    heater_on: number<0 .. 1 step 1>
}

type HouseheatConfig {
    outside_phase: number
}

simulator simulink_sim(action: HouseheatAction, config: HouseheatConfig): HouseheatState {
}

graph (input: HouseheatState): HouseheatAction {
    concept thermostat(input): HouseheatAction {
        curriculum {
            source simulink_sim
            lesson my_first_lesson {
                constraint {
                    outside_phase: number<0 .. 12>
                }
            }
            lesson my_second_lesson {
                constraint {
                    outside_phase: number<0 .. 24>
                }
            }
            lesson my_third_lesson {
                constraint {
                    outside_phase: number<0 .. 48>
                }
            }
        }
    }
    output thermostat
}
```

> Python code for Househeat

```python
#!/usr/bin/env python3

import logging
import math
from collections import deque

_STEPLIMIT = 480

class Model:
    def load(self, eng):
        """
        Load the specified simulink model.
        """
        eng.eval("load_system('simulink_househeat')", nargout=0)

    def executable_name(self):
        """
        Returns the name of the executable (Simulink Coder Only).
        """
        return "./simulink_househeat"

    def episode_init(self):
        """
        This method is called at the beginning of each episode.
        """
        self.nsteps = 0
        self.action = None
        self.state = None
        self.logged_state = None
        self.reward = None
        self.terminal = None
        self.total_reward = 0.0
        empty_observation = [0.0,0.0,0.0,0.0,0.0]
        self.temperature_difference_history = deque(empty_observation)

    def episode_step(self):
        """
        This method is called at the beginning of each iteration.
        """
        self.nsteps += 1
        
    def convert_config(self, conf):
        """
        Convert the dictionary of config from the brain into an ordered
        list of config for the simulation.
        """
        if len(conf) == 0:
            conf['outside_phase'] = 0.0
        return [ conf['outside_phase'], ]

    def convert_input(self, simulator_state):
        """
        Called with a list of inputs from the model,
        returns (state, reward, terminal).
        """
        self.logged_state = {
            'heat_cost':           simulator_state[0],
            'set_temp':            simulator_state[1],
            'room_temp':           simulator_state[2],
            'room_temp_change':    simulator_state[3],
            'outside_temp':        simulator_state[4],
            'outside_temp_change': simulator_state[5],
        }

        tdiff = math.fabs(self.logged_state['set_temp'] - self.logged_state['room_temp'])
        self.temperature_difference_history.appendleft(tdiff)
        self.temperature_difference_history.pop()

        self.state = {
            'heat_cost':                 simulator_state[0],
            'temperature_difference':    tdiff,
            'temperature_difference_t1': self.temperature_difference_history[0],
            'temperature_difference_t2': self.temperature_difference_history[1],
            'temperature_difference_t3': self.temperature_difference_history[2],
            'temperature_difference_t4': self.temperature_difference_history[3],
            'temperature_difference_t5': self.temperature_difference_history[4],
            'outside_temp_change':       simulator_state[5],
        }

        self.tstamp = simulator_state[6]

        tdiff = math.fabs(self.state['set_temp'] - self.state['room_temp'])
        nonlinear_diff = pow(tdiff, 0.4)
        scaled_diff = nonlinear_diff / 1.32
        self.reward = 1.0 - scaled_diff

        self.terminal = self.nsteps >= _STEPLIMIT or self.reward < 0.0

        if self.nsteps > 0:
            self.total_reward += self.reward

        return self.state, self.reward, self.terminal

    def convert_output(self, brain_action):
        """
        Called with a dictionary of actions from the brain, returns an
        ordered list of outputs for the simulation model.
        """
        outlist = []
        if brain_action is not None:
            clamped_action = min(1, max(-1, brain_action['heater_on'])) # clamp to [-1,1] 
            scaled_action = (clamped_action +2)/2 + 1 # scale to [0,1]

            self.action = brain_action
            outlist = [ brain_action['heater_on'], ]

        return outlist

    def format_start(self):
        """
        Emit a formatted header and initial state line at the beginning
        of each episode.
        """
        logging.info(" itr  time h =>    cost  set   troom   droom tout dout = t    rwd")
        logging.info("                %7.1f %4.1f %7.1f %7.1f %4.1f %4.1f" % (
            self.logged_state['heat_cost'],
            self.logged_state['set_temp'],
            self.logged_state['room_temp'],
            self.logged_state['room_temp_change'],
            self.logged_state['outside_temp'],
            self.logged_state['outside_temp_change'],
        ))

    def format_step(self):
        """
        Emit a formatted line for each iteration.
        """
        if self.terminal:
            totrwdstr = " %6.3f" % self.total_reward
        else:
            totrwdstr = ""
            
        logging.info(" %3d %5.3f %1.0f => %7.1f %4.1f %7.1f %7.1f %4.1f %4.1f = %i %6.3f%s" % (
            self.nsteps,
            self.tstamp,
            self.action['heater_on'],
            self.logged_state['heat_cost'],
            self.logged_state['set_temp'],
            self.logged_state['room_temp'],
            self.logged_state['room_temp_change'],
            self.logged_state['outside_temp'],
            self.logged_state['outside_temp_change'],
            self.terminal,
            self.reward,
            totrwdstr,
        ))
```

![Househeat image](../images/househeat.png)

The rest of this guide will walk through example code and best practices of training a BRAIN to keep the internal temperature of a house as close as possible to a set point temperature. This system models the outdoor environment, the thermal characteristics of the house, and the house heating system. This is simulating the real-world example of a HVAC system in a home.

Before we dive into the STAR components of this system, let’s first look at the Inkling concepts and types that describe the problem at hand. These state and action types combine with the reward to comprise the feedback loop that the BRAIN will use to learn. We will use the transformations outlined below to reformat the simulator inputs and outputs to match the Inkling types that we present to the BRAIN.

After you've glanced at the Inkling code you can take a quick look at the Python code. This guide will walk through segments of it and the full code (with more code comments) can be referenced under [Simulink Househeat][1] on GitHub if you want the full context. Specifically, the code snippets describing machine teaching can be found in `simulink_househeat.ink` for Inkling and `bonsai_model.py` for Python.

Note that if you want to run this Simulink househeat example yourself you'll need to have MATLAB and Simulink installed on your computer. The guide for this is in our [Simulink Examples][4] section.


# Transforming State Spaces

When preparing to train a BRAIN on the Bonsai Platform, you might realize that you want to modify the state signals from the simulation model before passing it to the BRAIN. For example, you might not need all of the simulation model outputs for your BRAIN training.  As another example, you might want to perform calculations on the state variables from the model to create your state space for BRAIN training. Let’s discuss three common state transformations for machine teaching. 

## Creating a State History Buffer Using Deques

> Initialize

```python
empty_observation = [0.0,0.0,0.0,0.0,0.0]
self.temperature_difference_history = deque(empty_observation)
```

> Transform

```python
        self.state = {
            'heat_cost':                 simulator_state[0],
            'temperature_difference':    tdiff,
            'temperature_difference_t1': self.temperature_difference_history[0],
            'temperature_difference_t2': self.temperature_difference_history[1],
            'temperature_difference_t3': self.temperature_difference_history[2],
            'temperature_difference_t4': self.temperature_difference_history[3],
            'temperature_difference_t5': self.temperature_difference_history[4],
            'outside_temp_change':       simulator_state[5],
        }
```

> Update

```python
self.temperature_difference_history.appendleft(tdiff)
self.temperature_difference_history.pop()
```

We can give the BRAIN a “memory” of previous states that resulted from its actions by creating a history buffer of several previous values of important state variables.  This may result in more efficient training and more effective resulting policies, particularly for systems that need to be continuously controlled over time.   For example, if you are training a BRAIN to assist with vehicle control and your simple model includes only position of the vehicle, you may want to include a history buffer of position in your training state.  This helps the BRAIN learn the dynamics of velocity and acceleration of the vehicle.

The way you would do this is to:

1. Initialize the a deque with average values for the state variables at the beginning of each episode.
2. Then, transform the simulation outputs into a state dictionary for the BRAIN.
3. Last, update the deque with new state values returned from the simulator.

## Converting and Combining State Variables

Sometimes the signals from your simulation model are not exactly what you want to pass to the BRAIN for training.  Here is our example of househeat (with code) and another example of a hybrid vehicle system so you can start to think about how to apply these concepts elsewhere of how to convert and combine state variables:

> HVAC example Python

```python
tdiff = math.fabs(self.logged_state['set_temp'] - self.logged_state['room_temp'])
self.temperature_difference_history.appendleft(tdiff)
self.temperature_difference_history.pop()
```

* HVAC:  In our example where we train a BRAIN to control a heating system, we receive the thermostat set temperature and the room temperature at each time step from the simulation model. In this case, we use the absolute value of the current room temperature minus the thermostat set temperature instead of both the set temperature and the room temperature in the state.  After calculating the temperature difference, we add it to a history buffer of the temperature difference at the five most recent previous time steps. 

* Hybrid Vehicle System:  As another example, if you are training a BRAIN to decide when to store energy and when to discharge energy from the battery in a hybrid vehicle system, the simulation model may provide the vehicle power requirement and the engine available power at each time step.  In order to decide when to discharge the battery, the BRAIN needs to know whether the engine needs a boost or not.  In this case, we return 'true' to the BRAIN if the power required exceeds the available power from the engine. 

## Normalizing State Variables

Another common state transformation is to normalize all state variables into a similar range.   This helps the learning algorithm modify the neural network weights more efficiently as it learns.  For example, you  might want to perform a state transformation to convert Celsius degrees to Fahrenheit degrees.  As another example, you might want to convert simulator time to unix time.


# Transforming Action Spaces

Action spaces comprise the complete list of actions that you are training the BRAIN to control.   Discrete action spaces contain a defined list of possible actions.   For example, the transmission in a car is a discrete action space.  At any given time, the gear shift specifies one of several gear ratio settings to the transmission.   Continuous action spaces contain an infinite number of possible actions within a range.  For example, a slider control for indoor lighting works in a continuous action space.

## Clamping and Scaling Continuous Action Spaces

> Example Python for clamping and scaling

```python
def convert_output(self, brain_action):
        """
        Called with a dictionary of actions from the brain, returns an
        ordered list of outputs for the simulation model.
        """
        outlist = []
        if brain_action is not None:

            # If you need to clamp the action, for example because the TRPO algorithm does not clamp automatically.
            clamped_action = min(1, max(-1, brain_action['heater_on'])) # clamp to [-1,1] 

            # If you need to scale the action, for example to to [0,1]
            scaled_action = (clamped_action +2)/2 + 1 # scale to [0,1]

            outlist = [ brain_action['heater_on'], ]

        return outlist

```

At times, you might want to use a Deep Reinforcement Learning (DRL) algorithm to train a BRAIN on an action space that is discrete.  For example, you might choose the Trust Region Policy Optimization (TRPO) algorithm to train this smart thermostat BRAIN because TRPO tends to train well for large time-dependent state spaces.  

TRPO produces continuous actions (any number from -1 to 1), but in this case the system controls are discrete actions (0 for heater off, 1 for heater on).  You need to convert each action from the BRAIN to a control action for the heater.  The Python function below clamps and scales BRAIN actions that were generated by a DRL algorithm like TRPO.

## Simplifying the action spaces of real world systems

Let’s deviate from our HVAC example to illustrate another application of transforming action spaces.  In the same way that you might want to omit some model output variables from the BRAIN training state, you might want to omit or consolidate some control actions from the BRAIN training action space.  For example, if you are training a BRAIN to control a Baxter Robot to lift a table you might want to transform the action space.  The Baxter robot has six joints: two elbows, two wrists, two shoulders.  But when lifting a table both arms operate in unison, so we consolidate the action space to three commands instead of six, one for each set of joints. 

## Overriding BRAIN Actions

In some situations, you will want to override the actions that are coming from the BRAIN.  You can do this in the STAR.py file.  For example, you might want to run the simulator against a fixed policy to provide a benchmark for the learning algorithm.  For the smart thermostat, this might mean running overriding the BRAIN actions and running the simulator as simple relay (turn the heater on if the room temperature is five degrees below the set temperature, turn the heater off if the room temperature is five degrees above the set temperature), logging the results and recording the rewards for comparison against the BRAIN. 


# Constructing Reward Functions

> Househeat reward function Python code

```python
# To compute the reward function value we start by taking the
# difference between the set point temperature and the actual
# room temperature.
tdiff = math.fabs(self.state['set_temp'] - self.state['room_temp'])

# Raise the difference to the 0.4 power.  The non-linear
# function enhances the reward distribution near the desired
# temperature range.  Please refer to the Bonsai training
# video on reward functions for more details.
nonlinear_diff = pow(tdiff, 0.4)

# Scale the nonlinear difference so differences in the range
# +/- 2 degrees (C) map between 0 and 1.0.
# 2 degree ^ 0.4 = 1.32
scaled_diff = nonlinear_diff / 1.32

# Since we need a positive going reward function, subtract the
# scaled difference from 1.0.  This reward value will be 1.0
# when we are precisely matching the set point and will fall
# to less than 0.0 when we exceed 2 degrees (C) from the set
# point.
self.reward = 1.0 - scaled_diff

self.terminal = self.nsteps >= _STEPLIMIT or self.reward < 0.0

if self.nsteps > 0:
    self.total_reward += self.reward
```

The reward function of a model simply does just that. You are rewarding the system either negatively or positively for its actions. The AI will learn to do exactly what it’s rewarded for, nothing more and nothing less. So you need to be careful to give enough information the system to result in the desired behavior.

As mentioned in the first part of this guide, the house in this example is being heated by an AI to keep as close as possible to a set point temperature. When constructing a reward function, you will need to keep in mind this objective. Currently in the platform the reward function is constructed in the simulation model, not in Inkling.

This example uses both a time limit and a negative [terminal condition][2]. The episode will end if there has been more than 120 iterations (steps) or the reward has gone negative. 

## Sparse vs. Shaped Rewards

> Sparse Reward Function

```python
if tdiff < 5:
     self.reward = 1;
elif tdiff > 5:
     self.reward = 0
```

The simplest reward function is called a sparse reward. In a sparse reward, the BRAIN receives a reward once it accomplishes its objective, and it receives 0 reward otherwise. This type of reward is very easy to conceptualize and write, but is very inefficient when it comes to training time because of the lack of continuous information provided to the BRAIN.

In the Sparse Reward Function shown, if there is a temperature difference of less than 5 degrees from the set point temperature, the reward would be 1, and otherwise the temperature is not close enough to the desired set point and a reward of 0 is given.

> Shaped Reward Function

```python
tdiff = math.fabs(self.state['set_temp'] - self.state['room_temp'])
        nonlinear_diff = pow(tdiff, 0.4)
        scaled_diff = nonlinear_diff / 1.32
        self.reward = 1.0 - scaled_diff
```

Shaping a reward function is an important part of reducing training time. By providing information throughout the simulation (before the objective is reached) the BRAIN can learn the types of behaviors we want to see in the simulation. For example, if you know that being closer to a set temperature is better than being farther away you can use that information to provide a smooth continuous gradient that says ‘for any given state, if that state is closer to the objective, then it should give more reward than a state that's further away from the objective. 

In this example, we have created an exponential function to shape the reward function. This function will reward close to 1 when the difference in temperature is close to 0 and has a curve to encourage the BRAIN to get closer to 0 when it’s farther away.

Problems that amenable to shaping are problems where it’s easy to say for any given state whether it’s better or worse. In the game Go for example, it’s very hard to determine whether one move is better or worse than any other given move, and makes it very hard to solve with Reinforcement Learning.

For detailed information and examples on shaping reward functions you can watch our training video on [Writing Great Reward Functions][3].

## Terminating Episodes During BRAIN Training

One of the most important considerations in designing your simulation and setting up your reward function is deciding on your terminal conditions. These are conditions in which your simulation will reset back to the initial conditions and a new episode will begin. Terminal conditions help bound exploration and they avoid wasting time (if the BRAIN gets stuck or gets into a degenerate condition). If the temperature in a house is hot enough to boil the occupants, you probably don’t need the BRAIN to do much exploring in that state space.

There are three types of **terminal conditions**:

* Time limits
* Positive terminals
* Negative terminals

### Speeding up Training

**Time Limits**

While shaping a great reward function helps the BRAIN train faster, having smart terminal conditions speeds up the time in between episodes which helps you iterate faster on your model. Time limits for example are often useful for preventing that stuck condition if the BRAIN is executing on a model that is never going to succeed. It’s better to just end the episode and bring it back to initial conditions so it can train again in a useful state space.

**Positive Terminals**

Positive terminals are your success conditions. In these cases, the task has been finished, and you want to restart the simulation so the BRAIN gets another chance to practice from the beginning.

It is important to pair positive rewards with positive terminals to avoid a BRAIN from accumulating reward until it hits a time limit, instead of completing the objective. Making sure that any positive terminal you have rewards more in a single step than the AI could expect to receive were it to continue the episode as long as possible will prevent this.

**Negative Terminals**

Going back to the example of simulating temperature in a house, you would want to provide the simulation a negative terminal if the temperature in the house reached a level too high for the occupants to survive. Once the BRAIN has reached a state from which it cannot succeed or has wandered outside the bounds of the environment we want it to restart and try again.


# Constructing Lessons for BRAIN Training

Each Inkling file contains a lesson plan that will be used to train the BRAIN. You can create a phased training plan for your BRAIN by configuring your simulation model at the beginning of each episode.  

## Initializing Episodes

> Inkling for episode configuration

```inkling2
type HouseheatConfig {
    outside_phase: number
}

concept thermostat(input): HouseheatAction {
    curriculum {
        source simulink_sim

        lesson my_first_lesson {
            constraint {
                outside_phase: number<0 .. 12>
            }
        }
    }
}
```

The episode_start call passes in configuration parameters that are specified in Inkling. In the example shown, a randomized phase of the sinusoidal temperature variations is passed into an HVAC simulation from the Inkling lesson.  You can think of this as the outside temperature at different times of the day. 

At the beginning of each episode, the phase is passed into the STAR.py file and you can use it for things like reward calculations. 

> Python for episode start

```python
def episode_start(self, parameters):
      """ called at the start of every episode. should
    reset the simulation and return the initial state
    """
      self.initial_temperature = parameters['outside_phase']
```

## Generalizing Training

We can also train more robust policies and better transition from simulation to live equipment using simulator configurations. Randomizing initial conditions can force the BRAIN to learn the underlying dynamics of systems instead of overfitting to one particular scenario. This technique can also help to better adapt to live equipment which often experiences much more variation than the simulation model that the BRAIN was trained on.

> Temperature variation in Inkling lesson

```inkling2
lesson my_first_lesson {
    constraint {
        outside_phase: number<0 .. 12>
    }
}
```

For example, if you are training a BRAIN to control an HVAC system to control a chiller valve there may be error in the valve calibration.   When the BRAIN specifies a the valve to be opened to allow 40% of maximum water flow, the valve actually opens to 42% maximum water flow.  One way to mitigate this common miscalibration would be to set a noise variable in Inkling configuration parameter.  At the beginning of each episode, this variable will be set to a random value within a range of possible miscalibration.  This mimics the effect of valves that are miscalibrated and forces the BRAIN to learn to effectively control the HVAC system through a wide variety of miscalibrated valves.

In our code sample, the outside phase, which represents the temperature variation at different times of day is randomly varied.

## Constructing Lessons

> Inkling curriculum

```inkling2
concept thermostat(input): HouseheatAction {
    curriculum {
        source simulink_sim

        lesson my_first_lesson {
            constraint {
                outside_phase: number<0 .. 12>
            }
        }

        lesson my_second_lesson {
            constraint {
                outside_phase: number<0 .. 24>
            }
        }

        lesson my_third_lesson {
            constraint {
                outside_phase: number<0 .. 48>
            }
        }
    }
}
```

In this example we want to train a BRAIN to control a residential HVAC system.  We want the BRAIN to successfully respond to a wide variety of initial room temperatures.  We create an Inkling configuration for initial temperature, but we don’t want to vary the initial room temperature randomly because the range of initial temperatures is very wide. This may take the learning algorithm a long time to explore the solution space. 

Instead, we use a phased training approach called lessons. In the early episodes, the first lesson, we specify a `constraint` such that the initial temperature is within a narrow range. After the BRAIN learns to control the system well in the first range of initial temperatures, we increase the range of possible initial temperatures in the second lesson. Then we repeat the process for subsequent lessons. 


# Logging BRAIN Training Progress

> Python Logging

```python
def format_start(self):
        """
        Emit a formatted header and initial state line at the beginning
        of each episode.
        """
        logging.info(" itr  time h =>    cost  set   troom   droom tout dout = t    rwd")
        logging.info("                %7.1f %4.1f %7.1f %7.1f %4.1f %4.1f" % (
            self.logged_state['heat_cost'],
            self.logged_state['set_temp'],
            self.logged_state['room_temp'],
            self.logged_state['room_temp_change'],
            self.logged_state['outside_temp'],
            self.logged_state['outside_temp_change'],
        ))

    def format_step(self):
        """
        Emit a formatted line for each iteration.
        """
        if self.terminal:
            totrwdstr = " %6.3f" % self.total_reward
        else:
            totrwdstr = ""
            
        logging.info(" %3d %5.3f %1.0f => %7.1f %4.1f %7.1f %7.1f %4.1f %4.1f = %i %6.3f%s" % (
            self.nsteps,
            self.tstamp,
            self.action['heater_on'],
            self.logged_state['heat_cost'],
            self.logged_state['set_temp'],
            self.logged_state['room_temp'],
            self.logged_state['room_temp_change'],
            self.logged_state['outside_temp'],
            self.logged_state['outside_temp_change'],
            self.terminal,
            self.reward,
            totrwdstr,
        ))
```

Training a BRAIN to control or optimize a complex system is an iterative process. It proves helpful to log each action that the BRAIN takes and each system state that results from the action. Each of these action state pairs is called an iteration. A series of iterations that comprises a training scenario is called an episode. For example, if you are training a BRAIN to control an autonomous vehicle to stay within a driving lane at a manufacturing plant, each steering correction and the resulting position of the vehicle is an iteration.  One trip around the plant might be an episode.

Logs that document states and actions for iterations and episodes help identify errors in the simulator output and help assess how the training is progressing. Logs can be written to files or to the command line terminal. The example shown is a Python function in a STAR.py file that writes each iteration to the terminal and to a log file. 



[1]: https://github.com/BonsaiAI/bonsai-simulink/tree/master/examples/simulink-househeat
[2]: #speeding-up-training
[3]: https://www.youtube.com/watch?v=0R3PnJEisqk&index=4&list=PLAktfMEMCsOY9HUZKIuGI6yqefGBuszAV]
[4]: ../examples.html#simulink-examples